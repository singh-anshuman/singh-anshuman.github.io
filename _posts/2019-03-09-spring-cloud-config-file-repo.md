---
layout: post
title:  "Spring Cloud Config - File Based Repository"
categories: Spring-Cloud
image:  /images/bg.jpg
---

Handling configurations for applications in a microservice based architecture can turn into a maintenance nightmare. Spring Cloud Config provides a viable solution for this problem by externalising application configurations. This post talks about how to setup Spring Config Server with file based config repositories and how to make your services can act as clients to the config server.

To demonstrate the Spring Cloud Config server setup and usage we'll be creating an application called Narrator. This is an eCommerce application that can be used to buy Audio Books. This application has microservice based architecture with 2 different sub-applications deployed separately.

* User
* Order

<img src="/images/2019-03-09-spring-cloud-config-file-repo/architecture_block_diagram.png" alt="Architecture Block Diagram"/>

Let's start by creating the Spring Cloud Config Server.

 <h2>Narrator's Spring Cloud Config Server</h2>

 Create a blank maven project in your IDE of choice. For this project I'll be using Idea intelliJ IDE. Once the project is fully setup the folder structure is going to look like this.

<img src="/images/2019-03-09-spring-cloud-config-file-repo/narrator_config_server_folder_structure.png" alt="Config Server Folder Structure"/>

Lets start by modifying pom.xml of the project and add following items.

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

After adding all these dependencies in pom.xml maven is going to fetch all the required jars for you and add them to the application's classpath. The next step now is to create the starter class which will be run to start the Spring Cloud Config server.

Lets call this class _NarratorConfigServerStarter.java_. In this class specify that the application is a Springboot application by adding the annotation _@SpringBootApplication_. To make this Springboot application a Spring Cloud Config Server add the annotation _@EnableConfigServer_.

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

Add a property file named _application.properties_ containing configurations for the config server. This property file will be considered by Springboot when the application is started. We are going to add 3 configurations to this property file.

1. Port on which the config server is going to listen for configuration requests.
2. Profile which indicates the type of configuration repository that will be used by the config server. We'll provide _native_ value to this configuration as we are going to use file based configuration repository for this post.
3. Path of the directories where configuration files are stored for client applications.  

<h3>application.properties</h3>

{% highlight properties %}

server.port=8181
spring.profiles.active=native
spring.cloud.config.server.native.search-locations=file:///Users/anshumansingh/Anshuman's_Stuff/Interview_Preparation/GIT_Downloads/narrator-config-file-repo/narrator-spring-config-file-repo/config-repo/user,file:///Users/anshumansingh/Anshuman's_Stuff/Interview_Preparation/GIT_Downloads/narrator-config-file-repo/narrator-spring-config-file-repo/config-repo/order

{% endhighlight %}

The next step now is to add the configuration repositories from which the spring cloud is going to serve configurations to the client applications.

Start by adding directories named _order_ and _user_ in the local system. Next add the files mentioned below containing properties in order and user directories. For this post we're going to have _country_ property for order application and _privacy.enabled_ property for user application.

<h3>order.properties</h3>

{% highlight properties %}
country=IN
{% endhighlight %}

Different property files for different environments can be added to the config repository. Here we are going to add another property file for order application for the _qa_ environment. Notice the naming convention of the property file.

<h3>order-qa.properties</h3>

{% highlight properties %}
country=US
{% endhighlight %}

Similarly add property files for user application for different environments for the user application.

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

Spring Cloud Config server setup is complete. Run the _NarratorConfigServerStarter_ class to start the config server.

To check if config server is up and is serving the configuration for client applications you can fire http requests using any REST api client like postman.

<h2>Http Response for User Configuration for the <i>default</i> profile</h2>
<img src="/images/2019-03-09-spring-cloud-config-file-repo/postman_default_user.png" alt="Order Client Folder Structure"/>

<h2>Http Response for User Configuration for the <i>qa</i> profile</h2>
<img src="/images/2019-03-09-spring-cloud-config-file-repo/postman_qa_user.png" alt="Order Client Folder Structure"/>

Default profile configuration will be overwritten by the profile specify configuration when the config is requested by client application. As you can see when you make a REST request directly to the config server you get configuration for both default and selected profile.  

Config server for Narrator application is ready to accept requests from client applications.

Now lets create couple of client applications for the config server. These applications are going to send requests to the config Server for configurations.

<h2>Spring Cloud Config Client - User</h2>

We'll create the user application which will act as a client for the config server. This is a REST based application with 2 layers - REST Controller layer and Service layer.

Once done the folder structure is going to look like this.

<img src="/images/2019-03-09-spring-cloud-config-file-repo/narrator_user_client_folder_structure.png" alt="User Client Folder Structure"/>

Create a new maven application and modify the pom.xml to make this application a Springboot application. Next, add the dependencies for Spring Web Starter and Spring Cloud Config Client packages.

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

Add a starter class for the application with the _public static void main(String []args)_ method which would be run to start the application.

@SpringBootApplication
public class UserManagementStarter {

    public static void main(String []args) {
        SpringApplication.run(UserManagementStarter.class,args);
    }
}

{% endhighlight %}

Add User pojo to the project. This class is going to carry the information for User entity.

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

Now lets add the REST controller to the application where we are going to define the REST mappings for User REST service.

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

Now add _UserService.java_ class containing business logic for the user application. In this class we're going to populate the value of _privacyEnabled_ field from configuration passed by config server.

<h3>UserService.java</h3>

{% highlight java %}

@Service
public class UserService {

    //  Getting the value for this field from the configuration sent by config server  
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

Add the property file containing information about the port on which the user application server is listen for requests.

<h3>application.properties</h3>

{% highlight properties %}

server.port=8182

{% endhighlight %}

Now comes the step where you'll specify the details of config server. Add a file _bootstrap.properties_ which will contain URI of config server and name of the application. This name will be used by the config server to identify the configuration set for the application. Also, in this file you specify the environment for which the configuration is to be fetched from the config server.

Configuration present in the _bootstrap.properties_ file is loaded before any other configuration.

<h3>bootstrap.properties</h3>

{% highlight properties %}

spring.profiles.active=qa
spring.application.name=user
spring.cloud.config.uri=http://localhost:8181

{% endhighlight %}

Similar to the User service we're going to create order application which is going to ask for configuration from the config server.

<h2>Spring Cloud Config Server - Order</h2>

Once done the folder structure is going to look like this.

<img src="/images/2019-03-09-spring-cloud-config-file-repo/narrator_order_client_folder_structure.png" alt="Order Client Folder Structure"/>

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

    // Getting the value of this field from configuration sent by config server
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

Now that both the clients are setup run the starter classes for both the applications. When the applications start you'll notice a log entry indicating the config server to which the request for configuration is sent by client applications.

Complete and working code for these applications can be downloaded from <a href="https://github.com/singh-anshuman/narrator-config-file-repo" target="blank">github</a>.
