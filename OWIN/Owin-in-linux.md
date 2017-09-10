# 记录一下将简单的webapi迁移到linux
## 第一步 使用VS新建一个WebApi项目，并用上OWIN，在Windows上跑通
#### 1、新建项目OwinSelfhostSample，输出方式为控制台程序
#### 2、使用nuget安装两个库：Microsoft.Owin和System.Web.Http.Cors，前者是OWIN，后者用于使用CORS跨域
#### 3、编辑OwinSelfhostSample下的Program.cs
```
class Program
{
    static void Main(string[] args)
    {
        string baseAddress = "http://+:9000/"; //绑定所有地址，外网可以用ip访问，这里的加号为通配符
        // 启动 OWIN host 
        WebApp.Start<Startup>(url: baseAddress);
        Console.WriteLine("程序已启动,按任意键退出");
        Console.ReadLine(); 

    }
}
```

#### 4、编辑OwinSelfhostSample下的Startup.cs
```
public class Startup
{
    // This code configures Web API. The Startup class is specified as a type
    // parameter in the WebApp.Start method.
    public void Configuration(IAppBuilder appBuilder)
    {
        HttpConfiguration config = new HttpConfiguration();

        //跨域配置
        config.EnableCors(new EnableCorsAttribute("*", "*", "*"));

        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "api/{controller}/{id}",
            defaults: new { id = RouteParameter.Optional }
        );
        
        appBuilder.UseWebApi(config);
    } 
}
```

#### 5、新建类库Adapter.cs，建立Jexus适配器代码
```
public class Adapter
{
    static Func<IDictionary<string, object>, Task> _owinApp;

    /// <summary>
    /// 默认构造函数
    /// </summary>
    public Adapter()
    {
        //创建默认的AppBuilder
        var builder = new AppBuilder();

        //创建用户定义的 Startup类
        //这个类中必须有“Configuration”方法
        var startup = new Startup();

        //调用Configuration方法，把自己的处理函数注册到处理流程中
        startup.Configuration(builder);

        //生成OWIN“入口”函数
        _owinApp = builder.Build();
    }

    /// <summary>
    /// *** JWS所需要的关键函数 ***
    /// <para>每个请求到来，JWS都把请求打包成字典，通过这个函数交给使用者</para>
    /// </summary>
    /// <param name="env">新请求的环境字典，具体内容参见OWIN标准</param>
    /// <returns>返回一个正在运行或已经完成的任务</returns>
    public Task OwinMain(IDictionary<string, object> env)
    {
        if (_owinApp == null) return null;

        // 将请求交给Microsoft.Owin对这个请求进行处理
        //（你的处理方法已经在本类的构造函数中加入到它的处理序列中了）
        return _owinApp(env);
    }
}
```

#### 6、新建熟悉的控制器代码，我新建了Controller文件夹，在Controller中又新建了UserController控制器
```
public class UserController : ApiController
{
    // GET api/values 
    public IEnumerable<string> Get()
    {
        return new string[] { "value1", "value2" };
    }

    // GET api/values/5 
    public string Get(int id)
    {
        return "value";
    }

    // POST api/values 
    public UserModel Post([FromBody]UserModel model)
    {
        return model;
    }

    // PUT api/values/5 
    public void Put(int id, [FromBody]string value)
    {
    }

    // DELETE api/values/5 
    public void Delete(int id)
    {
    }
}

public class UserModel     //用于测试post
{
    public string UserID { get; set; }
    public string UserName { get; set; }
}
```

#### 7、先在Windows下试下，我代码中使用的是9000端口，那么需要在windows服务器上开放9000端口，开放端口方法请google或baidu
#### 8、在windows服务器上运行，然后使用postman测试了下，没问题

## 第二步 将代码放到Linux上测试
#### 1、我使用的是CentOS6.5，在Linux服务器上安装ftp工具vsftpd
#### 2、设置vsftpd
#### service vsftpd start 启动vsftpd
#### chkconfig --level 35 vsftpd on 开机自启动
#### vi /etc/vsftpd/vsftpd.conf 编辑vsftpd的配置文件 具体配置自行百度，我这边只进行了禁止匿名用户的访问，不编辑用默认配置也可以
#### useradd ftpadmin -s /sbin/nologin 添加用户ftpadmin
#### passwd ftpadmin 设置ftpadmin的密码，设置完后就可以使用ftpadmin这个账户登录使用ftp了
#### 3、开发电脑（我的是windows10），安装ftp工具，我安装的是FileZilla，安装完后连接Linux服务器，vsftpd的默认端口是21
#### 4、在VS中，“生成”-“配置管理器”，将本项目OwinSelfhostSample的配置改成“Release”（这一步不知道是不是必要的），再生成一次项目
#### 5、将项目目录下bin/Release中的内容，通过FileZilla复制到Linux默认的vsftpd目录home/ftpadmin下，这里建议新建一个bin目录，直接将bin/Release中的内容复制到home/ftpadmin/bin中（因为部署到Linux中的网站目录下需要一个bin文件夹，到时直接复制过去就行）
#### 6、安装jexus独立版（整合了Mono，不需要另外安装Mono），安装方法参考：http://www.cnblogs.com/yunei/p/5452120.html
#### 7、进入jexus目录下的siteconf文件夹，这里面就是网站配置文件，默认有一个default（如果要用80端口，直接编辑这个文件即可），我们这边将default复制一份，文件名为OwinSelfhostSample
#### 其中的内容只需要改前面部分：
#### port=9000
#### root=/var/www/OwinSelfhostSample   #这个是网站文件目录，过会再创建
#### hosts=*
#### OwinMain=OwinSelfhostSample.dll,OwinSelfhostSample.Adapter   #这一行是新增的，也是必须的
#### 其他部分不用改，保存即可
#### 8、进入/var/www目录，创建我们的网站目录OwinSelfhostSample
#### 9、将通过vsftpd传上来的文件夹bin复制到OwinSelfhostSample下面，目录结构：/var/www/OwinSelfhostSample/bin/我们本地Release中的所有文件
#### 10、通过jexus运行我们的网站
#### sudo /usr/jexus/jws start OwinSelfhostSample
#### 11、网站启动成功后，使用postman测试下webapi，成功获取到数据
