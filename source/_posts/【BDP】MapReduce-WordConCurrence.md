---
title: 【BDP】MapReduce WordConCurrence
date: 2021-01-03 20:27:21
tags:
- MapReduce
- BigData
category: Big Data Processing
---

### Definition

目的是在海量语料库/文章中发现固定窗口（如5词以内、一句话内甚至一段内）单词a和单词b共同出现的频率，并以此构建单词共现矩阵。（矩阵可对称也可不对称（强调顺序），取决于具体应用）.

<!-- more -->

### 单词共现算法实现

利用MapReduce实现单词共现算法的伪代码如下：

- Mapper：对窗口中的单词对进行遍历并输出，当窗口到达文档尾部时通过头部向后缩进来实现滑动，直到窗口大小为2时停止.

```
class Mapper 
    method Map(dociddid, doc d)
    for all word w in d
for all word u in Window(w)
//发射出现计数 1
        Emit(pair (w, u), 1)
 
```

- Reducer：对相同单词对的出现次数进行汇总

```
class Reducer
    method Reduce(pair p; countlist [c1, c2,..])
        s = 0
for all count c in countlist [c1, c2, ...]
sum += c
Emit(pair p, count sum)
```

### Java MapReduce实现

#### 自定义Key

- 通过继承`WritableComparable`实现`Key：WordPair`类

- 为保证所有相同的单词对都能传入相同的Reducer进行处理，需要重写`hashCode()`方法使相同的单词对（不考虑顺序）在规约时处于同一Reducer. 

- 通过重写`compareTo()`和`equals()`方法使得相同的键的值可以进行大小的比较和排序

```java
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.io.WritableComparable;

public class WordPair implements WritableComparable<WordPair>{
	private String wordA;
	private String wordB;
	
	public WordPair(){		
	}
	
	public WordPair(String wordA,String wordB){
		this.wordA = wordA;
		this.wordB = wordB;
	}
	
	public String getWordA(){
		return this.wordA;
	}
	
	public String getWordB(){
		return this.wordB;
	}
    
    @Override
	public void write(DataOutput out) throws IOException {
		// TODO Auto-generated method stub
		out.writeUTF(wordA);
		out.writeUTF(wordB);
	}

	@Override
	public void readFields(DataInput in) throws IOException {
		// TODO Auto-generated method stub
		wordA = in.readUTF();
		wordB = in.readUTF();
	}
	
	@Override
	public String toString(){
		return wordA + "," + wordB;
	}
    
    //compareTo
	@Override
	public int compareTo(WordPair o) {
		if(this.equals(o))
			return 0;
		else
			return (wordA + wordB).compareTo(o.getWordA() + o.getWordB());
	}
    //equals 
    @Override
	public boolean equals(Object o){
		//无序对，不用考虑顺序
		if(!(o instanceof WordPair))
			return false;
		WordPair w = (WordPair) o;
		if((this.wordA.equals(w.wordA) && this.wordB.equals(w.wordB))
           || (this.wordB.equals(w.wordA) && this.wordA.equals(w.wordB)))
				return true;
		return false;
	}
    //hashcode
    @Override 
	public int hashCode(){
		return (wordA.hashCode() + wordB.hashCode()) * 17;
	}	
}
```

#### Map端实现

```java
public class WordConcurrnce {
	private static int MAX_WINDOW = 20;//单词同现的最大窗口大小
	private static String wordRegex = "([a-zA-Z]+)";//仅仅匹配由字母组成的简单英文单词
	private static Pattern wordPattern = Pattern.compile(wordRegex);//用于识别英语单词(带连字符-)
	private static IntWritable one = new IntWritable(1);
	
	public static class WordConcurrenceMapper extends Mapper<Text, BytesWritable, WordPair, IntWritable>{
		private int windowSize;
		private Queue<String> windowQueue = new LinkedList<String>(); 
		
		@Override
		protected void setup(Context context) throws IOException,InterruptedException{
			windowSize = Math.min(context.getConfiguration().getInt("window", 2) , MAX_WINDOW);
		}
		
		/**
		 * 输入键位文档的文件名，值为文档中的内容的字节形式。
		 * 
		 */
		@Override
		public void map(Text docName, BytesWritable docContent, Context context)throws 
		IOException, InterruptedException{
			Matcher matcher = wordPattern.matcher(new String(docContent.getBytes(),"UTF-8"));
			while(matcher.find()){
				windowQueue.add(matcher.group());
				if(windowQueue.size() >= windowSize){
					//对于队列中的元素[q1,q2,q3...qn]发射[(q1,q2),1],[(q1,q3),1],
					//...[(q1,qn),1]出去
					Iterator<String> it = windowQueue.iterator();
					String w1 = it.next();
					while(it.hasNext()){
						String next = it.next();
						context.write(new WordPair(w1, next), one);
					}
					windowQueue.remove();
				}
			}
			if(!(windowQueue.size() <= 1)){
				Iterator<String> it = windowQueue.iterator();
				String w1 = it.next();
				while(it.hasNext()){
					context.write(new WordPair(w1,it.next()), one);
				}
			}
		}
		
	}
```

#### WholeFileInputFormat

因为需要统计各文章中单词之间的关系，因此需要实现单个文件读入以保证一个文本不被拆分地将内部单词对传入Map节点（具体实现待更新..）



