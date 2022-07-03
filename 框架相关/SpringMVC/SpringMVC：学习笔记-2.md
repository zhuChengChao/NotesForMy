# SpringMVC：学习笔记-2

> 后端学习ING，在学完了Java语法部分，JavaWeb知识后，开始一套组合拳来带走经典框架SSM。
>
> 此文为第三拳：SpringMVC学习笔记的 **2/2**。

## 1. 响应数据和结果视图

### 1.1 返回值分类

#### 1.1.1 字符串  

controller 方法返回字符串可以指定逻辑视图名，通过视图解析器解析为物理视图地址。

```java
// 指定逻辑视图名，经过视图解析器解析为 jsp 物理路径： /WEB-INF/pages/success.jsp
@RequestMapping("/testReturnString")
public String testReturnString() {
    System.out.println("AccountController 的 testReturnString 方法执行了");
    // 跳转到XX页面
    return "success";
}
```

#### 1.1.2 void

上一篇中，可以知道 Servlet 原始 API 可以作为控制器中方法的参数：

```java
@RequestMapping("/testReturnVoid")
public void testReturnVoid(HttpServletRequest request,HttpServletResponse response) throws Exception {
} 
```

在 controller 方法形参上可以定义 request 和 response，使用 request 或 response 指定响应结果：

1. 使用 request 转向页面，如下：  

   `request.getRequestDispatcher("/WEB-INF/pages/success.jsp").forward(request,
   response);`

2. 也可以通过 response 页面重定向：

   `response.sendRedirect("testRetrunString")`

3. 也可以通过 response 指定响应结果， 例如响应 json 数据：

   ```java
   response.setCharacterEncoding("utf-8");
   response.setContentType("application/json;charset=utf-8");
   response.getWriter().write("json 串");
   ```

#### 1.1.3 ModelAndView

ModelAndView 是 SpringMVC 提供的一个对象，该对象也可以用作控制器方法的返回值。

该对象中有两个方法：  

```java
/**
 * Add an attribute to the model.
 * @param attributeName name of the object to add to the model
 * @param attributeValue object to add to the model (never {@code null})
 * @see ModelMap#addAttribute(String, Object)
 * @see #getModelMap()
 */
public ModelAndView addObject(String attributeName, Object attributeValue) {
    // 添加模型到该对象中，在页面上可以直接通过el表达式获取，${attributeName}
    getModelMap().addAttribute(attributeName, attributeValue);
    return this;
}

/**
 * Set a view name for this ModelAndView, to be resolved by the
 * DispatcherServlet via a ViewResolver. Will override any
 * pre-existing view name or View.
 * 用于设定逻辑视图名称，视图解析器会根据名称前往指定的视图
 */
public void setViewName(@Nullable String viewName) {
	this.view = viewName;
}
```

实例代码：

```java
// 返回 ModeAndView
@RequestMapping("/testReturnModelAndView")
public ModelAndView testReturnModelAndView() {
    ModelAndView mv = new ModelAndView();
    mv.addObject("username", "张三");
    mv.setViewName("success");
    return mv;
}
```

响应的 jsp 代码：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>执行成功</title>
    </head>
    <body>
        执行成功！ ${requestScope.username}
    </body>
</html>
```

输出结果：`执行成功！张三`	

> 注意：在页面上上获取使用的是 requestScope.username 取的，所以返回 ModelAndView 类型时，浏览器跳转只能是请求转发。  

### 1.2 转发和重定向

#### 1.2.1 forward 转发

controller 方法在提供了 String 类型的返回值之后，默认就是请求转发。也可以写成：  

```java
// 转发
@RequestMapping("/testForward")
public String testForward() {
    System.out.println("AccountController 的 testForward 方法执行了");
    return "forward:/WEB-INF/pages/success.jsp";
}
```

需要注意的是，如果用了 `forward:` 则路径必须写成实际视图 url，不能写逻辑视图。

它相当于 `“ request.getRequestDispatcher("url").forward(request,response)” `。使用请求转发，既可以转发到 jsp，也可以转发到其他的控制器方法。  

#### 1.2.2 Redirect 重定向

contrller 方法提供了一个 String 类型返回值之后， 它需要在返回值里使用：`redirect:` 

```java
// 重定向
@RequestMapping("/testRedirect")
public String testRedirect() {
    System.out.println("AccountController 的 testRedirect 方法执行了");
    return "redirect:testReturnModelAndView";
}
```

它相当于 `“response.sendRedirect(url)”`。需要注意的是，如果是重定向到 jsp 页面，则 jsp 页面不能写在 WEB-INF 目录中，否则无法找到。

### 1.3 ResponseBody响应json数据

**使用说明**

* 作用：该注解用于将 Controller 的方法返回的对象，通过 HttpMessageConverter 接口转换为指定格式的数据如： json,xml 等，通过 Response 响应给客户端

**使用示例**

* 需求：使用 @ResponseBody 注解实现将 controller 方法返回对象转换为 json 响应给客户端。

* 前置知识点：Springmvc 默认用 MappingJacksonHttpMessageConverter 对 json 数据进行转换，需要导入相应依赖

  ```xml
      <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.0</version>
      </dependency>
      <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
        <version>2.9.0</version>
      </dependency>
      <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-annotations</artifactId>
        <version>2.9.0</version>
      </dependency>
  ```

* jsp 中的代码：  

```jsp
<script type="text/javascript"
        src="${pageContext.request.contextPath}/js/jquery.min.js"></script>
