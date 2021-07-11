#### Servlet Config

##### XML配置

目录 src/main/webapp/
```xml

<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>
 
<!-- servlet 配置   -->
    <servlet>
        <servlet-name>app1</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app1-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

<!-- servlet 路由映射 -->
    <servlet-mapping>
        <servlet-name>app1</servlet-name>
        <url-pattern>/app1/*</url-pattern>
    </servlet-mapping>

</web-app>

```

##### Java 配置

```java

@Configuration
public class MvcWebServletConfig implements WebApplicationInitializer {

    // 上传文件 配置的Resolver
    @Bean
    public MultipartResolver multipartResolver() {
        return new CommonsMultipartResolver();
    }

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        // 加载web配置
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(MvcWebConfig.class);
        // 创建和注册dispatcherServlet
        DispatcherServlet ds = new DispatcherServlet(context);
        ServletRegistration.Dynamic registration = servletContext.addServlet("springmvc", ds);
        registration.addMapping("/api/*");
        registration.setLoadOnStartup(1);

        AnnotationConfigWebApplicationContext context2 = new AnnotationConfigWebApplicationContext();
        context2.register(MvcWebMultipartConfig.class);
        DispatcherServlet dsMultipart = new DispatcherServlet(context2);
        ServletRegistration.Dynamic registrationMultipart = servletContext.addServlet("springmultipart", dsMultipart);
        registrationMultipart.addMapping("/upload/*");
        registrationMultipart.setLoadOnStartup(1);
        registrationMultipart.setMultipartConfig(new MultipartConfigElement("D:/download/tmp/uploads"));

    }
}
```

##### 另一种JAVA配置
```java

@Component
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[0];
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{MvcWebMultipartConfig.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/upload/*"};
    }

    @Override
    protected String getServletName() {
        return "multipartServlet";
    }

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {
        registration.setMultipartConfig(new MultipartConfigElement("D:/download/tmp/uploads"));
    }
}

```