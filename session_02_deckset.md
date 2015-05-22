slidenumbers: true

# [fit] Architecting for
# [fit] Continuous Delivery
# [fit] with
# [fit] Microservices
![](https://raw.githubusercontent.com/spring-projects/spring-cloud/gh-pages/img/project-icon-large.png)

---

# [fit] Session
# [fit] Two
![](Common/images/cf_logo.png)

---

# [fit] Building  
# [fit] and Composing
# [fit] Microservices

---

# [fit] Building
# [fit] Twelve-Factor
# [fit] Apps
# [fit] with Spring Boot

---

![](Common/images/12factor.png)
# [fit] http://12factor.net

---

# Patterns

- Cloud-native application architectures
- Optimized for speed, safety, & scale
- Declarative configuration
- Stateless/shared-nothing processes
- Loose coupling to application environment

---

# Twelve Factors (1/2)

- One Codebase in Version Control
- Explicit Dependencies
- Externalized Config
- Attached Backing Services
- Separate Build, Release, and Run Stages
- Stateless, Shared-Nothing Processes

---

# Twelve Factors (2/2)

- Export Services via Port binding
- Scale Out Horizontally for Concurrency
- Instances Should Be Disposable
- Dev/Prod Parity
- Logs Are Event Streams
- Admin Processes

---

![fit](Common/images/heroku.png)
# [fit] http://heroku.com

---

![125%](Common/images/cloud_foundry.png)
# [fit] http://cloudfoundry.org

---

![filter](Common/images/dropwizard.png)
![](Common/images/spring-boot.png)
# Microframeworks

- Dropwizard ([http://www.dropwizard.io/](http://www.dropwizard.io/))
- Spring Boot ([http://projects.spring.io/spring-boot/](http://projects.spring.io/spring-boot/))

---

# Spring Boot
- [http://projects.spring.io/spring-boot](http://projects.spring.io/spring-boot)
- Opinionated convention over configuration
- Production-ready Spring applications
- Embed Tomcat, Jetty or Undertow
- *STARTERS*
- Actuator: Metrics, health checks, introspection

---

# [http://start.spring.io](start.spring.io)
![](Common/images/start_spring.png)

---

# [fit] Writing a Single Service is
# [fit] Nice...

---

# [fit] But No Microservice
# [fit] is an Island
![](Common/images/island-house.jpg)

---

# Challenges of Distributed Systems

* Configuration Management
* Service Registration & Discovery
* Routing & Load Balancing
* Fault Tolerance (Circuit Breakers!)
* Monitoring

---

![](Common/images/netflix_oss.jpeg)

---

![](https://raw.githubusercontent.com/spring-projects/spring-cloud/gh-pages/img/project-icon-large.png)
# [fit] Spring Cloud
# [fit] Distributed System Patterns FTW!

---

![](https://raw.githubusercontent.com/spring-projects/spring-cloud/gh-pages/img/project-icon-large.png)
# [fit] Configuration
# [fit] Management

---

# [fit] Distributed?

---

![](https://raw.githubusercontent.com/spring-projects/spring-cloud/gh-pages/img/project-icon-large.png)
# [fit] Config
# [fit] Server!

---

![inline fit](Common/images/Config_Server.png)

---

# Config Server `app.groovy`

```java
@Grab("org.springframework.cloud:spring-cloud-starter-bus-amqp:1.0.0.RC1")
@EnableConfigServer
class ConfigServer {
}
```

---

# Config Server `application.yml`

```
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/mstine/config-repo.git
```

---

![](Common/images/github.jpeg)

# `https://github.com/mstine/config-repo/blob/master/demo.yml`

```
greeting: Bonjour
```

---

# Config Client `app.groovy`

```java
@Grab("org.springframework.cloud:spring-cloud-starter-bus-amqp:1.0.0.RC1")
@RestController
class BasicConfig {

  @Autowired
  Greeter greeter

  @RequestMapping("/")
  String home() {
    "${greeter.greeting} World!"
  }
}

@Component
@RefreshScope
class Greeter {

  @Value('${greeting}')
  String greeting

}
```

---

# Config Client `bootstrap.yml`

```
spring:
  application:
    name: demo
```

---

![right fit](Common/images/rabbitmq.png)
# [fit] Cloud
# [fit] Bus!

---

![inline fit](Common/images/Cloud_Bus.png)

---

# [fit] `curl -X POST http://localhost:8080/bus/refresh`

---

![](https://raw.githubusercontent.com/spring-projects/spring-cloud/gh-pages/img/project-icon-large.png)
# [fit] Service
# [fit] Registration &
# [fit] Discovery

---

![inline fit](Common/images/Service_Registry.png)

---

# [fit] Eureka
![](Common/images/netflix_oss.jpeg)

---

# [fit] Producer
# [fit] Consumer

---

# Eureka Service Registry

```java
@GrabExclude("ch.qos.logback:logback-classic")
@EnableEurekaServer
class Eureka {
}
```

---

# Producer

```java
@EnableDiscoveryClient
@RestController
public class Application {

  int counter = 0

  @RequestMapping("/")
  String produce() {
    "{\"value\": ${counter++}}"
  }
}
```

---

# Consumer

```java
@EnableDiscoveryClient
@RestController
public class Application {

  @Autowired
  DiscoveryClient discoveryClient

  @RequestMapping("/")
  String consume() {
    InstanceInfo instance = discoveryClient.getNextServerFromEureka("PRODUCER", false)

    RestTemplate restTemplate = new RestTemplate()
    ProducerResponse response = restTemplate.getForObject(instance.homePageUrl, ProducerResponse.class)

    "{\"value\": ${response.value}}"
  }
}

public class ProducerResponse {
  Integer value
}
```

---

![](https://raw.githubusercontent.com/spring-projects/spring-cloud/gh-pages/img/project-icon-large.png)
# [fit] Routing &
# [fit] Load Balancing

---

![inline fit](Common/images/Ribbon.png)

---

# [fit] Ribbon
![](Common/images/netflix_oss.jpeg)

---

# Consumer with Load Balancer

```java
@Autowired
LoadBalancerClient loadBalancer

@RequestMapping("/")
String consume() {
  ServiceInstance instance = loadBalancer.choose("producer")
  URI producerUri = URI.create("http://${instance.host}:${instance.port}");

  RestTemplate restTemplate = new RestTemplate()
  ProducerResponse response = restTemplate.getForObject(producerUri, ProducerResponse.class)

  "{\"value\": ${response.value}}"
}
```

---

# Consumer with Ribbon-enabled `RestTemplate`

```java
@Autowired
RestTemplate restTemplate

@RequestMapping("/")
String consume() {
  ProducerResponse response = restTemplate.getForObject("http://producer", ProducerResponse.class)

  "{\"value\": ${response.value}}"
}
```

---

# Feign Client

```java
@FeignClient("producer")
public interface ProducerClient {

  @RequestMapping(method = RequestMethod.GET, value = "/")
  ProducerResponse getValue();
}
```

---

# Consumer with Feign Client

```java
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
@RestController
public class Application {

  @Autowired
  ProducerClient client;

  @RequestMapping("/")
  String consume() {
    ProducerResponse response = client.getValue();

    return "{\"value\": " + response.getValue() + "}";
  }

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

---

![](https://raw.githubusercontent.com/spring-projects/spring-cloud/gh-pages/img/project-icon-large.png)
# [fit] Fault
# [fit] Tolerance

---

# [fit] Hystrix
![](Common/images/netflix_oss.jpeg)

---

# Circuit Breaker
![inline](Common/images/circuit-breaker.gif)

---

# Consumer `app.groovy`

```java
@EnableDiscoveryClient
@EnableCircuitBreaker
@RestController
public class Application {

  @Autowired
  ProducerClient client

  @RequestMapping("/")
  String consume() {
    ProducerResponse response = client.getProducerResponse()

    "{\"value\": ${response.value}}"
  }

}
```

---

# Producer Client

```java
@Component
public class ProducerClient {

  @Autowired
  RestTemplate restTemplate

  @HystrixCommand(fallbackMethod = "getProducerFallback")
  ProducerResponse getProducerResponse() {
    restTemplate.getForObject("http://producer", ProducerResponse.class)
  }

  ProducerResponse getProducerFallback() {
    new ProducerResponse(value: 42)
  }
}
```

---

![](https://raw.githubusercontent.com/spring-projects/spring-cloud/gh-pages/img/project-icon-large.png)
# [fit] Monitoring

---

# [fit] Hystrix
# [fit] Dashboard
![](Common/images/netflix_oss.jpeg)

---

# Hystrix Dashboard

![inline](Common/images/hystrix-dashboard.png)

---

# Hystrix Dashboard

```java
@Grab("org.springframework.cloud:spring-cloud-starter-hystrix-dashboard:1.0.0.RC1")

import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard

@EnableHystrixDashboard
class HystrixDashboard {
}
```

---

# [fit] TO THE
# [fit] LABS!
