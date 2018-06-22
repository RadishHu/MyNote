# Spring Boot定时任务

- 在Spring Boot的主类中加入 `@EnableScheduling`注解，启用定时任务配置

  ```java
  @SpringBootApplication
  @EnableScheduling
  public class Application {
  	public static void main(String[] args) {
  		SpringApplication.run(Application.class, args);
  	}
  }
  ```

- 创建定时任务实现类

  ```java
  @Component
  public class ScheduledTasks {
  
      private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");
  
      @Scheduled(fixedRate = 5000)
      public void reportCurrentTime() {
          System.out.println("现在时间：" + dateFormat.format(new Date()));
      }
  
  }
  ```

## @Scheduled详解

@Schedeled使用方式：

- @Scheduled(fixedRate = 5000)：上次开始执行时间点之后5秒再执行
- @Scheduled(fixedDelay = 5000)：上次执行完毕时间点之后5秒再执行
- @Scheduled(initialDelay = 1000,fixedRate = 5000)：第一次延迟1秒后执行，之后按fixedRate的规则执行
- @Scheduled(cron="*/5 * * * * *")：通过cron表达式定义规则执行，cron表达式使用方法，

