# Spring Security

- pom依赖

  ```xml
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-security</artifactId>
  </dependency>
  ```

- 创建自定义安全配置

  ```java
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.security.config.annotation.authentication.
  builders.AuthenticationManagerBuilder;
  import org.springframework.security.config.annotation.web.builders.
  HttpSecurity;
  import org.springframework.security.config.annotation.web.configuration.
  EnableWebSecurity;
  import org.springframework.security.config.annotation.web.configuration.
  WebSecurityConfigurerAdapter;
  import org.springframework.security.core.userdetails.UserDetails;
  import org.springframework.security.core.userdetails.UserDetailsService;
  import org.springframework.security.core.userdetails.
  UsernameNotFoundException;
  @Configuration
  @EnableWebSecurity
  public class SecurityConfig extends WebSecurityConfigurerAdapter {
      @Autowired
      private ReaderRepository readerRepository;	//示例，需要修改为自己的类
      @Override
      protected void configure(HttpSecurity http) throws Exception {
          http
          .authorizeRequests()
          .antMatchers("/").access("hasRole('READER')")	//要求登录这有READER角色
          .antMatchers("/**").permitAll()
          .and()
          .formLogin()
          .loginPage("/login")		//设置登表单的路径
          .failureUrl("/login?error=true");
      }
  
      @Override
      protected void configure(AuthenticationManagerBuilder auth) throws Exception {
          auth.userDetailsService(new UserDetailsService() {	//自定义UserDetailsService
              @Override
              public UserDetails loadUserByUsername(String username)
              throws UsernameNotFoundException {
                  return readerRepository.findOne(username);	//从库里查找用户信息
              }
          });
      }
  }
  ```

  自定义配置继承WebSecurityConfigurerAdapter，重写了类里的两个configure()方法。

  - 第一个configure()方法指明，”/"的请求只有经过身份认证拥有READER角色的用户才能访问，其它的所有请求路径向所有用户开放了访问权限，这里将登录页和登录失败页指定到了/login
  - 第二个configure()方法设置了一个自定义的UserDetailsService，用于查找指定用户名的用户