# Spring MVC
### Spring的模型-视图-控制器（MVC）框架是围绕一个DispatcherServlet来设计的，这个Servlet会把请求分发给各个处理器，并支持可配置的处理器映射、视图渲染、本地化、时区与主题渲染等，甚至还能支持文件上传。处理器是你的应用中注解了@Controller和@RequestMapping的类和方法，Spring为处理器方法提供了极其多样灵活的配置。Spring 3.0以后提供了@Controller注解机制、@PathVariable注解以及一些其他的特性，你可以使用它们来进行RESTful web站点和应用的开发。

## 1.DispatcherServlet
### Spring MVC框架，与其他很多web的MVC框架一样：请求驱动；所有设计都围绕着一个中央Servlet来展开，它负责把所有请求分发到控制器；同时提供其他web应用开发所需要的功能。不过Spring的中央处理器，DispatcherServlet，能做的比这更多。它与Spring IoC容器做到了无缝集成，这意味着，Spring提供的任何特性，在Spring MVC中你都可以使用。
![](https://i.imgur.com/yjwiAwW.jpg)
### DispatcherServlet其实就是个Servlet（它继承自HttpServlet基类），同样也需要在你web应用的web.xml配置文件下声明。你需要在web.xml文件中把你希望DispatcherServlet处理的请求映射到对应的URL上去。这就是标准的Java EE Servlet配置；下面的代码就展示了对DispatcherServlet和路径映射的声明：
	<web-app>
	    <servlet>
	        <servlet-name>example</servlet-name>
	        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	        <load-on-startup>1</load-on-startup>
	    </servlet>
	
	    <servlet-mapping>
	        <servlet-name>example</servlet-name>
	        <url-pattern>/example/*</url-pattern>
	    </servlet-mapping>
	</web-app>
### 在上面的例子中，所有路径以/example开头的请求都会被名字为example的DispatcherServlet处理。在Servlet 3.0+的环境下，你还可以用编程的方式配置Servlet容器。下面是一段这种基于代码配置的例子，它与上面定义的web.xml配置文件是等效的。
	public class MyWebApplicationInitializer implements WebApplicationInitializer {

	    @Override
	    public void onStartup(ServletContext container) {
	        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet());
	        registration.setLoadOnStartup(1);
	        registration.addMapping("/example/*");
	    }
	
	}
### WebApplicationInitializer是Spring MVC提供的一个接口，它会查找你所有基于代码的配置，并应用它们来初始化Servlet 3版本以上的web容器。它有一个抽象的实现AbstractDispatcherServletInitializer，用以简化DispatcherServlet的注册工作：你只需要指定其servlet映射（mapping）即可。

## 2.DispatcherServlet的处理流程
- 首先，搜索应用的上下文对象WebApplicationContext并把它作为一个属性（attribute）绑定到该请求上，以便控制器和其他组件能够使用它。属性的键名默认为DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE
- 将地区（locale）解析器绑定到请求上，以便其他组件在处理请求（渲染视图、准备数据等）时可以获取区域相关的信息。如果你的应用不需要解析区域相关的信息，忽略它即可
- 将主题（theme）解析器绑定到请求上，以便其他组件（比如视图等）能够了解要渲染哪个主题文件。同样，如果你不需要使用主题相关的特性，忽略它即可
- 如果你配置了multipart文件处理器，那么框架将查找该文件是不是multipart（分为多个部分连续上传）的。若是，则将该请求包装成一个MultipartHttpServletRequest对象，以便处理链中的其他组件对它做进一步的处理
- 为该请求查找一个合适的处理器。如果可以找到对应的处理器，则与该处理器关联的整条执行链（前处理器、后处理器、控制器等）都会被执行，以完成相应模型的准备或视图的渲染
- 如果处理器返回的是一个模型（model），那么框架将渲染相应的视图。若没有返回任何模型（可能是因为前后的处理器出于某些原因拦截了请求等，比如，安全问题），则框架不会渲染任何视图，此时认为对请求的处理可能已经由处理链完成了
### 如果在处理请求的过程中抛出了异常，那么上下文WebApplicationContext对象中所定义的异常处理器将会负责捕获这些异常。通过配置你自己的异常处理器，你可以定制自己处理异常的方式。

## 3.控制器(Controller)的实现
	@Controller
	public class HelloWorldController {
	
	    @RequestMapping("/helloWorld")
	    public String helloWorld(Model model) {
	        model.addAttribute("message", "Hello World!");
	        return "helloWorld";
	    }
	}
### 3.1使用@Controller注解定义一个控制器
### @Controller注解表明了一个类是作为控制器的角色而存在的。Spring不要求你去继承任何控制器基类，也不要求你去实现Servlet的那套API。
### 你需要在配置中加入组件扫描的配置代码来开启框架对注解控制器的自动检测。请使用下面XML代码所示的spring-context schema：	
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xmlns:p="http://www.springframework.org/schema/p"
	    xmlns:context="http://www.springframework.org/schema/context"
	    xsi:schemaLocation="
	        http://www.springframework.org/schema/beans
	        http://www.springframework.org/schema/beans/spring-beans.xsd
	        http://www.springframework.org/schema/context
	        http://www.springframework.org/schema/context/spring-context.xsd">
	
	    <context:component-scan base-package="org.springframework.samples.petclinic.web"/>
	
	    <!-- ... -->
	
	</beans>
### 3.2使用@RequestMapping注解映射请求路径
### 使用@RequestMapping注解来将请求URL，如/appointments等，映射到整个类上或某个特定的处理器方法上。一般来说，类级别的注解负责将一个特定（或符合某种模式）的请求路径映射到一个控制器上，同时通过方法级别的注解来细化映射，即根据特定的HTTP请求方法（“GET”“POST”方法等）、HTTP请求中是否携带特定参数等条件，将请求映射到匹配的方法上。

## 4.处理器异常解析器HandlerExceptionHandler
### Spring的处理器异常解析器HandlerExceptionResolver接口的实现负责处理各类控制器执行过程中出现的异常。某种程度上讲，HandlerExceptionResolver与你在web应用描述符web.xml文件中能定义的异常映射（exception mapping）很相像，不过它比后者提供了更灵活的方式。比如它能提供异常被抛出时正在执行的是哪个处理器这样的信息。并且，一个更灵活（programmatic）的异常处理方式可以为你提供更多选择，使你在请求被直接转向到另一个URL之前（与你使用Servlet规范的异常映射是一样的）有更多的方式来处理异常。
### 实现HandlerExceptionResolver接口并非实现异常处理的唯一方式，它只是提供了resolveException(Exception, Hanlder)方法的一个实现而已，方法会返回一个ModelAndView。除此之外，你还可以框架提供的SimpleMappingExceptionResolver或在异常处理方法上注解@ExceptionHandler。SimpleMappingExceptionResolver允许你获取可能抛出的异常类的名字，并把它映射到一个视图名上去。这与Servlet API提供的异常映射特性是功能等价的，但你也可以基于此实现粒度更精细的异常映射。而@ExceptionHandler注解的方法则会在异常抛出时被调用以处理该异常。这样的方法可以定义在@Controller注解的控制器类里，也可以定义在@ControllerAdvice类中，后者可以使该异常处理方法被应用到更多的@Controller控制器中。

## 5.@ExceptionHandler注解
### HandlerExceptionResolver接口以及SimpleMappingExceptionResolver解析器类的实现使得你能声明式地将异常映射到特定的视图上，还可以在异常被转发（forward）到对应的视图前使用Java代码做些判断和逻辑。不过在一些场景，特别是依靠@ResponseBody返回响应而非依赖视图解析机制的场景下，直接设置响应的状态码并将客户端需要的错误信息直接写回响应体中，可能是更方便的方法。
### 你也可以使用@ExceptionHandler方法来做到这点。如果@ExceptionHandler方法是在控制器内部定义的，那么它会接收并处理由控制器（或其任何子类）中的@RequestMapping方法抛出的异常。如果你将@ExceptionHandler方法定义在@ControllerAdvice类中，那么它会处理相关控制器中抛出的异常。下面的代码就展示了一个定义在控制器内部的@ExceptionHandler方法：
	@Controller
	public class SimpleController {
	
	    // @RequestMapping methods omitted ...
	
	    @ExceptionHandler(IOException.class)
	    public ResponseEntity<String> handleIOException(IOException ex) {
	        // prepare responseEntity
	        return responseEntity;
	    }
	
	}

### 引用自<a href="https://linesh.gitbooks.io/spring-mvc-documentation-linesh-translation/content/">Spring MVC中文文档</a>