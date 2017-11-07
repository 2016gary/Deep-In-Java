# Spring MVC
### Spring MVC框架，与其他很多web的MVC框架一样：请求驱动；所有设计都围绕着一个中央Servlet来展开，它负责把所有请求分发到控制器；同时提供其他web应用开发所需要的功能。不过Spring的中央处理器，DispatcherServlet，能做的比这更多。它与Spring IoC容器做到了无缝集成，这意味着，Spring提供的任何特性，在Spring MVC中你都可以使用。
### Spring的模型-视图-控制器（MVC）框架是围绕一个DispatcherServlet来设计的，这个Servlet会把请求分发给各个处理器，并支持可配置的处理器映射、视图渲染、本地化、时区与主题渲染等，甚至还能支持文件上传。处理器是你的应用中注解了@Controller和@RequestMapping的类和方法，Spring为处理器方法提供了极其多样灵活的配置。Spring 3.0以后提供了@Controller注解机制、@PathVariable注解以及一些其他的特性，你可以使用它们来进行RESTful web站点和应用的开发。

## 1.DispatcherServlet
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


### 引用自<a href="https://linesh.gitbooks.io/spring-mvc-documentation-linesh-translation/content/">Spring MVC中文文档</a>