---
layout: post
title:  "Spring Cloud Config - File Based Repository"
categories: Spring-Cloud
image:  /images/bg.jpg
---

Handling configurations for various applications in a microservice based architecture can turn into a maintenance nightmare. Spring Cloud Config provides a viable solution for this problem by externalising application configurations. This post talks about how to setup Spring Config Server with file based config repositories and how to make your services can act as clients to the config server.

To demonstrate the Spring Cloud Config server setup and usage we'll be creating an application called Narrator. This is an eCommerce application that can be used to buy Audio Books. This application has Microservice based architecture with 2 different sub-applications deployed separately.

* User
* Order

Let's start by creating the Spring Cloud Config Server.

 <h2>Narrator's Spring Cloud Config Server</h2>

 Create a blank maven project in your IDE of choice. For this project I'll be using Idea intelliJ IDE. Modify the pom.xml of the project and add following items.

 1. Make this a Springboot Application by adding the parent tag. At the time of writing this blog post the latest stable version of Springboot is _2.1.3.RELEASE_.
 2. Add the dependency management tag for getting the bill of material for config server.
 3. Add dependency for Spring Cloud Config Starter and the Spring Cloud Config Server.

<h3>pom.xml</h3>

{% highlight xml %}

<groupId>com.anshuman</groupId>
<artifactId>narrator-spring-config-file-repo</artifactId>
<version>1.0-SNAPSHOT</version>

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.3.RELEASE</version>
</parent>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Greenwich.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
        <version>2.0.2.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
        <version>2.0.2.RELEASE</version>
    </dependency>
</dependencies>

{% endhighlight %}

<h3>NarratorConfigServerStarter.java</h3>

{% highlight java %}

@SpringBootApplication
@EnableConfigServer
public class NarratorConfigServerStarter {

    public static void main(String []args) {
        SpringApplication.run(NarratorConfigServerStarter.class,args);
    }

}

{% endhighlight %}

<h3>application.properties</h3>

{% highlight properties %}

server.port=8181
spring.profiles.active=native
spring.cloud.config.server.native.search-locations=file:///Users/anshumansingh/Anshuman's_Stuff/Interview_Preparation/GIT_Downloads/narrator-config-file-repo/narrator-spring-config-file-repo/config-repo/user,file:///Users/anshumansingh/Anshuman's_Stuff/Interview_Preparation/GIT_Downloads/narrator-config-file-repo/narrator-spring-config-file-repo/config-repo/order

{% endhighlight %}

<h3>order.properties</h3>

{% highlight properties %}
country=IN
{% endhighlight %}

<h3>order-qa.properties</h3>

{% highlight properties %}
country=US
{% endhighlight %}

<h3>user.properties</h3>

{% highlight properties %}
privacy.enabled=y
{% endhighlight %}

<h3>user-qa.properties</h3>

{% highlight properties %}
privacy.enabled=n
{% endhighlight %}

<h3>user-uat.properties</h3>

{% highlight properties %}
privacy.enabled=y
{% endhighlight %}

<h2>Spring Cloud Config Client - User</h2>

<h3>pom.xml</h3>

{% highlight xml %}
<groupId>com.anshuman</groupId>
<artifactId>narrator-user-api-config-file-repo</artifactId>
<version>1.0-SNAPSHOT</version>

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.3.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
        <version>2.1.0.RELEASE</version>
    </dependency>
</dependencies>
{% endhighlight %}

<h3>UserManagementStarter.java</h3>

{% highlight java %}

@SpringBootApplication
public class UserManagementStarter {

    public static void main(String []args) {
        SpringApplication.run(UserManagementStarter.class,args);
    }
}

{% endhighlight %}

<h3>User.java</h3>

{% highlight java %}

@JsonInclude(JsonInclude.Include.NON_NULL)
public class User {

    private Integer userId;
    private String title;
    private String firstName;
    private String lastName;
    private String zipCode;

    //  Getters and Setters
}

{% endhighlight %}

<h3>UserController.java</h3>

{% highlight java %}
@RestController
@RequestMapping("/user-mgmt/v1")
public class UserController {

    @Autowired
    private UserService userService;

    public UserService getUserService() {
        return userService;
    }

    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/users")
    public ResponseEntity<List<User>> getAllUsers() {
        return new ResponseEntity<>(userService.getAllUsers(), HttpStatus.OK);
    }

}

{% endhighlight %}