<script type="text/javascript">
    $(function(){
        // 绑定点击事件
        $("#testJson").click(function(){
            $.ajax({
                type:"post",
                url:"${pageContext.request.contextPath}/testResponseJson",
                contentType:"application/json;charset=utf-8",
                data:'{"id":1,"name":"test","money":999.0}',
                dataType:"json",
                success:function(data){
                    alert(data);
                }
            });
        });
    })
</script>
<!-- 测试异步请求 -->
<input type="button" value="测试ajax请求json和响应json" id="testJson"/>
```

* 控制器中的代码

```java
// 响应json数据的控制器，public后的注解能将对象解析为json数据发送，参数上的注解能自动解析json数据封装如对象中
@Controller("jsonController")
public class JsonController {
    // 测试响应 json 数据
    @RequestMapping("/testResponseJson")
    public @ResponseBody Account testResponseJson(@RequestBody Account account) {
        System.out.println("异步请求： "+account);
        return account;
    }
}
```

运行结果：

```
异步请求： Account[id=1, name=test, money=999.0]
```

## 2. SpringMVC异常处理

### 2.1 异常处理思路

系统中异常包括两类：编译器异常和运行时异常，前者通过捕获异常从而获取异常信息，后者主要通过规范代码开发、测试通过手段减少运行时异常的发生。

系统的 dao、 service、 controller 出现都通过 throws Exception 向上抛出，最后由 springmvc 前端控制器交由异常处理器进行异常处理，如下图：

![异常处理](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252305698.PNG)

### 2.2 实现步骤

#### 2.2.1 编写异常类和错误页面  

* **异常类**

```java
// 自定义异常
public class CustomException extends Exception {
    
    private String message;
    
    public CustomException(String message) {
        this.message = message;
    }
    
    public String getMessage() {
        return message;
    }
}
```

* jsp 页面：  

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
"http://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>执行失败</title>
    </head>
    <body>
        执行失败! ${message }
    </body>
</html>
```

#### 2.2.2 自定义异常处理器  

```java
// 自定义异常处理器
public class CustomExceptionResolver implements HandlerExceptionResolver {
    
    @Override
    public ModelAndView resolveException(HttpServletRequest request,
                                         HttpServletResponse response, 
                                         Object handler, Exception ex) {
        ex.printStackTrace();
        CustomException customException = null;
        //如果抛出的是系统自定义异常则直接转换
        if(ex instanceof CustomException){
            customException = (CustomException)ex;
        }else{
            //如果抛出的不是系统自定义异常则重新构造一个系统错误异常。
            customException = new CustomException("系统错误，请与系统管理员联系！ ");
        }
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("message", customException.getMessage());
        modelAndView.setViewName("error");
        return modelAndView;
    }
}
```

#### 2.2.3 配置异常处理器  

```xml
<!-- 配置自定义异常处理器 -->
<bean id="handlerExceptionResolver"
      class="cn.xyc.exception.CustomExceptionResolver"/>
```

## 3. SpringMVC拦截器

### 3.1 拦截器的作用

Spring MVC 的处理器拦截器类似于 Servlet 开发中的过滤器 Filter，用于对处理器进行预处理和后处理。

用户可以自己定义一些拦截器来实现特定的功能。

谈到拦截器，还要提及一个词：拦截器链（Interceptor Chain）。拦截器链就是将拦截器按一定的顺序联结成一条链。在访问被拦截的方法或字段时，拦截器链中的拦截器就会按其之前定义的顺序被调用。

又点类似之前的过滤器吗，是的它和过滤器是有几分相似，但是也有区别，如下：

