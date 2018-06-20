# Spring注解

- @Component，component scanning，标明该类作为组件类，并告知Spring要为这个类创建bean。Spring会根据类的名字为其制定一个ID，默认ID为类名的第一个字母变为小写。也可以自己设置ID：

  ```
  @Component("ID")
  ```
  @Controller，声明控制器

- @ComponentScan，使用java进行装配时，使用该注解启用组件扫描。默认扫描与配置类相同的包，查找带有@Component注解的类。也可以自己设置扫描的包路径：

  ```
  @ComponentScan("packagePath")
  #或者
  @ComponentScan(basePackages="packagePath")
  #指定多个包进行扫描
  @ComponentScan(basePackages={"package1","package2"})
  #指定包中的类或接口
  @ComponentScan(basePackageClasses={scanClass.class,scanInterface.class})
  ```

- @RunWith(SpringJUnit4ClassRunner.class)，进行单元测试

- @ContextConfiguration(classes=JavaConfig.class)，指向java配置文件的类，标明需要在Java配置类中加载配置

- @Autowired，在Spring Application Context中寻找匹配某个bean需求的其它bean。

- @Import，将另一个JavaConfig中定义的Bean组合到该JavaConfig中

  ```java
  package soundsystem;
  
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.Import;
  import org.springframework.context.annotation.ImportResource;
  
  @Configuration
  @Import(CDPlayerConfig.class)
  public class SoundSystemConfig {
  
  }
  ```

- @ImportResource，将XML中定义的Bean组合到JavaConfig中

  ```java
  package soundsystem;
  
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.Import;
  import org.springframework.context.annotation.ImportResource;
  
  @Configuration
  @Import(CDPlayerConfig.class)
  @ImportResource("classpath:cd-config.xml")
  public class SoundSystemConfig {
  
  }
  ```

- 

## Spring MVC

- @Controller，生命一个控制器

- @RequestMapping，声明所要处理的请求

  ```java
  //处理对"/"的GET请求
  @RequestMapping(value="/",method=GET)
  
  //处理对"/my/request"请求
  @RequestMapping("/my/request")
  ```

  

