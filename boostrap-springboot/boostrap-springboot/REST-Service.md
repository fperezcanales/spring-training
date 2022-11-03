## 5 The Controller
**The @RestController is the central artefact in the entire Web Tier of the RESTful API.** 
For the purpose of this post, the controller is modeling a simple REST resource - Foo:

```java
@RestController
@RequestMapping("/foos")
class FooController {
    
    @Autowired
    private IFooService service;
    
    @GetMapping
    public List<Foo> findAll() {
        return service.findAll();
    }
    
    @GetMapping(value = "/{id}")
    public Foo findById(@PathVariable("id") Long id) {
        return RestPreconditions.checkFound(service.findById(id));
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Long create(@RequestBody Foo resource) {
        Preconditions.checkNotNull(resource);
        return service.create(resource);
    }
    
    @PutMapping(value = "/{id}")
    @ResponseStatus(HttpStatus.OK)
    public void update(@PathVariable( "id" ) Long id, @RequestBody
    Foo resource) {
        Preconditions.checkNotNull(resource);
        RestPreconditions.checkNotNull(service.getById(resource.
                getId()));
        service.update(resource);
    }
    
    @DeleteMapping(value = "/{id}")
    @ResponseStatus(HttpStatus.OK)
    public void delete(@PathVariable("id") Long id) {
        service.deleteById(id);
    }
    
    /**
     * You may have noticed I’m using a straightforward, Guava style RestPreconditions utility:
     * */
    public static <T> T checkFound(T resource) {
        if (resource == null) {
            throw new MyResourceNotFoundException();
        }
        return resource;
    }
}
```
**The Controller implementation is non-public - this it because it does not need to be.**
Usually, the controller is the last in the chain of dependencies. It receives HTTP requests from the Spring front controller (the DispatcherServlet) and simply delegates them forward to a service layer.
If there's no use case where the controller has to be injected or manipulated through a direct reference, then I prefer not to declare it as public. 

**The request mappings are straightforward. AS with any controller, the actual value of the mapping, as well as de HTTP method, determine the target method for the request.**
@RequestBody will bind the parameters of the method to the body of the HTTP request, whereas @ResponseBody does the same for the response and return type.

The @RestController is a shorthan to include both @ReponseBody and the @Controller annotations in our class.

They also ensure than the resource will be marshalled and unmarshalled using the correct HTTP converter. 
Content negotiation will take place to choose which one of the active converters will be used, based mostly on the Acceptheader, although other HTTP headers may be used to determine the representation as well.

## 6 Mapping th HTTP Response 
The status codes of the HTTP response are one of the most important parts of REST service, and the subject can quickly becomes very complicated.
Getting these right can be what makes or breaks these service.

### 6.1 Unmapped Request
If Spring MVC receives a request which doesn't have a mapping, it considers the request not to be allowed and returns a 405 METHOD NOT ALLOWED back to the client.

It's also a good practice to include the Allow HTTP header when returning a 405 to the client, to specify which operation are allowed. This is the standard behavior of Spring MVC and doesn't require any additional configuration.

### 6.2 Valid Mapped Requests
For any request that does have a mapping, Spring MVC considers the request valid and responds with 200 OK if no other status code is specified otherwise.

It's because of this that the controller declares different @ResponseStatus for create, update and delete actions but not for get, which should indeed return the default 200 OK.

### 6.3 Client Error
**In the case of a client error, custom exceptions are defined and mapped to the appropiate error codes.**
Simply, throwing these exceptions from any of the layers of the web tier ensure Spring maps the corresponding status code on the HTTP response:

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class BadRequestException extends RuntimeException {
 //
}
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
 //
```

## 6.4 Using @ExceptionHandler
Another option to map custom exceptions on specific status code is to use the @ExceptionHandler annotation in the controller.
The problem with that approach is that the annotation only applies to the controller in which it's defined. This means that we need to declares in each controller individually.

Of course, there are more way to handle errors in both Spring Boot that offer more flexibility.

### 7 Additional Maven Dependencies
In addition to spring-webmvc dependency , we'll need to set up content marchalling and unmarshalling for the REST API.

```xml
<dependencies>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.8</version>
    </dependency>
    <dependency>
        <groupId>javax.xml.bind</groupId>
        <artifactId>jaxb-api</artifactId>
        <version>2.3.1</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

There are the libraries used to convert the representation of the REST resource to either JSON or XML.

### 7.1. Using Spring Boot
If we want retrieve JSON-formatted resource, Spring Boot provides supports for different libraries, namely Jackson, Gson, and JSON-B.

Auto-configuration is carried out by just including any of the mapping libraries in the classpath.
Usually, if we're developing a web application, we'll just add spring/boot-started-web dependency and rely on it include all the necesary artefacts to our project:

```xml
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-web</artifactId>
 <version>2.1.2.RELEASE</version>
</dependency>
```

Spring Boot uses Jackson by default.
If we want to serialize our resources in an XML format, we'll have to add the Jackson XML extension (jackson-dataformat-xml) to our to our dependencies, or fallback to the JAXB implementation (provided by default in the JDK) by using the @XmlRootElement annotation on our resource.

### 8. Conlusion

this chapter illustrated how implement and configure a REST Service using Spring and Java-based configuration.
