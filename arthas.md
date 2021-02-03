# Arthas

## [Arthas 简介](http://www.macrozheng.com/#/reference/arthas_start?id=arthas-%e7%ae%80%e4%bb%8b)
Arthas是Alibaba开源的Java诊断工具，深受开发者喜爱。它采用命令行交互模式，同时提供丰富的 Tab 自动补全功能，进一步方便进行问题的定位和诊断。
## [安装](http://www.macrozheng.com/#/reference/arthas_start?id=%e5%ae%89%e8%a3%85)
> 为了还原一个真实的线上环境，我们将通过Arthas来对Docker容器中的Java程序进行诊断。

- 使用`arthas-boot`，下载对应jar包，下载地址：[https://alibaba.github.io/arthas/arthas-boot.jar](https://alibaba.github.io/arthas/arthas-boot.jar)

- 将我们的Spring Boot应用`mall-tiny-arthas`使用Docker容器的方式启动起来，打包和运行脚本在项目的`src\main\docker`目录下；

- 将`arthas-boot.jar`拷贝到我们应用容器的`\`目录下；

```
docker container cp arthas-boot.jar mall-tiny-arthas:/Copy to clipboardErrorCopied
```

- 进入容器并启动`arthas-boot`，直接当做jar包启动即可；
```
docker exec -it mall-tiny-arthas /bin/bash
java -jar arthas-boot.jarCopy to clipboardErrorCopied
```

- 启动成功后，选择当前需要诊断的Java程序的序列号，这里是`1`，就可以开始诊断了；

![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488700155-d1f2409d-8188-4352-99d5-a10040883744.png#align=left&display=inline&height=126&margin=%5Bobject%20Object%5D&originHeight=126&originWidth=1222&size=0&status=done&style=none&width=1222)

- 期间会下载一些所需的文件，完成后控制台打印信息如下，至此Arthas就安装启动完成了。

![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488700419-1a742560-343e-4756-9ae4-7b6b236b4e0b.png#align=left&display=inline&height=424&margin=%5Bobject%20Object%5D&originHeight=424&originWidth=711&size=0&status=done&style=none&width=711)
## [常用命令](http://www.macrozheng.com/#/reference/arthas_start?id=%e5%b8%b8%e7%94%a8%e5%91%bd%e4%bb%a4)
> 我们先来介绍一些Arthas的常用命令，会结合实际应用来讲解，带大家了解下Arthas的使用。

### [dashboard](http://www.macrozheng.com/#/reference/arthas_start?id=dashboard)
使用`dashboard`命令可以显示当前系统的实时数据面板，包括线程信息、JVM内存信息及JVM运行时参数。
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488700436-3aa6e22f-a642-4953-8bde-0c85293c030a.png#align=left&display=inline&height=469&margin=%5Bobject%20Object%5D&originHeight=469&originWidth=1131&size=0&status=done&style=none&width=1131)
### [thread](http://www.macrozheng.com/#/reference/arthas_start?id=thread)
查看当前线程信息，查看线程的堆栈，可以找出当前最占CPU的线程。
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488700533-fec8ea44-ecf8-41d5-98c5-b44b05f7751d.png#align=left&display=inline&height=257&margin=%5Bobject%20Object%5D&originHeight=257&originWidth=1265&size=0&status=done&style=none&width=1265)
常用命令：
```
# 打印当前最忙的3个线程的堆栈信息
thread -n 3
# 查看ID为1都线程的堆栈信息
thread 1
# 找出当前阻塞其他线程的线程
thread -b
# 查看指定状态的线程
thread -state WAITINGCopy to clipboardErrorCopied
```
### [sysprop](http://www.macrozheng.com/#/reference/arthas_start?id=sysprop)
查看当前JVM的系统属性，比如当容器时区与宿主机不一致时，可以使用如下命令查看时区信息。
```
sysprop |grep timezoneCopy to clipboardErrorCopied
```
```
user.timezone                  Asia/ShanghaiCopy to clipboardErrorCopied
```
### [sysenv](http://www.macrozheng.com/#/reference/arthas_start?id=sysenv)
查看JVM的环境属性，比如查看下我们当前启用的是什么环境的Spring Boot配置。
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488700412-6cfe4dd3-1c52-407a-bca2-484055566e6f.png#align=left&display=inline&height=433&margin=%5Bobject%20Object%5D&originHeight=433&originWidth=745&size=0&status=done&style=none&width=745)
### [logger](http://www.macrozheng.com/#/reference/arthas_start?id=logger)
使用`logger`命令可以查看日志信息，并改变日志级别，这个命令非常有用。
比如我们在生产环境上一般是不会打印`DEBUG`级别的日志的，当我们在线上排查问题时可以临时开启`DEBUG`级别的日志，帮助我们排查问题，下面介绍下如何操作。

- 我们的应用默认使用的是`INFO`级别的日志，使用`logger`命令可以查看；

![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488700412-2d76aba2-b82b-41ea-94e0-71ffc0c25e1c.png#align=left&display=inline&height=274&margin=%5Bobject%20Object%5D&originHeight=274&originWidth=1025&size=0&status=done&style=none&width=1025)

- 使用如下命令改变日志级别为`DEBUG`，需要使用`-c`参数指定类加载器的HASH值；
```
logger -c 21b8d17c --name ROOT --level debugCopy to clipboardErrorCopied
```

- 再使用`logger`命令查看，发现`ROOT`级别日志已经更改；

![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488700437-1d746ac2-d4ba-4c35-bcc4-2524adf4f94f.png#align=left&display=inline&height=268&margin=%5Bobject%20Object%5D&originHeight=268&originWidth=1022&size=0&status=done&style=none&width=1022)

- 使用`docker logs -f mall-tiny-arthas`命令查看容器日志，发现已经打印了DEBUG级别的日志；

![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488700414-ee68753f-751d-4741-b510-06374c2decef.png#align=left&display=inline&height=341&margin=%5Bobject%20Object%5D&originHeight=341&originWidth=851&size=0&status=done&style=none&width=851)

- 查看完日志以后记得要把日志级别再调回`INFO`级别。
```
logger -c 21b8d17c --name ROOT --level infoCopy to clipboardErrorCopied
```
### [sc](http://www.macrozheng.com/#/reference/arthas_start?id=sc)
查看JVM已加载的类信息，`Search-Class`的简写，搜索出所有已经加载到 JVM 中的类信息。

- 搜索`com.macro.mall`包下所有的类；
```
sc com.macro.mall.*Copy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488700658-82d8fa14-cddd-4be5-859c-235d7debe025.png#align=left&display=inline&height=380&margin=%5Bobject%20Object%5D&originHeight=380&originWidth=971&size=0&status=done&style=none&width=971)

- 打印类的详细信息，加入`-d`参数并指定全限定类名；
```
sc -d com.macro.mall.tiny.common.api.CommonResultCopy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488701131-37026e1b-0b38-41b8-8549-af1b3cb3efc1.png#align=left&display=inline&height=456&margin=%5Bobject%20Object%5D&originHeight=456&originWidth=760&size=0&status=done&style=none&width=760)

- 打印出类的Field信息，使用`-f`参数。
```
sc -d -f com.macro.mall.tiny.common.api.CommonResultCopy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488701115-2e56338f-22c5-44ba-bbbe-03fc13784867.png#align=left&display=inline&height=217&margin=%5Bobject%20Object%5D&originHeight=217&originWidth=583&size=0&status=done&style=none&width=583)
### [sm](http://www.macrozheng.com/#/reference/arthas_start?id=sm)
查看已加载类的方法信息，`Search-Method`的简写，搜索出所有已经加载的类的方法信息。

- 查看类中的所有方法；
```
sm com.macro.mall.tiny.common.api.CommonResultCopy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488700463-4936607e-d694-4935-aaa4-3888e6bc7b8e.png#align=left&display=inline&height=342&margin=%5Bobject%20Object%5D&originHeight=342&originWidth=1261&size=0&status=done&style=none&width=1261)

- 查看指定方法信息，使用`-d`参数并指定方法名称；
```
sm -d com.macro.mall.tiny.common.api.CommonResult getCodeCopy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488701300-e34563de-ae41-4208-9586-16282d6388b6.png#align=left&display=inline&height=174&margin=%5Bobject%20Object%5D&originHeight=174&originWidth=626&size=0&status=done&style=none&width=626)
### [jad](http://www.macrozheng.com/#/reference/arthas_start?id=jad)
反编译已加载类的源码，觉得线上代码和预期不一致，可以反编译看看。

- 查看启动类的相关信息，默认会带有`ClassLoader`信息；
```
jad com.macro.mall.tiny.MallTinyApplicationCopy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488701396-2c0db779-d6d7-44d8-a5e8-00f267682332.png#align=left&display=inline&height=505&margin=%5Bobject%20Object%5D&originHeight=505&originWidth=666&size=0&status=done&style=none&width=666)

- 使用`--source-only`参数可以只打印类信息。
```
jad --source-only com.macro.mall.tiny.MallTinyApplicationCopy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488701318-240764d5-df2f-435e-9705-361d22f13e99.png#align=left&display=inline&height=371&margin=%5Bobject%20Object%5D&originHeight=371&originWidth=683&size=0&status=done&style=none&width=683)
### [mc](http://www.macrozheng.com/#/reference/arthas_start?id=mc)
内存编译器，`Memory Compiler`的缩写，编译`.java`文件生成`.class`。
### [redefine](http://www.macrozheng.com/#/reference/arthas_start?id=redefine)
加载外部的`.class`文件，覆盖掉 JVM中已经加载的类。
### [monitor](http://www.macrozheng.com/#/reference/arthas_start?id=monitor)
实时监控方法执行信息，可以查看方法执行成功此时、失败次数、平均耗时等信息。
```
monitor -c 5 com.macro.mall.tiny.controller.PmsBrandController listBrandCopy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488702422-b0181c22-d9c9-4a35-a2b8-4134d6dc671d.png#align=left&display=inline&height=142&margin=%5Bobject%20Object%5D&originHeight=142&originWidth=1420&size=0&status=done&style=none&width=1420)
### [watch](http://www.macrozheng.com/#/reference/arthas_start?id=watch)
方法执行数据观测，可以观察方法执行过程中的参数和返回值。
使用如下命令观察方法执行参数和返回值，`-x`表示结果属性遍历深度。
```
watch com.macro.mall.tiny.service.impl.PmsBrandServiceImpl listBrand "{params,returnObj}" -x 2Copy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488701363-6a541800-659b-4d7e-bb97-e8e306285319.png#align=left&display=inline&height=348&margin=%5Bobject%20Object%5D&originHeight=348&originWidth=962&size=0&status=done&style=none&width=962)
## [热更新](http://www.macrozheng.com/#/reference/arthas_start?id=%e7%83%ad%e6%9b%b4%e6%96%b0)
> 尽管在线上环境热更代码并不是一个很好的行为，但有的时候我们真的很需要热更代码。下面介绍下如何使用`jad/mc/redefine`来热更新代码。

- 首先我们有一个商品详情的接口，当我们传入`id<=0`时，会抛出`IllegalArgumentException`；
```
/**
 * 品牌管理Controller
 * Created by macro on 2019/4/19.
 */
@Api(tags = "PmsBrandController", description = "商品品牌管理")
@Controller
@RequestMapping("/brand")
public class PmsBrandController {
    @Autowired
    private PmsBrandService brandService;
    private static final Logger LOGGER = LoggerFactory.getLogger(PmsBrandController.class);
    @ApiOperation("获取指定id的品牌详情")
    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    @ResponseBody
    public CommonResult<PmsBrand> brand(@PathVariable("id") Long id) {
        if(id<=0){
            throw new IllegalArgumentException("id not excepted id:"+id);
        }
        return CommonResult.success(brandService.getBrand(id));
    }
}Copy to clipboardErrorCopied
```

- 调用接口会返回如下信息，调用地址：[http://192.168.5.94:8088/brand/0](http://192.168.5.94:8088/brand/0)
```
{
  "timestamp": "2020-06-12T06:20:20.951+0000",
  "status": 500,
  "error": "Internal Server Error",
  "message": "id not excepted id:0",
  "path": "/brand/0"
}Copy to clipboardErrorCopied
```

- 我们想对该问题进行修复，如果传入`id<=0`时，直接返回空数据的`CommonResult`，代码修改内容如下；
```
/**
 * 品牌管理Controller
 * Created by macro on 2019/4/19.
 */
@Api(tags = "PmsBrandController", description = "商品品牌管理")
@Controller
@RequestMapping("/brand")
public class PmsBrandController {
    @Autowired
    private PmsBrandService brandService;
    private static final Logger LOGGER = LoggerFactory.getLogger(PmsBrandController.class);
    @ApiOperation("获取指定id的品牌详情")
    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    @ResponseBody
    public CommonResult<PmsBrand> brand(@PathVariable("id") Long id) {
        if(id<=0){
//            throw new IllegalArgumentException("id not excepted id:"+id);
            return CommonResult.success(null);
        }
        return CommonResult.success(brandService.getBrand(id));
    }
}Copy to clipboardErrorCopied
```

- 首先我们需要对`PmsBrandController`类代码进行修改，接着上传到服务器，然后使用如下命令将`java`文件拷贝到容器的`/tmp`目录下；
```
docker container cp /tmp/PmsBrandController.java mall-tiny-arthas:/tmp/Copy to clipboardErrorCopied
```

- 之后我们需要查看该类的类加载器的Hash值；
```
sc -d *PmsBrandController | grep classLoaderHashCopy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488701288-0f88bf75-3337-4c8a-a94d-ab6e91b1571a.png#align=left&display=inline&height=38&margin=%5Bobject%20Object%5D&originHeight=38&originWidth=563&size=0&status=done&style=none&width=563)

- 之后使用内存编译器把改`.java`文件编译成`.class`文件，注意需要使用`-c`指定类加载器；
```
mc -c 21b8d17c /tmp/PmsBrandController.java -d /tmpCopy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488701301-60f2c904-f781-4a5a-bfab-c39372cfbd68.png#align=left&display=inline&height=72&margin=%5Bobject%20Object%5D&originHeight=72&originWidth=580&size=0&status=done&style=none&width=580)

- 最后使用`redefine`命令加载`.class`文件，将原来加载的类覆盖掉；
```
redefine -c 21b8d17c /tmp/com/macro/mall/tiny/controller/PmsBrandController.classCopy to clipboardErrorCopied
```
![](https://cdn.nlark.com/yuque/0/2020/png/1742254/1604488701286-e3da87b0-e902-46db-be45-a7989fda834d.png#align=left&display=inline&height=35&margin=%5Bobject%20Object%5D&originHeight=35&originWidth=842&size=0&status=done&style=none&width=842)

- 我们再次调用接口进行测试，发现已经返回了预期的结果，调用地址：[http://192.168.3.101:8088/brand/0](http://192.168.3.101:8088/brand/0)
```
{
  "code": 200,
  "message": "操作成功",
  "data": null
}
```