* 过滤器是 servlet 规范中的一部分， 任何 java web 工程都可以使用。
* 拦截器是 SpringMVC 框架自己的，只有使用了 SpringMVC 框架的工程才能用。
* 过滤器在 url-pattern 中配置了 `/*`之后，可以对所有要访问的资源拦截。
* 拦截器它是只会拦截访问的控制器方法，如果访问的是 jsp / html / css / image 或者 js 是不会进行拦截的。

它也是 AOP 思想的具体应用。

要想自定义拦截器， 要求必须实现： HandlerInterceptor 接口。

### 3.2 自定义拦截器的步骤

**第一步：编写一个普通类实现 HandlerInterceptor 接口**  

```java
// 自定义拦截器
public class HandlerInterceptorDemo1 implements HandlerInterceptor {
    @Override
    public boolean preHandle(
        HttpServletRequest request, 
        HttpServletResponse response, 
        Object handler) throws Exception {
        System.out.println("preHandle 拦截器拦截了");
        return true;
    }
    
    @Override
    public void postHandle(
        HttpServletRequest request, 
        HttpServletResponse response, 
        Object handler, 
        ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle 方法执行了");
    }
    
    @Override
    public void afterCompletion(
        HttpServletRequest request, 
        HttpServletResponse response, 
        Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion 方法执行了");
    }
}
```

**第二步：配置拦截器**  

```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean id="handlerInterceptorDemo1"
              class="cn.xyc.web.interceptor.HandlerInterceptorDemo1"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

**测试运行结果：**  

```
preHandle 拦截器拦截了
控制器中的方法执行了
postHandle 方法执行了
afterCompletion 方法执行了
```

### 3.3 拦截器的细节

#### 3.3.1 拦截器的放行

放行的含义是指，如果有下一个拦截器就执行下一个，如果该拦截器处于拦截器链的最后一个，则执行控制器中的方法。 

#### 3.3.2 拦截器中方法的说明  

```java
public interface HandlerInterceptor {
    /**
     * 如何调用：按拦截器定义顺序调用
     * 何时调用：是controller方法执行前拦截的方法
     * 有什么用：
     *   如果程序员决定该拦截器对请求进行拦截处理后还要调用其他的拦截器，或者是业务处理器去进行处理，则返回 true。
     *   如果程序员决定不需要再调用其他的组件去处理请求，则返回 false。
     */
    default boolean preHandle(HttpServletRequest request, 
                              HttpServletResponse response, 
                              Object handler) throws Exception {
        return true;
    }
    
    /**
     * 如何调用：按拦截器定义逆序调用
     * 何时调用：在controller方法执行后执行的方法，在JSP视图执行前
     * 有什么用：
     * 	 在业务处理器处理完请求后，但是DispatcherServlet向客户端返回响应前被调用，
     * 	 在该方法中对用户请求 request 进行处理。
     */
    default void postHandle(HttpServletRequest request, 
                            HttpServletResponse response, 
                            Object handler,
                            @Nullable ModelAndView modelAndView) throws Exception {
    }
    
    /**
	 * 如何调用：按拦截器定义逆序调用
     * 何时调用：是在JSP执行后才执行
     * 有什么用：
     * 		在DispatcherServlet完全处理完请求后被调用，
     * 		可以在该方法中进行一些资源清理的操作。
     */
    default void afterCompletion(HttpServletRequest request,
                                 HttpServletResponse response, 
                                 Object handler,
                                 @Nullable Exception ex) throws Exception {
    }
}
```

此时如果有多个拦截器，这时拦截器 1 的 preHandle 方法返回 true，但是拦截器 2 的 preHandle 方法返回 false，而此时拦截器 1 的 afterCompletion 方法是否执行？  

#### 3.3.3 拦截器的作用路径

作用路径可以通过在配置文件中配置。  

```xml
<!-- 配置拦截器的作用范围 -->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**" /><!-- 用于指定对拦截的 url -->
        <mvc:exclude-mapping path=""/><!-- 用于指定排除的 url-->
        <bean id="handlerInterceptorDemo1"
              class="cn.xyc.web.interceptor.HandlerInterceptorDemo1"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 3.3.4 多个拦截器的执行顺序

多个拦截器是按照配置的顺序决定的，如下：

![多个拦截器执行](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252305530.PNG)

### 3.4 正常流程测试

#### 3.4.1 配置文件  

