# 스코프

## 어노테이션

* `@JobScope`, `@StepScope` 를 통해 Job, Step 객체가 실제로 사용되는 실행 시점에 생성되도록 지연시킬 수 있다.
* `@JobScope`는 Step 선언문에서 사용 가능하고, `@StepScope`는 Tasklet이나 ItemReader, ItemWriter, ItemProcessor에서 사용할 수 있다.
* 내부, 외부에서 파라미터를 받아 여러 Batch 컴포넌트에서 사용 가능한 `Job Parameters`를 사용하려면, 1. Bean을 정의하는 부분에 스코프를 선언하고 2. 인자에 `@Value("#{jobParameters[파라미터명]}")` 을 등록해주어야 한다.

