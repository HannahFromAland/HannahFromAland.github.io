---
title: 【BDP】MapReduce-MatrixMultiply
date: 2021-01-01 16:12:03
tags:
- MapReduce
- BigData
category: Big Data Processing
---

### Definition

任意矩阵$M$和$N$,若$M$列数等于$N$的列数，则记M和N的乘积$P=M \cdot N$,且乘积矩阵$P$中元素均可表示为：
$$
p_{ik} = (M\cdot N)_{ik} = \sum_j{m_{ij}n_{jk}}
$$

- 最终决定$p_{ik}$位置的元素为$i,k$，因此将其作为Reduce的输出key值；
- 为记录$m_{ij}$和$n_{jk}$，需要对矩阵名称及元素所在行列进行记录，这些属性将通过Mapper处理得到.

<!-- more -->

并行化矩阵乘法的MapReduce实现如下：

- **Map阶段：**为两矩阵相乘元素进行数据准备
  - 将矩阵元素及其属性整理为<key,value>对；
  - 矩阵M中的元素：$<(i, k) ,(M, j, m_{ij})>$，其中k为遍历直到矩阵N的总列数；
  - 同理，矩阵M中的元素：$<(i, k) ,(N, j, n_{jk})>$，其中i为遍历直到矩阵M的总行数；
- **Reduce阶段：**将key相同的元素进行分组，并最终对组内j相同的元素相乘并最后相加，即可得到$p_{ik}$的值.

### 代码实现

#### Mapper类

- 执行map()之前需要先将相乘两矩阵的行列信息作为全局变量读入，因此需要进行setup()

```java
  public static class MatrixMapper extends Mapper<Object, Text, Text, Text> {
    private Text map_key = new Text();
    private Text map_value = new Text();

    /**
     * 执行map()函数前先由conf.get()得到main函数中提供的必要变量， 这也是MapReduce中共享变量的一种方式
     */
    public void setup(Context context) throws IOException {
      Configuration conf = context.getConfiguration();
      columnN = Integer.parseInt(conf.get("columnN"));
      rowM = Integer.parseInt(conf.get("rowM"));
    }

    public void map(Object key, Text value, Context context)
        throws IOException, InterruptedException {
      /** 得到输入文件名，从而区分输入矩阵M和N **/
      FileSplit fileSplit = (FileSplit) context.getInputSplit();
      String fileName = fileSplit.getPath().getName();

      if (fileName.contains("M")) {
        String[] tuple = value.toString().split(",");
        int i = Integer.parseInt(tuple[0]);
        String[] tuples = tuple[1].split("\t");
        int j = Integer.parseInt(tuples[0]);
        int Mij = Integer.parseInt(tuples[1]);

        for (int k = 1; k < columnN + 1; k++) {
          map_key.set(i + "," + k);
          map_value.set("M" + "," + j + "," + Mij);
          context.write(map_key, map_value);
        }
      }

      else if (fileName.contains("N")) {
        String[] tuple = value.toString().split(",");
        int j = Integer.parseInt(tuple[0]);
        String[] tuples = tuple[1].split("\t");
        int k = Integer.parseInt(tuples[0]);
        int Njk = Integer.parseInt(tuples[1]);

        for (int i = 1; i < rowM + 1; i++) {
          map_key.set(i + "," + k);
          map_value.set("N" + "," + j + "," + Njk);
          context.write(map_key, map_value);
        }
      }
    }
  }
```

#### Reducer类

```java  
  public static class MatrixReducer extends Reducer<Text, Text, Text, Text> {
    private int sum = 0;

    public void setup(Context context) throws IOException {
      Configuration conf = context.getConfiguration();
      columnM = Integer.parseInt(conf.get("columnM"));
    }
    
    public void reduce(Text key, Iterable<Text> values, Context context)
        throws IOException, InterruptedException {
      int[] M = new int[columnM + 1];
      int[] N = new int[columnM + 1];
    
      for (Text val : values) {
        String[] tuple = val.toString().split(",");
        if (tuple[0].equals("M")) {
          M[Integer.parseInt(tuple[1])] = Integer.parseInt(tuple[2]);
        } else
          N[Integer.parseInt(tuple[1])] = Integer.parseInt(tuple[2]);
      }
    
      /** 根据j值，对M[j]和N[j]进行相乘累加得到乘积矩阵的数据 **/
      for (int j = 1; j < columnM + 1; j++) {
        sum += M[j] * N[j];
      }
      context.write(key, new Text(Integer.toString(sum)));
      sum = 0;
    }
  }
```