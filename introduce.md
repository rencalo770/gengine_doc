### gengine简介
- gengine是一款基于golang和AST(抽象语法树)开发的规则引擎,gengine支持的语法是一种自定义的DSL
- gengine于2020年7月由哔哩哔哩(bilibili.com)授权开源
- gengine现已应用于B站风控系统、流量投放系统、AB测试、推荐平台系统等多个业务场景
- 你也可以将gengine应用于golang应用的任何需要规则或指标支持的业务场景

gengine相比于java领域的著名规则引擎drools的优势如下:

| 对比 | drools |  gengine | 
| :--------: | :--------: | :--------------------: |
| 执行模式 | 仅支持顺序模式 | 支持顺序模式、并发模式、混合模式,以及其他细分执行模式 | 
| 规则编写难易程度 | 高,与java强相关 | 低,自定义简单语法,与golang弱相关 | 
| 规则执行性能 | 低、无论是规则之间还是规则内部,都是顺序执行 | 高,无论是规则间、还是规则内,都支持并发执行.用户基于需要来选择合适的执行模式 | 

### 为什么不使用gopher-lua或者js on golang
- 因为我们开发业务的主语言是golang,如果使用gopher-lua或者javascript on golang,那么业务逻辑会从golang中"逃逸"到lua上或者javascript上,
使用者需要额外去学习lua或者javascript, 因此增加了业务逻辑的开发难度与测试难度;使用gengine,业务逻辑始终用golang开发,逻辑实现始终控制在golang代码内,且保持golang的语言性能,
当不再需要gengine支持的时候,基本上无需做任何改动,就可以将gengine上配置的代码(规则)转化为golang原生代码.


### 设计思想
- gengine的设计思想的帖子,您可以看这里:https://xie.infoq.cn/article/40bfff1fbca1867991a1453ac


### 使用
- go.mod文件(请使用最新版本! 查看最新tag版本: https://github.com/rencalo770/gengine/tags )

如果你不仅依赖gengine,还依赖其他框架,如grpc,那么go.mod 文件这么写:

```
module your_module_name

go 1.14

require (
    google.golang.org/grpc v1.22.0
	gengine v1.3.1
)

replace gengine => github.com/rencalo770/gengine v1.3.1
```

如果你只依赖gengine,那么go.mod文件这么写:

```
module your_module_name

require gengine v1.3.1

replace gengine => github.com/rencalo770/gengine v1.3.1
```



### github地址
- https://github.com/rencalo770/gengine


### 版本
- gengine新版本完全兼容老版本,请使用最新版本
- gengine处于持续迭代中,我们也需要你的宝贵意见和想法
- https://github.com/rencalo770/gengine/tags

### 联系
- 在github上提issue
- 或者发送邮件至:
 - renyunyi@bilibli.com
 - M201476117@alumni.hust.edu.cn