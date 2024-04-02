---
title: WebMvcConfigurer
date: 2024-04-02 16:04:30
tags:
  - SpringMVC
  - Configurer
---
# WebMvcConfigurer

SpringMVC 允许用户通过实现 `WebMvcConfigurer` 接口配置 Spring MVC 的各种自定义行为，而无需修改已有代码，主要作用包括但不限于

1. **配置视图解析器（View Resolvers）**：决定如何映射视图名称到实际的视图页面
2. **处理静态资源（比如 JS、CSS、图片等）**：配置静态资源的路径，以便Spring MVC可以将请求映射到静态资源上
3. **配置内容裁决的视图**：可以基于请求的不同配置不同的视图解析策略
4. **配置拦截器（Interceptors）**：拦截器可以在请求被处理之前或之后执行代码，用于执行如权限检查、日志记录等操作
5. **跨域请求处理（CORS）**：允许或限制跨源请求
6. **格式化数据和转换数据**：注册自定义的格式化程序和值转换器，用于请求和响应中数据的转换
7. **消息转换器（Message Converters）**：配置用于请求和响应体的HTTP消息转换器，比如将对象自动转换为JSON或XML
8. **异常处理**：配置全局异常处理器

使用时，通常通过创建一个配置类 （[@Configuration 注解](..\Spring\注解\@Configuration.md)）来实现 `WebMvcConfigurer` 接口，然后重写其中的方法实现自定义配置

## 处理静态资源

通过实现 `addResourceHandlers` 方法定义静态资源的位置和映射路径

静态资源包括 JavaScript、CSS、图片等，通过该配置使得这些静态资源可以从不同的位置被访问，比如类路径、文件系统或其他地方

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MvcConfig implements WebMvcConfigurer {

  @Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
    // 映射所有 /static/** 的请求到类路径下的 /static/ 目录
    registry
      .addResourceHandler("/static/**")
      .addResourceLocations("classpath:/static/");

    // 映射所有 /media/** 的请求到文件系统的 /data/media/ 目录
    registry
      .addResourceHandler("/media/**")
      .addResourceLocations("file:/data/media/");
  }
}
```

- **第一条规则**是为类路径下的 `/static/` 目录内的资源提供服务。这意味着，当请求URL匹配 `/static/**` 模式时（例如，`/static/js/app.js`），Spring MVC会尝试从应用的类路径下的 `/static/` 目录中提供这些文件。
- **第二条规则**配置了文件系统上的一个特定路径(`/data/media/`)来服务 `/media/**` 模式的请求。这样，任何匹配该模式的请求（例如，`/media/images/logo.png`）都会被映射到文件系统上的 `/data/media/`目录下寻找对应的资源

`addResourceHandlers` 方法作用与 [mvc resources 标签](静态资源访问.md#mvc%20resources%20标签) 作用相同

## 配置拦截器

通过实现 `addinterceptors` 方法注册自定义的拦截器。这些拦截器可以拦截进入 Controller 之前或者在视图渲染之后的请求，实现一些如权限检查、日志记录、事务管理等跨切面功能

首先，创建一个拦截器类，实现 `HandlerInterceptor` 接口

```java
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public class MyInterceptor implements HandlerInterceptor {

  @Override
  public boolean preHandle(
    HttpServletRequest request,
    HttpServletResponse response,
    Object handler
  ) throws Exception {
    // 在Controller处理请求之前调用，返回false则中断请求
    System.out.println("Pre Handle method is Calling");
    return true;
  }

  @Override
  public void postHandle(
    HttpServletRequest request,
    HttpServletResponse response,
    Object handler,
    ModelAndView modelAndView
  ) throws Exception {
    // 在Controller处理请求之后、视图渲染之前调用
    System.out.println("Post Handle method is Calling");
  }

  @Override
  public void afterCompletion(
    HttpServletRequest request,
    HttpServletResponse response,
    Object handler,
    Exception ex
  ) throws Exception {
    // 在整个请求结束之后调用，也就是在DispatcherServlet渲染了视图执行
    System.out.println("Request and Response is completed");
  }
}
```

在 Spring MVC 配置类中注册这个拦截器

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

  @Autowired
  private MyInterceptor myInterceptor;

  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    // 注册拦截器到Spring MVC机制，然后它会返回一个拦截器注册
    registry
      .addInterceptor(myInterceptor)
      // 指定该拦截器需要拦截的URL路径模式
      .addPathPatterns("/api/**")
      // 指定该拦截器需要过滤的URL路径模
      .excludePathPatterns("/admin/employee/login");
  }
}
```

