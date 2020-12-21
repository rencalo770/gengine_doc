# 规则引擎池
"规则引擎池",指的是一个具有多个gengine实例的池子,你可以将"规则引擎池"类比于"数据库连接池",都是为了解决性能和并发安全的问题.

### 背景

- 加载到一个gengine实例中的一系列规则,在执行的时候,是有状态的.
- 当一个gengine实例作为服务核心中调用的的模块的时候,当规则少的时候,一个请求数据执行所有规则的时间非常短,基于当前请求执行的规则的状态维持的时间也非常短,因此不会引发任何问题;
- 但当一个gengine实例中加载了几十个甚至上百个规则的时候,一个请求执行完所有的规则的时间就会变长,尤其是处于高QPS的情况下,当前请求还未执行完并返回时,下一个请求就已经到来,这个时候下一个请求就极有可能破坏当前请求执行规则的状态,并最终导致不可预知的结果
- 为了解决这个问题,gengine框架仿照"数据库连接池",实现了一个"gengine池".
- pool的所有API都是线程安全的

### 代码位置
https://github.com/rencalo770/gengine/blob/master/engine/gengine_pool.go

### 引擎池初始化
```go
//初始化一个规则引擎池
func NewGenginePool(poolMinLen ,poolMaxLen int64, em int, rulesStr string, apiOuter map[string]interface{})
```
#### 参数说明
- poolMinLen 池中初始实例化的gengine实例个数 
- poolMaxLen 池子中最多可实例化的gengine实例个数, 且poolMaxLen > poolMinLen; 当poolMinLen个实例不够用的时候,最多还可实例化(poolMaxLen-poolMinLen)个gengine实例
- em 规则执行模式,em只能等于1、2、3; em=1时,表示顺序执行规则,em=2的时候,表示并发执行规则,em=3的时候,表示混合模式执行规则;当使用```ExecuteSelectedRules```和```ExecuteSelectedRulesConcurrent```等指明了具体的执行模式的方法执行规则时,此参数自动失效
- rulesStr要初始化的所有的规则字符串
- apiOuter需要注入到gengine中使用的所有api,最好仅注入一些公共的无状态函数或参数;对于那些具体与某次请求(执行)相关的参数,则在执行规则方法时使用```data map[string]interface{} ``` 注入;这样会有利于状态管理。

#### pool中执行规则的方法
- pool中提供了一些基于执行模式的标示字段em 来控制一些通用方法的具体模式是什么,虽然对于某些需要频繁的模式切换场景带来了便利，但也为仅需要某种模式的场景带来了疑惑.因此，gengine pool实现以下策略
- gengine 单实例中的方法，对应的pool模式中也会有一个功能完全一样的同名方法实现, 这样，即确保了用户易于测试(单实例中进行)，又确保了用户易于使用(pool中同名方法),具有对应名称的方法时,初始化pool时的em字段失效
- 具体对应如下:

