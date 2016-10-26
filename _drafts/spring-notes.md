---
layout: 
title: spring-notes
categories: []

---

# RestTemplate
spring框架提供的专门用于处理客户端http访问相关的类.简化与http服务器通信并遵循RESTful原则.

* 使用

post表单

{% highlight java %}
RestTemplate restTemplate = new RestTemplate();
MultiValueMap<String, String> values = new LinkedMultiValueMap<>();
values.add("userId", "1");
values.add("duration", "1000");
values.add("type", "2");
values.add("outputsResolution", "1280x720,320x240,1280x720");
ResponseResult res = restTemplate.postForObject("http://xxxx", values, ResponseResult.class);
{% endhighlight %}

post表单并设置http头信息

{% highlight java %}
RestTemplate restTemplate = new RestTemplate();
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
MultiValueMap<String, String> values = new LinkedMultiValueMap<>();
values.add("userId", "1");
values.add("duration", "1000");
values.add("type", "2");
values.add("outputsResolution", "1280x720,320x240,1280x720");
HttpEntity<MultiValueMap<String, String>> requestEntity = new HttpEntity<>(values, headers);
ResponseResult res = restTemplate.postForObject("http://xxxx", requestEntity, ResponseResult.class);
{% endhighlight %}

**注意**
使用表单提交时,如果MultiValueMap的值类型不是String时将会出现异常

{% highlight java %}
MultiValueMap<String, Object> values = new LinkedMultiValueMap<>();
values.add("name", "john");
values.add("id", 1)
ResponseResult res = restTemplate.postForObject("http://xxxx", values, ResponseResult.class);
{% endhighlight %}
上面的代码执行后,将出现以下异常

	HttpMessageNotWritableException: Could not write request: no suitable HttpMessageConverter found for request type [java.lang.Integer]

通过异常信息分析是由于没有用于Integer类型的HttpMessageConverter导致的.在FormHttpMessageConverter的文档描述指出,读写普通的html表单时使用MultiValueMap<String,String>,multipart form(包含文件数据)时使用MultiValueMap<String,Object>.

* RestTemplateHelper

{% highlight java %} 
@Component
@SuppressWarnings("all")
public class RestTemplateHelper {

    private static final Logger logger = LoggerFactory.getLogger(RestTemplateHelper.class);

    private final RestTemplate restTemplate;

    @Autowired
    public RestTemplateHelper(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public <R extends Response> R get(String url, ParameterizedTypeReference ptr, Object... uriVariables) {
        return get(url, ptr, null, uriVariables);
    }

    public <R extends Response> R get(String url, ParameterizedTypeReference ptr, Map<String, Object> parameters,
                                      Object... uriVariables) {
        UriComponentsBuilder components = UriComponentsBuilder.fromHttpUrl(url);
        if (parameters != null) {
            for (Map.Entry<String, Object> e: parameters.entrySet()) {
                components.queryParam(e.getKey(), e.getValue());
            }
        }
        return get(components.build().toUriString(), null, ptr, uriVariables);
    }

    private <R extends Response> R get(String url, HttpEntity entity, ParameterizedTypeReference ptr,
                                       Object... uriVariables) {
        logRequest(url);
        return (R) restTemplate.exchange(url, HttpMethod.GET, entity, ptr, uriVariables).getBody();
    }

    public <R extends Response> R post(String url, Map<String, Object> parameters, ParameterizedTypeReference ptr,
                                       Object... uriVariables) {
        logRequest(url);
        return (R) restTemplate.exchange(url, HttpMethod.POST, createEntity(parameters), ptr, uriVariables).getBody();
    }

    private HttpEntity createEntity(Map<String, Object> parameters) {
        HttpHeaders headers = new HttpHeaders();
        if (parameters != null) {
            LinkedMultiValueMap<String, Object> params = new LinkedMultiValueMap<>();
            for (Map.Entry<String, Object> e: parameters.entrySet()) {
                params.add(e.getKey(), e.getValue() + "");
            }
            return new HttpEntity(params, headers);
        }

        return new HttpEntity(headers);
    }

    private void logRequest(String url) {
        logger.info("Request url: {}", url);
    }
}
{% endhighlight %}

# @Value
使用@Value设置默认值

* 使用spring的Elvis operator设置默认值

{% highlight java %}
#{expression?:default value}
{% endhighlight %}

示例

{% highlight java %}
@Value("#{systemProperties['mongodb.port'] ?: 27017}")
private String port;

@Value("#{config['mongodb.url'] ?: '127.0.0.1'}")
private String url;	

@Value("#{aBean.age ?: 21}")
private int age;
{% endhighlight %}

* 使用Property placeholder设置默认值

{% highlight java %}
${property:default value}
{% endhighlight %}

示例

{% highlight java %}
@Value("${mongodb.url:127.0.0.1}")
private String mongodbUrl;

@Value("#{'${mongodb.url:172.0.0.1}'}")
private String mongodbUrl;

@Value("#{config['mongodb.url']?:'127.0.0.1'}")
private String mongodbUrl;
{% endhighlight %}

使用Property placeholder时必须注册PropertySourcesPlaceholderConfigurer,否则spring无法解析${}.

{% highlight java %}
@Bean
public static PropertySourcesPlaceholderConfigurer propertyConfigIn() {
return new PropertySourcesPlaceholderConfigurer();
}
{% endhighlight %}

当需要设置多个值时,通过逗号分隔并将属性类型设置为数组

{% highlight java %}
@Value("${aliyun.bucketNames}")
private String[] bucketNames;
{% endhighlight %}

{% highlight java %}
aliyun.bucketNames=stg-bj-hsy-images,stg-bj-hsy-live,stg-bj-hsy-public,stg-bj-hsy-vod
{% endhighlight %}

# spring boot 程序指定main class

1. 通过properties指定

{% highlight xml %}
<properties>
    <!-- The main class to start by executing java -jar -->
    <start-class>com.mycorp.starter.HelloWorldApplication</start-class>
</properties>
{% endhighlight %}

2. 通过spring-boot-maven-plugin插件的configuration指定

{% highlight xml %}
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>1.1.3.RELEASE</version>
    <configuration>
        <mainClass>my.package.MyStartClass</mainClass>
        <layout>ZIP</layout>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

# 注册PropertySourcesPlaceholderConfigurer
当使用@Value或${}注入属性值时,需要注册PropertySourcesPlaceholderConfigurer,在xml中可以通过<context:property-placeholder>实现.使用注解配置时,可以通过注册一个PropertySourcesPlaceholderConfigurer bean来实现.

{% highlight java %}
@Bean
public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
    return new PropertySourcesPlaceholderConfigurer();
}
{% endhighlight %}

# mvc requestMapping参数

因spring解析url参数时，将参数名和值放在treemap中，会根据参数名自然排序，然后对属性进行反射设置值操作。所以，当参数为pojo对象时，需要注意避免在set方法内依赖另一属性的值设置值，否则将导致结果不一致的情况。