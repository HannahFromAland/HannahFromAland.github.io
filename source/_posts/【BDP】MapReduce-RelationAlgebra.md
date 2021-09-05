---
title: 【BDP】MapReduce-RelationAlgebra
date: 2021-01-02 10:06:07
tags:
- MapReduce
- BigData
category: Big Data Processing
---

利用MapReduce并行化实现选择、投影、并、交、差及自然连接的关系代数操作。

<!-- more -->

### Selection

**Description：**获得某属性满足某一条件的所有记录

- **Map操作：**对输入记录的条件属性进行判断，若满足条件则输出该键值对
- 选择操作中只需要直接输出Map的结果，不需要Reduce操作

**Example:** 在关系R中找出所有属性号为id，值为value的项并输出

- 首先对已有关系表进行类构建，并创建`isCondition`方法直接对选择条件进行判断

```java
	public boolean isCondition(int col, String value){
		if(col == 0 && Integer.parseInt(value) == this.id)
			return true;
		else if(col == 1 && name.equals(value))
			return true;
		else if(col ==2 && Integer.parseInt(value) == this.age)
			return true;
		else if(col ==3 && Double.parseDouble(value) == this.weight)
			return true;
		else
			return false;
	}
```
- **Mapper操作实现**：通过setup从main函数读入选择的变量作为全局参数，调用RelationA中的判断方法对所有记录进行筛选并输出合格键值对

```java
public static class SelectionMap extends Mapper<LongWritable, Text, RelationA, NullWritable>{

		private int id;
		private String value;
		@Override
		protected void setup(Context context) 
		throws IOException,InterruptedException{
			id = context.getConfiguration().getInt("col", 0);
			//second param represents the default value
			value = context.getConfiguration().get("value");
		}
		
		@Override
		public void map(LongWritable offSet, Text line, Context context)throws 
		IOException, InterruptedException{
			RelationA record = new RelationA(line.toString());
			if(record.isCondition(id, value))
				context.write(record, NullWritable.get());
		}
	}

```

- **Reducer操作实现：**无需进行额外操作，`setNumReduceTasks(0)`即可

### Projection

投影运算：选择列col的全部值并输出

- **Mapper操作：**选择所有的列col值并输出

```java
	public static class ProjectionMap extends Mapper<LongWritable, Text, Text, NullWritable>{
		private int col;
		@Override
		protected void setup(Context context) throws IOException,InterruptedException{
			col = context.getConfiguration().getInt("col", 0);
		}
		
		@Override
		public void map(LongWritable offSet, Text line, Context context)throws 
		IOException, InterruptedException{
			RelationA record = new RelationA(line.toString());
			context.write(new Text(record.getCol(col)), NullWritable.get());
		}	
	}
```

- **Reducer操作：**若直接输出全部结果，则和选择操作一样将Reducer个数设置为0即可；若需要对全部列值进行去重，则需要创建Reducer对NullWritable进行遍历，调用该Reducer即可输出去重后的全部列值（代码实现如下）

```java
	public static class ProjectionReduce 
	extends Reducer<Text, NullWritable, Text, NullWritable>{
		@Override
		public void reduce(Text key, Iterable<NullWritable> value, Context context) throws 
		IOException,InterruptedException{
			context.write(key, NullWritable.get());
		}
	}
```



### InterSection

对相同模式的关系表R和关系表T求交集

- **Mapper操作：**读取R和T中每一条记录r，输入<r,1>

```java
public static class IntersectionMap 
extends Mapper<LongWritable, Text, RelationA, IntWritable>{
		private IntWritable one = new IntWritable(1);
		@Override
		public void map(LongWritable offSet, Text line, Context context)throws 
		IOException, InterruptedException{
			RelationA record = new RelationA(line.toString());
			context.write(record, one);
		}
	}
```



- **Reducer操作：**合并键相同的元素值，若value=2则为交集，将该条记录输出

```java
public static class IntersectionReduce 
extends Reducer<RelationA, IntWritable, RelationA, NullWritable>{
		@Override
		public void reduce(RelationA key, 
		Iterable<IntWritable> value, Context context) throws 
		IOException,InterruptedException{
			int sum = 0;
			for(IntWritable val : value){
				sum += val.get();
			}
			if(sum == 2)
				context.write(key, NullWritable.get());
		}
	}
```

- Reducer处理的关键在于：必须保证R和T表中相同的元素记录被发送到了相同的Reducer

- 解决方案1：对于较小数据集将Reducer个数设置为1即可
- 解决方案2：在Map中以RelationA作为主键时，将相同记录的hashcode设定为相同值即可保证两条记录被发送到同一Reducer


```java
public int hashCode(){
    int result = 17;
    result = 31 * result + id;
    result = 31 * result + name.hashcode();
    result = 31 * result + age;
    result = 31 * result + grade;
    return result;
}
```



### Difference

同一个模式的R和T求差集，基本处理逻辑和交运算相同

- **Mapper操作：**输入R中记录为<r, R>，T中记录为<r,T>（也可以直接按照并集的处理方式，在Reducer端进行相减操作，返回value=1的值即可）
- **Reducer操作：**若某键的对应值只有R没有T，将该条记录输出
- 同样需要重写hashCode()或限制reducer数量为1

### NaturalJoin

以某属性为主键做关系表R和关系表S的自然连接

- Mapper操作：将主键作为key，其余属性作为value（需包括两表名称）

```java
	public static class NaturalJoinMap extends Mapper<Text, BytesWritable, Text, Text>{
		private int col;
		@Override
		protected void setup(Context context) throws IOException,InterruptedException{
			col = context.getConfiguration().getInt("col", 0);
		}
		@Override
		public void map(Text relationName, BytesWritable content, Context context)throws 
		IOException, InterruptedException{
			String[] records = new String(content.getBytes(),"UTF-8").split("\\n");
			for(int i = 0; i < records.length; i++){
				RelationA record = new RelationA(records[i]);
				context.write(new Text(record.getCol(col)), 
						new Text(relationName.toString() + " " 
						+ record.getValueExcept(col)));
			}
		}
	}
```

- Reducer操作：对key相同的键值对分为不同来源进行笛卡尔积

```java
	public static class NaturalJoinReduce extends Reducer<Text,Text,Text,NullWritable>{
		private String relationNameA;
		protected void setup(Context context) throws IOException,InterruptedException{
			relationNameA = context.getConfiguration().get("relationNameA");
		}
		public void reduce(Text key, Iterable<Text> value, Context context)throws 
		IOException,InterruptedException{
			ArrayList<Text> setR = new ArrayList<Text>();
			ArrayList<Text> setS = new ArrayList<Text>();
			//按照来源分为两组然后做笛卡尔乘积
			for(Text val : value){
				String[] recordInfo = val.toString().split(" ");
				if(recordInfo[0].equalsIgnoreCase(relationNameA))
					setR.add(new Text(recordInfo[1]));
				else
					setS.add(new Text(recordInfo[1]));
			}
			for(int i = 0; i < setR.size(); i++){
				for(int j = 0; j < setS.size(); j++){
					Text t = new Text(setR.get(i).toString() + ","
                    + key.toString() + "," + 
                    + setS.get(j).toString());
					context.write(t, NullWritable.get());
				}
			}
		}
	}
```

