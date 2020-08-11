### gengine简介
- gengine是一款基于golang和AST(抽象语法树)开发的规则引擎
- 于2020年7月由哔哩哔哩(bilibili.com)授权开源
- 现已应用于B站风控系统、流量投放系统、AB测试、推荐平台系统等多个业务场景

gengine相比于java领域的著名规则引擎drools的优势如下：

| 对比 | drools |  gengine | 
| :--------: | :--------: | :--------------------: |
| 执行模式 | 仅支持顺序模式 | 支持顺序模式、并发模式、混合模式，以及其他细分执行模式 | 
| 规则编写难易程度 | 高，与java强相关 | 弱，与golang弱相关 | 
| 规则执行性能 | 低、无论是规则之间还是规则内部，都是顺序执行 | 高，无论是规则间、还是规则内，都支持并发执行.用户基于需要来选择合适的执行模式 | 

### 设计思想
- gengine的设计思想的帖子，您可以看这里:https://xie.infoq.cn/article/40bfff1fbca1867991a1453ac

### github地址
- https://github.com/rencalo770/gengine
