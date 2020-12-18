---
title: 关于使用@CrossOrigin遇见的的一些问题和理解
date: 2019-12-06 00:57:09
categories: 笔记
tags: [Spring]
type: posts
description: 关于解决跨域遇到的一些问题
---
### 记录今天遇到的一个问题
> 个人理解 参考需谨慎。如果有错误，谢大佬指点

是关于跨域的，在昨天写项目的时候遇到的跨域请求的问题，网上通常的配置方法都是使用 过滤器，或者 实现 `WebMvcConfigurer` 接口的方法。相对来说有些老了。不符合我想要的效果。
```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS")
                .allowCredentials(true)
                .maxAge(3600)
                .allowedHeaders("*");
    }
}
```

然后我在阅读 Spring 官网 看到有使用 `@CrossOrigin` 方式可以只需要加一个注解就行(简单粗暴我喜欢)。于是我便采用的这种方法。

```java
@RestController
@CrossOrigin(value = "*",allowCredentials = "true")
@RequestMapping("user")
public class UserController {
    @PostMapping(value = "login")
    public Map<String, Object> login(@RequestBody User user) {
        Map<String, Object> map = new HashMap<>();
        map.put("userName",user.getUserName());
        map.put("userPass",user.getUserPass());
        return map;
    }
    @PostMapping(value = "logout")
    public Map logout(@RequestBody String token) {
        Map<String, String> map = new HashMap<>();
        map.put("token",token);
        return map;
    }
}
```
配置之后问题就来了在访问 login 接口的时候可以正常访问，但是 请求 logout接口的时候，还是出现的跨域的问题。

在反复检查配置，阅读官网后。尝试了上面的方法配置，完全没问题 (那我代码应该没问题 2333),但是换回  `@CrossOrigin` 依旧请求失败。

由于那种配置方法比较死，不是很灵活 (我不喜欢)。所以我想继续使用 `@CrossOrigin` 的方式配置。

于是便把部分代码抽取出来，单独建一个干净的项目复现刚才的问题。

按照 Spring 官网的配置方法一路通畅。于是对比这个 demo 和我原先的 项目,便发现了原因 ！！！ 

因为我配置了一个登录拦截器，用于验证 token 。拦截器不拦截 login 方法。拦截需要验证 token 的方法，所以就出现了部分有效，部分无效的问题。
所有经过了拦截器的方法都无法访问。


好，知道了问题的所在，让我继续翻文档。。。。

了解到。`@CrossOrigin`  这个注解本质也是一个拦截器,往response里添加  `Access-Control-Allow-Origin` 等响应头信息。
如果在 controller 类上或在方法上标了 `@CrossOrigin` 注解，则 spring 在记录 mapping 映射时会记录对应跨域请求映射。

当一个跨域请求过来时，spring    在获取 handler 时会判断这个请求是否是一个跨域请求，如果是，则会返回一个可以处理跨域的 handler

QAQ 这时候找到 `HandlerMapping` 里的 ` getCorsHandlerExecutionChain `方法。

```java
	protected HandlerExecutionChain getCorsHandlerExecutionChain(HttpServletRequest request,
			HandlerExecutionChain chain, @Nullable CorsConfiguration config) {

		if (CorsUtils.isPreFlightRequest(request)) {
			HandlerInterceptor[] interceptors = chain.getInterceptors();
			chain = new HandlerExecutionChain(new PreFlightHandler(config), interceptors);
		}
		else {
		// 写死了 index 0 emmmm
			chain.addInterceptor(0, new CorsInterceptor(config));
		}
		return chain;
	}
```
找到 `CorsInterceptor`的 `preHandle`打上断电点，和自己的拦截器打上断点(自己的先执行，凉凉)

好像 Spring 也没给配置拦截器执行顺序的东东（菜鸡的我不知道）

到此可以知道，`@CrossOrigin` 是在 spring 映射 handler 的时候 干的活。那既然拦截器不行，那我就用 Spring AOP 吧 QWQ !!!
或者在拦截器中设置，只要是预请求（OPTIONS）全部直接放行