<h3>UserService.java</h3>

{% highlight java %}

@Service
public class UserService {

    @Value("${privacy.enabled}")
    private String privacyEnabled;

    public String getPrivacyEnabled() {
        return privacyEnabled;
    }

    public void setPrivacyEnabled(String privacyEnabled) {
        this.privacyEnabled = privacyEnabled;
    }

    public List<User> getAllUsers() {
        List<User> users = new ArrayList<>();

        User u1 = new User();
        u1.setUserId(1);
        u1.setTitle("Mr.");
        u1.setFirstName("Anshuman");
        u1.setLastName("Singh");
        u1.setZipCode("y".equals(privacyEnabled) ? null : "123456");

        User u2 = new User();
        u2.setUserId(2);
        u2.setTitle("Mr.");
        u2.setFirstName("Arora");
        u2.setLastName("Anuj");
        u2.setZipCode("y".equals(privacyEnabled) ? null : "313131");

        User u3 = new User();
        u3.setUserId(2);
        u3.setTitle("Mrs.");
        u3.setFirstName("Neha");
        u3.setLastName("Upadhyay");
        u3.setZipCode("y".equals(privacyEnabled) ? null : "302033");

        users.add(u1);
        users.add(u2);
        users.add(u3);

        return users;
    }
}

{% endhighlight %}

<h3>application.properties</h3>

{% highlight properties %}

server.port=8182

{% endhighlight %}

<h3>bootstrap.properties</h3>

{% highlight properties %}

spring.profiles.active=qa
spring.application.name=user
spring.cloud.config.uri=http://localhost:8181

{% endhighlight %}

<h2>Spring Cloud Config Server - Order</h2>

<h3>pom.xml</h3>

{% highlight xml %}

<groupId>com.anshuman</groupId>
<artifactId>narrator-order-api-config-file-repo</artifactId>
<version>1.0-SNAPSHOT</version>

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.3.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
        <version>2.1.0.RELEASE</version>
    </dependency>
</dependencies>

{% endhighlight %}

<h3>OrderManagementStarter.java</h3>

{% highlight java %}

@SpringBootApplication
public class OrderManagementStarter {

    public static void main(String []args) {
        SpringApplication.run(OrderManagementStarter.class,args);
    }
}

{% endhighlight %}

<h3>Order.java</h3>

{% highlight java %}

public class Order {

    private Integer orderId;
    private Date date;
    private String value;
    private Integer userId;

    //  Getters and Setters
}

{% endhighlight %}

<h3>OrderController.java</h3>

{% highlight java %}

@RestController
@RequestMapping("/order-mgmt/v1")
public class OrderController {

    @Autowired
    private OrderService orderService;

    public OrderService getOrderService() {
        return orderService;
    }

    public void setOrderService(OrderService orderService) {
        this.orderService = orderService;
    }

    @GetMapping("orders")
    public ResponseEntity<List<Order>> getAllOrdersForUser(@RequestParam("userId") Integer userId) {
        return new ResponseEntity<>(orderService.getAllOrdersForUser(userId), HttpStatus.OK);
    }
}

{% endhighlight %}

<h3>OrderService.java</h3>

{% highlight java %}

@Service
public class OrderService {

    @Value("${country}")
    private String country;

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }

    public List<Order> getAllOrdersForUser(Integer userId) {
        List<Order> orders = new ArrayList<>();
        String currency = "IN".equals(country) ? "INR" : "$";

        Order o1 = new Order();
        o1.setUserId(1);
        o1.setDate(new Date());
        o1.setValue(currency+"100");
        o1.setOrderId(1);

        Order o2 = new Order();
        o2.setUserId(2);
        o2.setDate(new Date());
        o2.setValue(currency+"200");
        o2.setOrderId(2);

        Order o3 = new Order();
        o3.setUserId(2);
        o3.setDate(new Date());
        o3.setValue(currency+"150");
        o3.setOrderId(3);

        orders.add(o1);
        orders.add(o2);
        orders.add(o3);

        return orders.stream().filter(o->o.getUserId().equals(userId)).collect(Collectors.toList());
    }

}

{% endhighlight %}

<h3>application.properties</h3>

{% highlight properties %}

server.port=8183

{% endhighlight %}

<h3>bootstrap.properties</h3>

{% highlight properties %}

spring.profiles.active=qa
spring.application.name=order
spring.cloud.config.uri=http://localhost:8181

{% endhighlight %}
