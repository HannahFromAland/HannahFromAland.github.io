---
title: 【BDP】MapReduce-Secondary Sort
date: 2021-01-03 15:36:26
tags:
- MapReduce
- BigData
category: Big Data Processing
---

## Definition

二级排序的目的是实现key-value对对键排序之后继续按值排序的功能.

- **解决方案1：** 思路完全为Java算法. 在Reducer内直接对给定key的所有值进行排序(例如: 把一个`key`对应的所有`value`放到一个`Array`或`List`中,再排序)，因此可能会出现数据量过大，Reducer内存溢出的情况.
- **解决方案2：** 方案2通过MapReduce思路进行实现，因为在`MapReduce`程序中，Mapper输出的键值对会经过`shuffle`过程再交给Reducer，在 shuffle 阶段，Mapper 输出的键值对会经过 `partition(分区)->sort(排序)->group(分组) `三个阶段，在MapReduce中可以通过重写`sort`来实现，因此只需要对key的格式进行修改，加入需要二级考虑的值并修改key类的`compareTo`或`Comparator class`即可.

<!-- more -->

方案2的具体流程如下图所示，其中虚线框表示`shuffle`过程：

![](https://i.loli.net/2021/01/03/zKHGbFpdWmnRorP.png)

## 实现

### **自定义Key：**
Mapper过程中需要实现key的重定义，即将数据读入后记录为`<(firstKey,secondKey), value>`，并对该key类的比较器进行重写

-  实现方式：继承`WritableComparable`

```java
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import org.apache.hadoop.io.WritableComparable;

public class PairWritable 
    implements WritableComparable<PairWritable>{
    
    private int first;
    private int second;
    
    /**
     * 构造函数
     */
    public PairWritable() { }
    
    public void set(int first, int second) {
        this.first = first;
        this.second = second;
    }
    
    /**
     * Getters and Setters
     */
    public int getFirst() {
        return first;
    }
    public void setFirst(int first) {
        this.first = first;
    }
    public int getSecond() {
        return second;
    }
    public void setSecond(int second) {
        this.second = second;
    }
    
    public void readFields(DataInput in) throws IOException {
        this.setFirst(in.readInt());
        this.setSecond(in.readInt());
    }
    public void write(DataOutput out) throws IOException {
        out.writeInt(this.getFirst());
        out.writeInt(this.getSecond());
    }
    public int compareTo(PairWritable o) {
        // 先比较 first
        int compare = Integer.valueOf(this.getFirst()).compareTo(o.getFirst());
        if (compare != 0) {
            return compare;
        }
        // first 相同再比较 second
        return Integer.valueOf(this.getSecond()).compareTo(o.getSecond());
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + first;
        result = prime * result + second;
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        PairWritable other = (PairWritable) obj;
        if (first != other.first)
            return false;
        if (second != other.second)
            return false;
        return true;
    }

    @Override
    public String toString() {
        return first + "\t" + second;
    }

}
```

###  **自定义Partition类：**
分区器默认会根据`Map`产出的`key`来决定数据进到哪个`Reduce`.在重写key之后键包含二级排序需要的全部参数，而二级排序需要将`firstKey`相同的记录发送到同一Reducer以便进行组内排序，因此需要重写分组器，按照 `PairWritable` 的第一个字段分区

- 实现方式：继承`partitioner`

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Partitioner;

public class FirstPartitioner 
    extends Partitioner<PairWritable, IntWritable> {

    @Override
    public int getPartition(PairWritable key, IntWritable value, 
            int numPartitions) {
        //default: (key.hashcode() & Integer_MAX_VALUE) % numPartitions
        return ((String.valueOf(key.getFirst()).hashCode() & Integer_MAX_VALUE) 
                % numPartitions;
    }
}
```

之后需要在`main`中自定义分区器:

```java
import org.apache.hadoop.mapreduce.Job;
...
Job job = ...;
...
job.setPartitionerClass(FirstPartitioner.class);
```

### **自定义Comparator（分组）类：**
由于同一个分区仍然有可能出现多个`firstKey`的分组，因此需要仍然按照`firstKey`进行分组

- 实现方式：继承`RawComparator`

```java
import org.apache.hadoop.io.RawComparator;
import org.apache.hadoop.io.WritableComparator;

public class FirstGroupingComparator implements RawComparator<PairWritable> {

    /*
     * 对象比较
     */
    public int compare(PairWritable o1, PairWritable o2) {
        return o1.getFirst().compareTo(o2.getFirst());
    }

    /*
     * 字节比较
     * arg0,arg3为要比较的两个字节数组
     * arg1,arg2表示第一个字节数组要进行比较的收尾位置，arg4,arg5表示第二个
     * 从第一个字节比到组合key中second的前一个字节，因为second为int型，所以长度为4
     */
    public int compare(byte[] arg0, int arg1, int arg2, byte[] arg3, int arg4, int arg5) {
        return WritableComparator.compareBytes(arg0, 0, arg2-4, arg3, 0, arg5-4);
    }
}
```

### MapReduce框架
- Map关键代码：

```java
        private PairWritable mapOutKey = new PairWritable();
        private IntWritable mapOutValue = new IntWritable();
        public void map(LongWritable key, Text value, Context context) 
            throws IOException, InterruptedException {
            String lineValue = value.toString();
            String[] strs = lineValue.split("\t");
            
            //设置组合key和value ==> <(key,value),value>
            mapOutKey.set(strs[0], Integer.valueOf(strs[1]));
            mapOutValue.set(Integer.valueOf(strs[1]));
            
            context.write(mapOutKey, mapOutValue);
        }
```

- Reduce关键代码：不需要额外的工作，将 shuffle 的结果遍历输出即可

```java
        private Text outPutKey = new Text(); 
        public void reduce(PairWritable key, Iterable<IntWritable> values, Context context)
                throws IOException, InterruptedException {
            //迭代输出
            for(IntWritable value : values) {
                outPutKey.set(key.getFirst());
                context.write(outPutKey, value);
            }
            
        }
```

- `Main`关键设置

```java
public static void main(String[] args) 
            throws IOException, ClassNotFoundException, InterruptedException {
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration, "secondarysort");
        ···
        // shuffle settings
        job.setPartitionerClass(FirstPartitioner.class);
        job.setGroupingComparatorClass(FirstGroupingComparator.class);
        ···
    }
```

