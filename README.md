# 4. 데이터공유

## ShareConfiguration 클래스 생성
```java
@Slf4j
@Configuration
public class ShareConfiguration {
	
}
```

---  

<br>
<br>  

## shareJob과 shareStep1, shareStep2 생성
```java
@Slf4j
@Configuration
public class ShareConfiguration {

	private final JobBuilderFactory jobBuilderFactory;
	private final StepBuilderFactory stepBuilderFactory;

	public ShareConfiguration(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory) {
		this.jobBuilderFactory = jobBuilderFactory;
		this.stepBuilderFactory = stepBuilderFactory;
	}

	@Bean
	public Job shareJob() {
		return jobBuilderFactory.get("shareJob")
								.incrementer(new RunIdIncrementer())
								.start(this.shareStep1())
								.next(this.shareStep2())
								.build();
	}

	@Bean
	public Step shareStep1() {
		return stepBuilderFactory.get("shareStep")
								 .tasklet((contribution, chunkContext) -> {
									 StepExecution stepExecution = contribution.getStepExecution();
									 ExecutionContext stepExecutionContext = stepExecution.getExecutionContext();
									 stepExecutionContext.put("stepKey", "step execution context");

									 JobExecution jobExecution = stepExecution.getJobExecution();
									 JobInstance jobInstance = jobExecution.getJobInstance();

									 ExecutionContext jobExecutionContext = jobExecution.getExecutionContext();
									 jobExecutionContext.putString("jobKey", "job execution context");
									 JobParameters jobParameters = jobExecution.getJobParameters();
									 System.out.println("===================1====================");
									 log.info("jobName : {}, stepName : {}, parameter {}",
											  jobInstance.getJobName(),
											  stepExecution.getStepName(),
											  jobParameters.getLong("run.id"));
									 return RepeatStatus.FINISHED;
								 })
								 .build();
	}

	@Bean
	public Step shareStep2() {
		return stepBuilderFactory.get("shareStep2")
								 .tasklet((contribution, chunkContext) -> {
									 StepExecution stepExecution = contribution.getStepExecution();
									 ExecutionContext stepExecutionContext = stepExecution.getExecutionContext();

									 JobExecution jobExecution = stepExecution.getJobExecution();
									 ExecutionContext jobExecutionContext = jobExecution.getExecutionContext();
									 JobInstance jobInstance = jobExecution.getJobInstance();
									 System.out.println("==================2=====================");
									 log.info("jobName : {}, jobKey : {}, stepKey : {}",
											  jobInstance.getJobName(),
											  jobExecutionContext.getString("jobKey", "emptyJobKey"),
											  stepExecutionContext.getString("stepKey", "emptyStepKey")
									 );

									 return RepeatStatus.FINISHED;
								 })
								 .build();
	}
}

```

<br>  

> 위 코드를 실행하면 아래 와 같은 결과가 나온다.
```text
===================1====================
2023-05-30 21:59:44.929  INFO 11596 --- [           main] c.e.s.part1.ShareConfiguration           : jobName : shareJob, stepName : shareStep, parameter 8
2023-05-30 21:59:44.975  INFO 11596 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [shareStep] executed in 132ms
2023-05-30 21:59:45.370  INFO 11596 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [shareStep2]
==================2=====================
2023-05-30 21:59:45.444  INFO 11596 --- [           main] c.e.s.part1.ShareConfiguration           : jobName : shareJob, jobKey : job execution context, stepKey : emptyStepKey
2023-05-30 21:59:45.492  INFO 11596 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [shareStep2] executed in 122ms
2023-05-30 21:59:45.588  INFO 11596 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=shareJob]] completed with the following parameters: [{run.id=8}] and the following status: [COMPLETED] in 809ms
```


<br>  

> step간에는 데이터 공유가 가능하다 하지만 주의할 점은 shareStep1 에서`stepExecutionContext.put("stepKey", "step execution context");` 는 shareStep2에서 공유가 안되고  `jobExecutionContext.putString("jobKey", "job execution context");` 는 공유가 된다는 것이다.

---

<br>
<br>


## 비고
> 현재는 HelloConfiguration와 ShareConfiguration클래스에 있는 job이 동시에 실행한다. 아래와 같은 설정을 해줘야 하나만 작동한다.
```yaml
server:
  port: 8090

spring:
  batch:
    job:
      names: ${job.name:NONE}    # 주석해제
    jdbc:
      initialize-schema: never
  datasource:
    url: jdbc:mysql://localhost:3306/spring_batch?characterEncoding=UTF-8&serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 1234
```

![img_3.png](img_3.png)
