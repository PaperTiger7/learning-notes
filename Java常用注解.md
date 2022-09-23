### Java常用注解

#### @FeignClient

属于Spring Cloud技术架构体系中的一个注解，其作用是可以让**当前服务调用其它应用**服务的**接口**.

1、引入依赖

```xml

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>2.2.4.RELEASE</version>
</dependency>
```

2、启动类里面加入注解@EnableFeignClients，声明开启Feign的远程调用

3、编写接口类，调用远程服务

```java
import java.util.List;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
 
@FeignClient(name="custorm",fallback=Hysitx.class)
public interface IRemoteCallService {
	@RequestMapping(value="/custorm/getTest",method = RequestMethod.POST)
    List<String> test(@RequestParam("names") String[] names);
}
```

#### @AfterReturning

通过@AfterReturning注解进行标注，该函数在目标函数执行完成后执行,并可以获取到目标函数最终的返回值returnVal，当目标函数没有返回值时，returnVal将返回null，必须通过returning = “returnVal”注明参数的名称而且必须与通知函数的参数名称相同。