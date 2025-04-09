# Spring组件

## security安全

- 导入依赖

  ```xml
  <!-- 
  该依赖是用于应用程序，保护web应用程序
  -->
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-security</artifactId>
  </dependency>
  <!-- 
  该依赖是 Thymeleaf 与 Spring Security 6 的集成库，主要作用是在 Thymeleaf 模板中直接使用 Spring Security 的功能，实现安全相关的动态内容渲染。
  -->
  <dependency>
  	<groupId>org.thymeleaf.extras</groupId>
  	<artifactId>thymeleaf-extras-springsecurity6</artifactId>
  	<!-- Temporary explicit version to fix Thymeleaf bug -->
  	<version>3.1.1.RELEASE</version>
  </dependency>
  <!-- 
  用于测试环境
  -->
  <dependency>
  	<groupId>org.springframework.security</groupId>
  	<artifactId>spring-security-test</artifactId>
  	<scope>test</scope>
  </dependency>
  ```

- 进行安全配置（官方模版举例）-- 保护web程序

  ```Java
  @Configuration//表示当前为配置类
  @EnableWebSecurity//启用Spring Security的web安全支持并提供SpringMVC集成
  public class WebSecurityConfig {
  	/**
  	该bean定义哪些URL路径应该是安全的，哪些不应该。
  	按照当前配置而言，“/ 和 /home”路径配置为不需要任何身份验证，所有的	  其他路径都必须经过身份验证。
  	当用户成功登录时，他们将被重定向到以前请求的需要身份验证页面。有一个自定义/login页面(指向loginPage())，每个人都能查看。
  	**/
  	@Bean
  	public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
  		http
  			.authorizeHttpRequests((requests) -> requests
  				.requestMatchers("/", "/home").permitAll()
  				.anyRequest().authenticated()
  			)
  			.formLogin((form) -> form
  				.loginPage("/login")
  				.permitAll()
  			)
  			.logout((logout) -> logout.permitAll());
  
  		return http.build();
  	}
  	/**
  	UserDetailsService bean使用单个用户在内存中设置用户存储。
  	该用户被赋予用户名user、密码password和角色USER。
  	**/
  	@Bean
  	public UserDetailsService userDetailsService() {
  		UserDetails user =
  			 User.withDefaultPasswordEncoder()
  				.username("user")
  				.password("password")
  				.roles("USER")
  				.build();
  		return new InMemoryUserDetailsManager(user);
  	}
  }
  ```

  