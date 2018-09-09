---
layout: post
title: springCloud--断路器
date: 2018-07-16 01:54:02
tags: springCloud
categories: springCloud
---

添加依赖：
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

### Ribbon中引入Hystrix

```
@SpringBootApplication
@EnableDiscoveryClient   
@EnableCircuitBreaker   #开启断路器功能
public class RibbonApplication {

	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(RibbonApplication.class, args);
	}

}
```

```
@RestController
public class ConsumerController {

    @Autowired
    private ComputeService computeService;

    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public String add() {
        return computeService.addService();
    }

}
```

```
@Service
public class ComputeService {

    @Autowired
    RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod = "addServiceFallback")  #指定回调方法
    public String addService() {
        return restTemplate.getForEntity("http://COMPUTE-SERVICE/add?a=10&b=20", String.class).getBody();
    }

    public String addServiceFallback() {
        return "error";
    }

}
```

### Feign使用Hystrix

```
@FeignClient(value = "compute-service", fallback = ComputeClientHystrix.class)
public interface ComputeClient {

    @RequestMapping(method = RequestMethod.GET, value = "/add")
    Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b);

}
```

```
@Component
public class ComputeClientHystrix implements ComputeClient {

    @Override
    public Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b) {
        return -9999;
    }

}
```

最后一定不要忘记的操作，在配置中加入以下代码，开启断路器功能：
```
feign:
  hystrix:
    enabled: true
```




