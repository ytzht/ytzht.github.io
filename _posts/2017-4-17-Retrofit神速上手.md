---
layout:     post
title:      Retrofit神速上手
subtitle:   这个纯干货
date:       2017-4-17
author:     ZHT
header-img: img/post-bg-miui6.jpg
catalog: true
tags:
    - Android
    - Retrofit
    - 网络框架
---

# 前言

> 2017-4-17 今儿个没活，想起来这里，一寻思好像能写的东西还挺多，于是越写越多越写越多...Retrofit是最近接的一个私活用到的技术，正好开始新的项目，于是自学了新东西整理了下...

Retrofit可以认为是OkHttp的“升级版”。之所以这么说，是因为其内部默认正是基于OkHttp来进行封装的。这点从Retrofit这个命名就可以看出端倪。 
回顾一下OkHttp，我们会发现虽然是封装过后的库，但OkHttp的封装是比较“碎片化”的。所以如果不自己再进行封装，使用时代码就比较容易耦合。 
而Retrofit作为其升级版，有一个最吸引人的特色就是：将所有的请求封装为interface，并通过“注解”来声明api的相关信息。让你爽到停不下来。



Retrofit的初衷是根据RESTful风格的API来进行封装的。 
关于RESTful的学习，可以参考一下[RESTful API 设计指南](http://www.androidchina.net/3749.html)此文。

RESTful的核心思想做一个最简练的总结，那就是：**所谓”资源”，就是网络上的一个实体，或者说是网络上的一个具体信息。那么我们访问API的本质就是与网络上的某个资源进行互动而已。**

### 不多bb，不讲理论，直接上手干

#### 首先导包：

```
compile 'com.squareup.okhttp3:okhttp:3.6.0'
compile 'com.squareup.retrofit2:retrofit:2.2.0'
compile 'com.squareup.retrofit2:converter-gson:2.0.2'
compile 'com.google.code.gson:gson:2.7'//这个不要也行
```

####  新建个接口，不管放啥地方都行，存放所有接口的

我建的CBiz.class，先不管里面东西，回头再说，别问为什么，略过代码往下看就行

```java
import java.util.Map;

import okhttp3.RequestBody;
import retrofit2.Call;
import retrofit2.http.GET;
import retrofit2.http.Multipart;
import retrofit2.http.POST;
import retrofit2.http.PartMap;
import retrofit2.http.Path;
import retrofit2.http.Query;


/**
 * Created by zht on 2017/04/13 14:56
 */

public interface CBiz {

    //1.1首页
    @POST("pub/c/homePage")
    Call<HomePage> getHomePage();

    //1.4快速咨询类别
    @POST("c/freeAskCategory")
    Call<FreeAskCategory> getTags();

    //1.2免费咨询提交
    @Multipart
    @POST("c/freeask")
    Call<AskResult> freeUpload(@PartMap Map<String, RequestBody> params);

    //1.3打赏咨询提交
    @Multipart
    @POST("c/makeMoneyAskOrderAndGetPayInfo")
    Call<AskResult> payUpload(@PartMap Map<String, RequestBody> params);

    //1.5免费咨询价格和用户当前积分
    @POST("c/my/pointsInfo")
    Call<PointsInfo> getPointsInfo(@Query("userId") String userId);

    //1.6获取id对应的图片
    @GET("pic/{picId}")
    Call<BaseResult> getPicById(@Path("picId") String id);

    //1.7获取单个会话列表
    @GET("c/conversation/getList")
    Call<ConversationList> getConversationList(@Query("userId") String userId,
                                               @Query("chatId") String chatId,
                                               @Query("freeaskId") String freeaskId,
                                               @Query("direction") String direction,
                                               @Query("size") String size);

    //1.8获取免费提问详情
    @GET("c/freeaskDetail")
    Call<FreeaskDetail> getFreeaskDetail(@Query("userId") String userId,
                                         @Query("freeaskId") String freeaskId);//免费提问Id

    
}
```

#### 配置全局Application

```java
public static MyApplication instance;

public static MyApplication getInstance() {
    return instance;
}

private CBiz cBiz;

public CBiz getCBiz() {
    return cBiz;
}

@Override
public void onCreate() {
    super.onCreate();

    instance = this;

    initRetrofit();

}

private void initRetrofit() {
    cBiz = new Retrofit.Builder()
            .baseUrl(Apis.API_BASE)
            .addConverterFactory(GsonConverterFactory.create())
            .build().create(CBiz.class);
}
```

#### 在要执行的网络请求代码里

```java
Call<ResponseInfo> call = MyApplication.getInstance().getCBiz().getTags();
call.enqueue(new Callback<ResponseInfo>() {
    @Override
    public void onResponse(Call<ResponseInfo> call, Response<ResponseInfo> response) {

        if (response.isSuccessful()) {
            ResponseInfo responseInfo = response.body();
			Log.d("返回字段：", responseInfo.getResponse());
        }else {
            try {
                L.e(response.errorBody().string());
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void onFailure(Call<FreeAskCategory> call, Throwable t) {
        L.e(t.toString());
    }
});
```

这里的ResponseInfo替换为返回的数据对应的实体类即可解析

```java
public class ResponseInfo {
    private String response;

    public String getResponse() {
        return response;
    }
}
```

getTags()方法即为CBiz.class里定义好的接口，要访问什么接口只需在CBiz.class里面定义好，然后在代码里调用。

好了，到这里聪明人已经会用了，笨人就继续往下看吧，还看不懂就真的没救了。。。

### 从官方介绍初识Retrofit

当我们要去学习一样新的技术，还有什么是比官方的资料更好的了呢？所以，第一步我们打开网址[http://square.github.io/retrofit/](http://square.github.io/retrofit/)。然后开始阅读： 
官方的介绍非常简单粗暴，一上来就通过一个**Introduction**来展示了如何使用Retrofit来进行一个最基本的请求。于是我也懒得翻译了，下面类似笔记之类的东西

```java
@GET("api/{name}")
Call<ResponseInfo> testHttpGet(@Path("name") String apiAction);
```

name通过testHttpGet("name");传进去，注意花括号和Path注解

```java
 @GET("api/retrofitTesting")
 Call<ResponseInfo> testHttpGet(@Query("param") String param);
```

调用方法 testHttpGet("value"); 实际上访问的是 BaseUrl + “api/retrofitTesting?param=value” ， 注意 @Query

假设我要在参数中上传10个参数呢？这意味着我要在方法中声明10个@Query参数？可以用@QueryMap

```java
@GET("api/retrofitTesting")
Call<ResponseInfo> testHttpGet(@QueryMap Map<String,String> params);
```

调用的时候自然也发生了变化：

```java
 Map<String,String> params =new HashMap<>();
 params.put("param1","value1");
 params.put("param2","value2");
 Call<ResponseInfo> call = MyApplication.getInstance().getCBiz().testHttpGet(params);
```

有了之前的基础，现在我们免不了要捣鼓一下POST。相信有了@GET的使用经验，如果只是想发发简单的POST请求是没有多大的难度，我们关注下POST的BODY，即请求体的使用技巧。

通过官方文档，我们发现出现了一个新的东西叫做“@Body”，顾名思义它就是用来封装请求体的。而同时通过后面的“User”参数类型，我们不难推断出：使用@Body时，是通过实体对象的形式来进行封装的。

```java
@POST("api/users")
Call<ResponseInfo> uploadNewUser(@Body User user);
```

User为实体类，属性无非用户年龄性别啥的，对应的调用：

```java
Call<ResponseInfo> call = service.uploadNewUser(new User("zht","male",24));
```

这里唯一需要说明的就是，通过@BODY这种方式封装请求体，Retrofit是通过JSON的形式来封装数据的。我们可以在服务器读取流中的数据打印：

通过表单形式来上传参数：

```java
 @FormUrlEncoded
 @POST("api/users")
 Call<ResponseInfo> uploadNewUser(@Field("username") String username,@Field("gender") String male,@Field("age") int age);
```

然后是依旧是调用，这时服务器就可以通过request.getParameter直接读取参数了：

```java
Call<ResponseInfo> call = service.uploadNewUser("zht","male",24);
```

当然，这个时候难免又会想起那个老梗：要写这么多的@Field参数？当然不是，也有@FieldMap供我们使用，使用方法参照@QueryMap

使用了@FormUrlEncoded之后，参数中含有中文信息，会出现乱码， 通过URLEncode对数据进行指定编码，然后服务器再进行对应的解码读取：

```java
String name =  URLEncoder.encode("张三","UTF-8");
Call<ResponseInfo> call = service.uploadNewUser(name,"male",24);
```

但如果了解一点HTTP协议的使用，我们知道还有另一种解决方式：在Request-Header中设置charset信息。于是，这个时候就涉及到添加请求头了，关于@Headers的使用看上去非常简单。那么，接下来我们就来修改一下我们之前的代码：

```java
@Headers("Content-type:application/x-www-form-urlencoded;charset=UTF-8")
@FormUrlEncoded
@POST("api/users")
Call<ResponseInfo> uploadNewUser(@Field("username") String username,@Field("gender") String male,@Field("age") int age);
```

通过@Headers我们在Content-type中同时指明了编码信息，除了@Headers之外，还有另一个注解叫做@Header。它的不同在于是动态的来添加请求头信息，也就是说更加灵活一点：

```java
@FormUrlEncoded
@POST("api/users")
Call<ResponseInfo> uploadNewUser(@Header("Content-Type") String contentType,@Field("username") String username,@Field("gender") String male,@Field("age") int age);
}
// 调用
Call<ResponseInfo> call = service.uploadNewUser("application/x-www-form-urlencoded;charset=UTF-8","张三","male",24);
```

读取response header

通过上面的总结我们知道通过@Header可以在请求中添加header，那么我们如何去读取响应中的header呢？我们会发现官方文档并没有相关介绍。 
那么显然我们就只能自己看看了，一找发现对于Retrofit2来说Response类有一个方法叫做headers()，通过它就获取了本次请求所有的响应头。 
那么，我们记得在OkHttp里面，除了headers()，还有用于获取单个指定头的header()方法。我们能不能在Retrofit里使用这种方式呢？答案是可以。 
我们发现Retrofit的Response还有一个方法叫做raw()，调用该方法就可以把Retrofit的Response转换为原生的OkHttp当中的Response。而现在我们就很容器实现header的读取了吧。

```java
okhttp3.Response okResponse = response.raw();
response.raw().header("Cache-Control");
```

### 进阶技巧

##### @Multipart 与文件上传

```java
    //1.2免费咨询提交
    @Multipart
    @POST("c/freeask")
    Call<AskResult> freeUpload(@PartMap Map<String, RequestBody> params);

    //1.3打赏咨询提交
    @Multipart
    @POST("c/makeMoneyAskOrderAndGetPayInfo")
    Call<AskResult> payUpload(@PartMap Map<String, RequestBody> params);
```

调用时这样式的：

```java
Map<String, RequestBody> photoMap = new HashMap<>();
photoMap.put("pic" + (i + 1) + "\"; filename=\"pic" + (i + 1) + photos.get(i).substring(photos.get(i).lastIndexOf(".")),RequestBody.create(MediaType.parse("multipart/form-data"), file));//pic0,pic1,pic2为键名，pic0demopic.png为文件名。
photoMap.put("userId", RequestBody.create(null, "2"));
photoMap.put("type", RequestBody.create(null, "2"));
Call<AskResult> payCall = MyApplication.getInstance().getCBiz().payUpload(photoMap);
```

##### 不同类型的Converter

相信我们还记得之前我们读取服务器返回的json数据时，依赖于GsonConverterFactory就成功完成了解析。那么问题来了： 
假设有的时候我们服务器返回的就是一段简单的文本信息呢？知悉要将Call的泛型指定为String；最后记得将Converter改掉，像下面这样：

```java
.addConverterFactory(ScalarsConverterFactory.create())
```

有了这个例子，对于同类型的需求我们就很容易举一反三了。但马上又想到：假设我们的数据类型官方没有提供现成的Converter呢？很简单，自己造！

自定义Converter即创建一个继承自Converter.Factory的类，然后提供自己的Converter实例就行了。我感觉没啥用，这里不作详解

##### 使用Interceptor

interceptor顾名思义就是拦截器，其具体使用方法可以参照[https://github.com/square/okhttp/wiki/Interceptors](https://github.com/square/okhttp/wiki/Interceptors)。

```java
import java.io.IOException;

import okhttp3.Interceptor;
import okhttp3.Request;
import okhttp3.Response;

/**
 * Created by zht on 2017/04/17 15:32
 */

public class LogInterceptor implements Interceptor {

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        long t1 = System.nanoTime();
        L.d(String.format("Sending request %s on %s%n%s", request.url(), chain.connection(), request.headers()));//打印log

        Response response = chain.proceed(request);

        long t2 = System.nanoTime();
        L.d(String.format("Received response for %s in %.1fms%n%s", response.request().url(), (t2 - t1) / 1e6d, response.headers()));//打印log

        return response.newBuilder().header("Cache-Control","max-age=100").build();//统一添加头信息

    }

}
```

然后呢？设置到 OKHttpClient 里，同样在 Application 里，initRetrofit() 方法改写：

```java
        OkHttpClient client = new OkHttpClient.Builder()
                .addInterceptor(new LogInterceptor())
                .build();
        cBiz = new Retrofit.Builder()
                .baseUrl(Apis.API_BASE)
                .client(client)
                .addConverterFactory(GsonConverterFactory.create())
                .build().create(CBiz.class);
```

大功告成！













