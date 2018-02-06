[回到目录](http://www.cnblogs.com/lori/p/7154409.html)
### LindDotNetCore中间件
大叔认识中间件就是主要对**http请求进行拦截**，然后添加具体个性化功能的逻辑，这种把请求切开，添加新逻辑的方式一般称为面向方面的逻辑AOP！
1. 授权中间件
2. 请求链跟踪中间件
3. 响应时间中间件
#### 授权中间件
请求有效性的校验，主要由以下几个步骤：`判断是否超时`,`判断是否为有效app`,`判断加密规则是否正确`
* 授权参数
```c#
 /// <summary>
 /// 授权配置
 /// </summary>
 public class AuthorizationConfig
 {
     /// <summary>
     /// 统一密钥
     /// </summary>
     public string EncryptKey { get; set; }
     /// <summary>
     /// 过期时间秒数
     /// </summary>
     public int ExpiredSecond { get; set; }
     /// <summary>
     /// 被授权的app
     /// </summary>
     public string[] AppList { get; set; }
 }
```
* 客户端请求参数
```c#
 /// <summary>
 /// 从http请求发过来的授权实体
 /// </summary>
 public class AuthorizationRequestInfo
 {
     public string ApplicationId { get; set; }
     public string Timestamp { get; set; }
     public string Sinature { get; set; }
 }
```
* 请求拦截器，处理请求有效性，对app，过期时间，加密方式进行校验
```c#
 string computeSinature = MD5($"{requestInfo.ApplicationId}-{requestInfo.Timestamp}-{_options.EncryptKey}");
 double tmpTimestamp;
 if (computeSinature.Equals(requestInfo.Sinature) &&
     double.TryParse(requestInfo.Timestamp, out tmpTimestamp))
 {
     if (ValidateExpired(tmpTimestamp, _options.ExpiredSecond))
     {
         await ReturnTimeOut(context);
     }
     else
     {
         await ValidateApp(context, requestInfo.ApplicationId);
     }
 }
 else
 {
     await ReturnNotAuthorized(context);
 }
```
* 为开发人员提供友好的扩展方法，用来注册中间件
```c#
 /// <summary>
 /// 注册授权服务-step1
 /// </summary>
 /// <param name="services">The <see cref="IServiceCollection"/> for adding services.</param>
 /// <param name="configureOptions">A delegate to configure the <see cref="ResponseCompressionOptions"/>.</param>
 /// <returns></returns>
 public static IServiceCollection AddLindAuthrization(this IServiceCollection services, Action<AuthorizationConfig> configureOptions = null)
 {
     if (services == null)
     {
         throw new ArgumentNullException(nameof(services));
     }
     var options = new AuthorizationConfig();
     configureOptions?.Invoke(options);
     ObjectMapper.MapperTo(options, ConfigFileHelper.Get<AuthorizationConfig>());
     services.AddSingleton(options);
     return services;
 }
 
 /// <summary>
 /// 使用授权中间件-step2
 /// </summary>
 /// <param name="builder"></param>
 /// <param name="options"></param>
 /// <returns></returns>
 public static IApplicationBuilder UseLindAuthrization(this IApplicationBuilder builder)
 {
     if (builder == null)
     {
         throw new ArgumentNullException(nameof(builder));
     }
     var options = builder.ApplicationServices.GetService<AuthorizationConfig>();
     return builder.UseMiddleware<AuthorizationMiddleware>(options);
 }
```
* 使用授权中间件Startup中注册
```c#
 // 注册服务
 services.AddLindAuthrization(options =>
 {
     options.EncryptKey = "abc123";
     options.ExpiredSecond = 50;
     options.AppList = new string[] { "1", "2", "3" };
 });
 // 注册中间件　
 public void Configure(IApplicationBuilder app, IHostingEnvironment env)
 {
     if (env.IsDevelopment())
     {
         app.UseDeveloperExceptionPage();
     }
     app.UseLindAuthrization();
     app.UseMvc();
 }
```
#### 请求链跟踪中间件
记录请求经过的整个过程，对于多api相互调用的场景比较有用
#### 响应时间中间件
记录大于指定时间的请求信息，方便做性能整体的提升
待续……
[回到目录](http://www.cnblogs.com/lori/p/7154409.html)