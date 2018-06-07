[맵 리듀스 개발환경 설정하기](https://play.google.com/books/reader?id=MeUlDwAAQBAJ&pg=GBS.PA205)
[맵 리듀스 잡으로 문제 분해하기](https://play.google.com/books/reader?id=MeUlDwAAQBAJ&hl=ko&printsec=frontcover&pg=GBS.PA243)
[두 개 이상의 JobControl](https://play.google.com/books/reader?id=MeUlDwAAQBAJ&hl=ko&printsec=frontcover&pg=GBS.PA245), [클러스터의 오지워크플로 정의하기](https://play.google.com/books/reader?id=MeUlDwAAQBAJ&hl=ko&printsec=frontcover&pg=GBS.PA247)
[input data - TextInputFormat](https://play.google.com/books/reader?id=MeUlDwAAQBAJ&hl=ko&printsec=frontcover&pg=GBS.PA303)
[input data - DBInputFormat](https://play.google.com/books/reader?id=MeUlDwAAQBAJ&hl=ko&printsec=frontcover&pg=GBS.PA311)

[클러스터 실행하기](https://play.google.com/books/reader?id=MeUlDwAAQBAJ&hl=ko&printsec=frontcover&pg=GBS.PA223)
[잡 튜닝하기](https://play.google.com/books/reader?id=MeUlDwAAQBAJ&hl=ko&printsec=frontcover&pg=GBS.PA241)

[맵리듀스 조인](https://play.google.com/books/reader?id=MeUlDwAAQBAJ&hl=ko&printsec=frontcover&pg=GBS.PA345)


##맵
데이터의 준비단계
연도별 최고 기온을 찾는 리듀스 함수를 위해 데이터를 제공하는 역할
잘못된 데이터를 걸러주는 작업 (기온 필드의 값이 누락되거나 이상하거나 문제가 있는 레코드를 제거하는 작업)
1. 원본데이터의 각 행에서 연도와 기온을 추출
\- 맵 함수의 키를 연도, 값을 기온값으로 설정.
2. 맵리듀스 프레임워크에 의해 맵함수의 출력이 리듀스 함수의 입력으로 보내진다. 
(이 과정에서 키-값 쌍은 키를 기준으로 정렬되고 그룹화되어 묶인다.)
\- 연도별로 측정된 모든 기온값이 하나의 리스트로 묶인다.

```java
import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class MaxTemperatureMapper
  extends Mapper<LongWritable, Text, Text, IntWritable> {

  private static final int MISSING = 9999;
  
  @Override
  public void map(LongWritable key, Text value, Context context)
      throws IOException, InterruptedException {
    
    String line = value.toString();
    String year = line.substring(15, 19);
    int airTemperature;
    if (line.charAt(87) == '+') { // parseInt doesn't like leading plus signs
      airTemperature = Integer.parseInt(line.substring(88, 92));
    } else {
      airTemperature = Integer.parseInt(line.substring(87, 92));
    }
    String quality = line.substring(92, 93);
    if (airTemperature != MISSING && quality.matches("[01459]")) {
      context.write(new Text(year), new IntWritable(airTemperature));
    }
  }
}
```

##리듀스
3. 리듀스 함수는 리스트 전체를 반복하여 최고 측정값을 추출한다. 
4. 최종 결과로 연도별 전 세계 최고 기온을 추출한다. 

```java
import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class MaxTemperatureReducer
  extends Reducer<Text, IntWritable, Text, IntWritable> {
  
  @Override
  public void reduce(Text key, Iterable<IntWritable> values,
      Context context)
      throws IOException, InterruptedException {
    
    int maxValue = Integer.MIN_VALUE;
    for (IntWritable value : values) {
      maxValue = Math.max(maxValue, value.get());
    }
    context.write(key, new IntWritable(maxValue));
  }
}
```
## Main 클래스
job을 수행하는 방법을 정의한 잡 명세서를 작성
- 하둡 클러스터에서 잡을 실행할 때는 먼저 코드를 jar 파일로 묶어야 한다. (하둡은 클러스터의 해당 머신에 jar파일을 배포한다.)
- 입력경로 지정 : 파일이나 디렉터리()

```java
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MaxTemperature {

  public static void main(String[] args) throws Exception {
    if (args.length != 2) {
      System.err.println("Usage: MaxTemperature <input path> <output path>");
      System.exit(-1);
    }
    
    Job job = new Job();
    job.setJarByClass(MaxTemperature.class);
    job.setJobName("Max temperature");

    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    
    job.setMapperClass(MaxTemperatureMapper.class);
    job.setReducerClass(MaxTemperatureReducer.class);

    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```