```xml
<!-- 配置拦截器的作用范围 -->
<mvc:interceptors>
    
    <!-- 拦截器1 -->
    <mvc:interceptor>
        <mvc:mapping path="/**" /><!-- 用于指定对拦截的 url -->
        <bean id="handlerInterceptorDemo1" class="cn.xyc.web.interceptor.HandlerInterceptorDemo1"></bean>
    </mvc:interceptor>
    
    <!-- 拦截器2 -->
    <mvc:interceptor>
        <mvc:mapping path="/**" />
        <bean id="handlerInterceptorDemo2"
              class="cn.xyc.web.interceptor.HandlerInterceptorDemo2"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 3.4.2 拦截器1

```java
//自定义拦截器：拦截器1
public class HandlerInterceptorDemo1 implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("拦截器 1： preHandle 拦截器拦截了");
        return true;
    }
    
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器 1： postHandle 方法执行了");
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("拦截器 1： afterCompletion 方法执行了");
    }
}
```

#### 3.4.3 拦截器2

```java
//自定义拦截器：拦截器2
public class HandlerInterceptorDemo2 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("拦截器 2： preHandle 拦截器拦截了");
        return true;
    }
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器 2： postHandle 方法执行了");
    }
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse
                                response, Object handler, Exception ex) throws Exception {
        System.out.println("拦截器 2： afterCompletion 方法执行了");
    }
}
```

#### 3.4.4 运行结果

```
拦截器 1： preHandle 拦截器拦截了
拦截器 2： preHandle 拦截器拦截了
控制器中的方法执行了
拦截器 2： postHandle 方法执行了
拦截器 1： postHandle 方法执行了
拦截器 2： afterCompletion 方法执行了
拦截器 2： afterCompletion 方法执行了
```

### 3.5 中断流程测试

#### 3.5.1 配置文件

```xml
<!-- 配置拦截器的作用范围 -->
<mvc:interceptors>
    <!-- 拦截器1 -->
    <mvc:interceptor>
        <mvc:mapping path="/**" /><!-- 用于指定对拦截的 url -->
        <bean id="handlerInterceptorDemo1"
              class="cn.xyc.web.interceptor.HandlerInterceptorDemo1"></bean>
    </mvc:interceptor>
    <!-- 拦截器2 -->
    <mvc:interceptor>
        <mvc:mapping path="/**" />
        <bean id="handlerInterceptorDemo2"
              class="cn.xyc.web.interceptor.HandlerInterceptorDemo2"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 3.5.2 拦截器1

```java
// 自定义拦截器拦截器1
public class HandlerInterceptorDemo1 implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse
                             response, Object handler) throws Exception {
        System.out.println("拦截器 1： preHandle 拦截器拦截了");
        return true;
    }
    
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器 1： postHandle 方法执行了");
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("拦截器 1： afterCompletion 方法执行了");
    }
}
```

#### 3.5.3 拦截器2

```java
// 自定义拦截器:拦截器2
public class HandlerInterceptorDemo2 implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("拦截器 2： preHandle 拦截器拦截了");
        return false;  // 注意这里
    }
    
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器 2： postHandle 方法执行了");
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("拦截器 2： afterCompletion 方法执行了");
    }
}
```

#### 3.5.4 运行结果

```
拦截器 1： preHandle 拦截器拦截了
拦截器 2： preHandle 拦截器拦截了
拦截器 1： afterCompletion 方法执行了
```

### 3.6 拦截器的简单案例

**简单案例**：验证用户是否登录

**实现思路**

1. 有一个登录页面，需要写一个 controller 访问页面
2. 登录页面有一提交表单的动作。需要在 controller 中处理。
   1. 判断用户名密码是否正确
   2. 如果正确 向 session 中写入用户信息
   3. 返回登录成功。
3. 拦截用户请求，判断用户是否登录
   1. 如果用户已经登录。放行
   2. 如果用户未登录，跳转到登录页面

**控制器代码**  

```java
//登陆页面
@RequestMapping("/login")
public String login(Model model)throws Exception{
    return "login";
}

//登陆提交
//userid：用户账号， pwd：密码
@RequestMapping("/loginsubmit")
public String loginsubmit(HttpSession session, String userid, String pwd)throws Exception{
    //向 session 记录用户身份信息
    session.setAttribute("activeUser", userid);
    return "redirect:/main.jsp";
}

//退出
@RequestMapping("/logout")
public String logout(HttpSession session)throws Exception{
    //session 过期
    session.invalidate();
    return "redirect:index.jsp";
}
```

**拦截器代码**  

```java
public class LoginInterceptor implements HandlerInterceptor{
    @Override
    Public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response, 
                             Object handler) throws Exception {
        //如果是登录页面则放行
        if(request.getRequestURI().indexOf("login.action")>=0){
            return true;
        }
        HttpSession session = request.getSession();
        //如果用户已登录也放行
        if(session.getAttribute("user")!=null){
            return true;
        }
        //用户没有登录挑战到登录页面
        request.getRequestDispatcher(
            "/WEB-INF/jsp/login.jsp").forward(request, response);
        return false;
    }
}
```
