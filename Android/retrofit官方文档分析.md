待完善[Retrofit](https://github.com/square/retrofit)：改进、更新的意思，作用有点类似于listview的适配器adapter，用来配置HTTP Client，帮你调用它去请求网络
==========================

##基本使用
- Type-safe HTTP client for Android and Java by Square, Inc.
Square公司为Android和Java设计的一个安全的HTTP客户端。
- For more information please see [the website](http://square.github.io/retrofit/).

- Introduction
Retrofit turns your HTTP API into a Java interface.
Retrofit 把你的HTTP API转换成Java接口，接口是一个规范，我们很多程序都是面向接口编程的。
```
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```
定义了一个listRepos方法来请求API，Retrofit 通过anotation(注解)方式来标注这个方法是通过GET请求，后面跟着相对的URL地址。这里返回了一个List<Repo>，省去了Json解析的步骤。@Path是动态参数。

- The Retrofit class generates an implementation of the GitHubService interface.
```
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();
GitHubService service = retrofit.create(GitHubService.class);
```
建造者模式传入baseUrl构建出Retrofit ，然后使用create方法通过代理的方式来把这个接口给创建出来。

- Each Call from the created GitHubService can make a synchronous or asynchronous HTTP request to the remote webserver.
```
Call<List<Repo>> repos = service.listRepos("octocat");
```
然后我们通过service来调用方法即可返回结果。

##继续向下看对整个文档进行了解+
- API Declaration
Annotations on the interface methods and its parameters indicate how a request will be handled.

- REQUEST METHOD
Every method must have an HTTP annotation that provides the request method and relative URL. There are five built-in annotations: GET, POST, PUT, DELETE, and HEAD. The relative URL of the resource is specified in the annotation.
每一个方法都必须有一个HTTP annotation去提供request method和relative URL。
```
//没有参数
@GET("users/list")
```
You can also specify query parameters in the URL.
```
//拼接url传参
@GET("users/list?sort=desc")
```
- URL MANIPULATION
A request URL can be updated dynamically using replacement blocks and parameters on the method. A replacement block is an alphanumeric string surrounded by { and }. A corresponding parameter must be annotated with @Path using the same string.
```
//通过@Path的方式去传参，参数是拼接在url里面的，是url的一部分,注意url需要{}占位符
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);
```
Query parameters can also be added.
```
//会构建url地址类似 users/list?sort=desc
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);
```
For complex query parameter combinations a Map can be used.
```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);
```

- REQUEST BODY
An object can be specified for use as an HTTP request body with the @Body annotation.
```
//POST请求有两种方式，一种是提交form表单的方式，另一种是提交json
//POST请求传参传一个Body。@Body代表是把User对象自动转换成json字符串，然后把他当做是一个参数进行传递
@POST("users/new")
Call<User> createUser(@Body User user);
```
The object will also be converted using a converter specified on the Retrofit instance. If no converter is added, only RequestBody can be used.

- FORM ENCODED AND MULTIPART
Methods can also be declared to send form-encoded and multipart data.
Form-encoded data is sent when @FormUrlEncoded is present on the method. Each key-value pair is annotated with @Field containing the name and the object providing the value.
```
//Form表单的形式
@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);
```
Multipart requests are used when @Multipart is present on the method. Parts are declared using the @Part annotation.
```
//Multipart多种类型参数的形式，是指文件或文本信息等
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
```
Multipart parts use one of Retrofit's converters or they can implement RequestBody to handle their own serialization.

- HEADER MANIPULATION
You can set static headers for a method using the @Headers annotation.
```
//通过增加@Headers修改或配置我们的Headers
@Headers("Cache-Control: max-age=640000")
@GET("widget/list")
Call<List<Widget>> widgetList();
```
```
@Headers({
    "Accept: application/vnd.github.v3.full+json",
    "User-Agent: Retrofit-Sample-App"
})
```
```
@GET("users/{username}")
Call<User> getUser(@Path("username") String username);
```
Note that headers do not overwrite each other. All headers with the same name will be included in the request.
A request Header can be updated dynamically using the @Header annotation. A corresponding parameter must be provided to the @Header. If the value is null, the header will be omitted. Otherwise, toString will be called on the value, and the result used.
```
//Header也可以通过这种方式直接加进来
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)
```
Headers that need to be added to every request can be specified using an [OkHttp interceptor](https://github.com/square/okhttp/wiki/Interceptors).
上述这样配置Headers需要给每一个请求都添加，不想这样做可以通过OkHttp interceptor


- SYNCHRONOUS VS. ASYNCHRONOUS
同步和异步
Call instances can be executed either synchronously or asynchronously. Each instance can only be used once, but calling clone() will create a new instance that can be used.
On Android, callbacks will be executed on the main thread. On the JVM, callbacks will happen on the same thread that executed the HTTP request.


- Retrofit Configuration
Retrofit is the class through which your API interfaces are turned into callable objects. By default, Retrofit will give you sane defaults for your platform but it allows for customization.

- CONVERTERS
By default, Retrofit can only deserialize HTTP bodies into OkHttp's ResponseBody type and it can only accept its RequestBody
 type for @Body
Retrofit是把HTTP的bodies转换成OkHttp's ResponseBody，是用下面这些东西进行转换的，需要自行配置。有点类似于适配器的风格。
Converters can be added to support other types. Six sibling modules adapt popular serialization libraries for your convenience.
[Gson](https://github.com/google/gson): com.squareup.retrofit2:converter-gson
[Jackson](http://wiki.fasterxml.com/JacksonHome): com.squareup.retrofit2:converter-jackson
[Moshi](https://github.com/square/moshi/): com.squareup.retrofit2:converter-moshi
[Protobuf](https://developers.google.com/protocol-buffers/): com.squareup.retrofit2:converter-protobuf
[Wire](https://github.com/square/wire): com.squareup.retrofit2:converter-wire
[Simple XML](http://simple.sourceforge.net/): com.squareup.retrofit2:converter-simplexml
Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars
Here's an example of using the GsonConverterFactory
 class to generate an implementation of the GitHubService
 interface which uses Gson for its deserialization.
```
Retrofit retrofit = new Retrofit.Builder()
 .baseUrl("https://api.github.com") 
//使用Gson把@Body的User转换成Json和转换结果
.addConverterFactory(GsonConverterFactory.create()) 
.build();
GitHubService service = retrofit.create(GitHubService.class);
```

- CUSTOM CONVERTERS
If you need to communicate with an API that uses a content-format that Retrofit does not support out of the box (e.g. YAML, txt, custom format) or you wish to use a different library to implement an existing format, you can easily create your own converter. Create a class that extends the [Converter.Factory
 class](https://github.com/square/retrofit/blob/master/retrofit/src/main/java/retrofit2/Converter.java) and pass in an instance when building your adapter.