| ```gengine.go```中的方法(上) <br/> ```gengine_pool.go```中对应的方法(下)| 方法含义 | 
| -------- | -------- | 
|```func (g *Gengine) Execute(rb *builder.RuleBuilder, b bool) error```<br/> ****``` func (gp *GenginePool) Execute(data map[string]interface{}, b bool) (error, map[string]interface{}) ```**** | 顺序执行模式,<br/>b=true表示即使某个规则出错,也继续执行剩下的规则;<br/>如果b=false表示如果某个规则出错,则不继续执行后面的规则 |
|```func (g *Gengine) ExecuteWithStopTagDirect(rb *builder.RuleBuilder, b bool, sTag *Stag) error ``` <br/> ```func (gp *GenginePool)ExecuteWithStopTagDirect(data map[string]interface{}, b bool, sTag *Stag) (error, map[string]interface{}) ```|顺序执行所有规则,<br/>b=true表示即使某个规则出错,也继续执行剩下的规则;<br/>如果b=false表示如果某个规则出错,则不继续执行后面的规则;<br/>如果sTag=true,则不继续执行后面的规则,如果为sTag=false则继续执行后面的规则|
|```func (g *Gengine) ExecuteConcurrent(rb *builder.RuleBuilder) error  ``` <br/> ``` func (gp *GenginePool)ExecuteConcurrent(data map[string]interface{})  (error, map[string]interface{}) ```|并发模式,并发执行所有规则|
|```func (g *Gengine) ExecuteMixModel(rb *builder.RuleBuilder) error ``` <br/> ```func  (gp *GenginePool)ExecuteMixModel(data map[string]interface{})  (error, map[string]interface{})  ```|混合模式,先执行一个优先级最高的规则,然后并发执行剩下的所有规则|
|```func (g *Gengine) ExecuteMixModelWithStopTagDirect(rb *builder.RuleBuilder, sTag *Stag) error  ``` <br/> ```func  (gp *GenginePool) ExecuteMixModelWithStopTagDirect(data map[string]interface{}, sTag *Stag)  (error, map[string]interface{}) ```|混合模式, <br/>先执行一个优先级最高的规则,然后并发执行剩下的所有规则;<br/>如果sTag在第一个被执行的规则中设置为true,则不会执行剩下的规则|
|```func (g *Gengine) ExecuteSelectedRules(rb *builder.RuleBuilder, names []string) error  ``` <br/> ``` func (gp *GenginePool) ExecuteSelectedRules(data map[string]interface{}, names []string)  (error, map[string]interface{}) ```|基于规则名选择一批规则,顺序模式执行,<br/>即使执行过程中有规则执行出错，也会继续执行剩下的所有规则|
|```func (g *Gengine) ExecuteSelectedRulesWithControl(rb *builder.RuleBuilder, b bool, names []string) error ``` <br/> ```func (gp *GenginePool) ExecuteSelectedRulesWithControl(data map[string]interface{}, b bool, names []string)  (error, map[string]interface{}) ```|基于规则名选择一批规则,顺序模式执行,<br/>即使执行过程中有规则执行出错，也会继续执行剩下的所有规则;<br/>b=true表示即使某个规则出错,也继续执行剩下的规则;<br/>如果b=false表示如果某个规则出错,则不继续执行后面的规则|
|```func (g *Gengine) ExecuteSelectedRulesWithControlAndStopTag(rb *builder.RuleBuilder, b bool, sTag *Stag, names []string) error ``` <br/> ```func  (gp *GenginePool)ExecuteSelectedRulesWithControlAndStopTag(data map[string]interface{}, b bool, sTag *Stag, names []string)  (error, map[string]interface{}) ```|基于规则名选择一批规则,顺序模式执行,<br/>即使执行过程中有规则执行出错，也会继续执行剩下的所有规则;<br/>b=true表示即使某个规则出错,也继续执行剩下的规则;<br/>如果b=false表示如果某个规则出错,则不继续执行后面的规则;<br/>如果sTag在第一个被执行的规则中设置为true,则不会执行剩下的规则|
|```func (g *Gengine) ExecuteSelectedRulesConcurrent(rb *builder.RuleBuilder, names []string) error  ``` <br/> ```func (gp *GenginePool) ExecuteSelectedRulesConcurrent(data map[string]interface{}, names []string) (error, map[string]interface{}) ```|基于规则名选择一批规则, 并发执行选中的规则|
|```func (g *Gengine) ExecuteSelectedRulesMixModel(rb *builder.RuleBuilder, names []string) error ``` <br/> ```func (gp *GenginePool) ExecuteSelectedRulesMixModel(data map[string]interface{}, names []string)  (error, map[string]interface{}) ```|基于规则名选择一批规则, 混合模式执行选中的规则|
|```func (g *Gengine) ExecuteInverseMixModel(rb *builder.RuleBuilder) error ```  <br/> ``` func (gp *GenginePool) ExecuteInverseMixModel(data map[string]interface{})  (error, map[string]interface{})  ```  |逆混合执行模式，执行所有规则|
|```func (g *Gengine) ExecuteSelectedRulesInverseMixModel(rb *builder.RuleBuilder, names []string) error ``` <br/> ``` func (gp *GenginePool)ExecuteSelectedRulesInverseMixModel(data map[string]interface{}, names []string) (error, map[string]interface{})  ```|基于规则名选择一批规则,逆混合执行模式，执行选中的规则|
|```func (g *Gengine) ExecuteNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool) error  ``` <br/> ```func (gp *GenginePool) ExecuteNSortMConcurrent(nSort, mConcurrent int, b bool, data map[string]interface{}) (error, map[string]interface{})  ```|N-M模式,前N个规则顺序执行，接下来的M个规则并发执行, 且N+M大于等于规则集的长度|
|```func (g *Gengine) ExecuteNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool) error  ``` <br/> ```func (gp *GenginePool) ExecuteNConcurrentMSort(nSort, mConcurrent int, b bool, data map[string]interface{}) (error, map[string]interface{}) ```|N-M模式,前N个规则并发执行，接下来的M个规则顺序执行, 且N+M大于等于规则集的长度|
|```func (g *Gengine) ExecuteNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool) error  ```<br/> ```func (gp *GenginePool) ExecuteNConcurrentMConcurrent(nSort, mConcurrent int, b bool, data map[string]interface{}) (error, map[string]interface{})  ```|N-M模式,前N个规则并发执行，接下来的M个规则也并发执行, 且N+M大于等于规则集的长度|
|```func (g *Gengine) ExecuteSelectedNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) error  ```<br/> ```func (gp *GenginePool) ExecuteSelectedNSortMConcurrent(nSort, mConcurrent int, b bool, names []string, data map[string]interface{}) (error, map[string]interface{}) ```|基于规则名选择一批规则,N-M模式,前N个规则顺序执行，接下来的M个规则并发执行, 且N+M大于等于选定的规则集的长度 |
|```func (g *Gengine) ExecuteSelectedNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool, names []string) error ``` <br/> ```func (gp *GenginePool) ExecuteSelectedNConcurrentMSort(nSort, mConcurrent int, b bool, names []string, data map[string]interface{}) (error, map[string]interface{})  ```  |基于规则名选择一批规则, N-M模式,前N个规则并发执行，接下来的M个规则顺序执行, 且N+M大于等于选定的规则集的长度|
|```func (g *Gengine) ExecuteSelectedNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) error  ```<br/>```func (gp *GenginePool) ExecuteSelectedNConcurrentMConcurrent(nSort, mConcurrent int, b bool, names []string, data map[string]interface{}) (error, map[string]interface{})  ```|基于规则名选择一批规则,N-M模式,前N个规则并发执行，接下来的M个规则也并发执行, 且N+M大于等于选定的规则集的长度|

- 其中pool执行方法的第二份返回值,是规则集中的每个规则执行之后的return语句的返回值

### 如何设置poolMaxLen大小
- 即如何设置引擎池中实例的最大个数
- poolMinLen = poolMaxLen / 2
- poolMaxLen =  cpu_core * 5 到 cpu_core * 10   

### 动态化实时更新规则
```go 
//更新规则引擎池中gengine实例中的规则
func (gp *GenginePool)UpdatePooledRules(ruleStr string) error
```
此方法允许用户在"规则引擎池"对外提供服务的时候,实时的更新规则,且线程安全

### 清空规则
```go
//清空所有规则
func (gp *GenginePool)ClearPoolRules()
```
此方法允许用户在"规则引擎池"对外提供服务的时候,实时的清空规则,且线程安全

### 设置执行模式
```go
//设置执行模式
func (gp *GenginePool)SetExecModel(execModel int) error 
```

- 此方法允许用户在"规则引擎池"对外提供服务的时候,实时的改变规则执行模式,且线程安全
- 对execModel有效的执行函数仅有如下三个(以后还可能新增,但方法名一定以WithSpecifiedEM结尾 ), execModel 可以为1、2、3、4
1. ```func (gp *GenginePool) ExecuteRulesWithSpecifiedEM(reqName string, req interface{}, respName string, resp interface{}) (error, map[string]interface{}) ```,此方法仅允许注入req和resp参数
2. ```func (gp *GenginePool) ExecuteRulesWithMultiInputWithSpecifiedEM(data map[string]interface{}) (error, map[string]interface{})  ```, 此方法允许在运行之前注入任意多个参数
3. ```func (gp *GenginePool) ExecuteSelectedWithSpecifiedEM(data map[string]interface{}, names []string) (error, map[string]interface{}) ```,此方法在选择模式下,允许在运行之前注入任意多个参数


### 其他可能重要的方法

#### 查看规则是否存在的方法
``` func (gp *GenginePool) IsExist(ruleName string) bool  ``` 检查某个规则是否存在于pool中

#### 获取规则的优先级
``` func (gp *GenginePool) GetRuleSalience(ruleName string) (int64, error) ``` 如果error!=nil 表示不存在此规则;否则可以获取到指定规则的优先级

#### 获取规则描述信息
``` func (gp *GenginePool) GetRuleDesc(ruleName string) (string, error)  ``` 如果error!=nil 表示不存在此规则;否则可以获取到指定规则的描述

#### 获取pool中不同规则的个数
``` func (gp *GenginePool) GetRulesNumber() int  ```












