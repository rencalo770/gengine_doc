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
- em 规则执行模式,em只能等于1、2、3; em=1时,表示顺序执行规则,em=2的时候,表示并发执行规则,em=3的时候,表示混合模式执行规则;当使用```ExecuteSelectedRulesWithMultiInput```和```ExecuteSelectedRulesConcurrentWithMultiInput```方法执行规则时,此参数自动失效
- rulesStr要初始化的所有的规则字符串
- apiOuter需要注入到gengine中使用的所有api

### 如何设置poolMaxLen大小
- 即如何设置引擎池中实例的最大个数
- cpu_core * 1000 / average_response_time_per_request   






### 动态化更新规则
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
此方法允许用户在"规则引擎池"对外提供服务的时候,实时的改变规则执行模式,且线程安全
- execModel 执行模式编号,只能为1、2或3中的一个值
- 当使用```ExecuteSelectedRulesWithMultiInput```和```ExecuteSelectedRulesConcurrentWithMultiInput```等指明了具体执行模式的方法时,此参数自动失效

### 执行规则

#### ExecuteRules方法
 ```go
func (gp *GenginePool)ExecuteRules(reqName string, req interface{}, respName string, resp interface{}) error
```
- reqName 对具体值req在gengine中使用的时候的命名
- req 具体的用户请求数据
- respName 对具体值resp在gengine中使用的时候的命名
- resp 具体的用户返回数据
- 此方法的局限是,仅能传入两个临时参数
- 此方法是4种执行模式四合一的方法,具体会选择哪种执行模式,由用户使用参数execModel来设定 

#### ExecuteRulesWithMultiInput方法
```go
func  (gp *GenginePool)ExecuteRulesWithMultiInput(data map[string]interface{}) error
```
- 此方法是为了解决```ExecuteRules```方法的局限的一个新方法,其他的内部实现和其完全一样.
- 用户可以传入任意多个临时参数到gengine中,但传入的参数要先放到map中
- 此方法是4种执行模式四合一的方法,具体会选择哪种执行模式,由用户使用参数execModel来设定 - 

#### ExecuteRulesWithStopTag方法
```go
func (gp *GenginePool)ExecuteRulesWithStopTag(reqName string, req interface{}, respName string, resp interface{}, stag *Stag) error
```
- 此方法和```ExecuteRules```唯一不同的地方在于,在顺序执行模式和混合模式下,用户可以使用stag来控制是否继续执行后续规则
- 此方法是4种执行模式四合一的方法,具体会选择哪种执行模式,由用户使用参数execModel来设定 

#### ExecuteRulesWithMultiInputAndStopTag方法
```go
func (gp *GenginePool)ExecuteRulesWithMultiInputAndStopTag(data map[string]interface{}, stag *Stag) error 
```
- 此方法是4种执行模式四合一的方法,具体会选择哪种执行模式,由用户使用参数execModel来设定
- 此方法继承了```ExecuteRulesWithMultiInput``` 可以传入任意多个临时参数的优点
- 同时继承了```ExecuteRulesWithStopTag```可以在顺序执行模式和混合模式下,用户可以使用stag来控制是否继续执行后续规则的优点
- v1.1.8中引入

#### ExecuteSelectedRulesWithMultiInput方法
```go
func  (gp *GenginePool)ExecuteSelectedRulesWithMultiInput(data map[string]interface{}, names []string) error
```
- 此方法允许用户输入任意多个临时参数
- 此方法允许用户在池中选择指定的规则去执行
- 此方法顺序执行用户选中的规则

#### ExecuteSelectedRulesConcurrentWithMultiInput方法
```go
func  (gp *GenginePool)ExecuteSelectedRulesConcurrentWithMultiInput(data map[string]interface{}, names []string) error
```
- 此方法允许用户输入任意多个临时参数
- 此方法允许用户在池中选择指定的规则去执行
- 此方法并发执行用户选中的规则

#### ExecuteSelectedRulesMixModelWithMultiInput方法
```go
func (gp *GenginePool) ExecuteSelectedRulesMixModelWithMultiInput(data map[string]interface{}, names []string) error {
``` 
- 此方法允许用户输入任意多个临时参数
- 此方法允许用户在池中选择指定的规则去执行
- 此方法混合模式执行用户选中的规则


#### ExecuteInverseMixModelWithMultiInput方法
```go
func (gp *GenginePool) ExecuteInverseMixModelWithMultiInput(data map[string]interface{}) error 
```
- 此方法允许用户输入任意多个临时参数
- 此方法逆混合模式执行用户规则
- 1.3.7加入


#### ExecuteSelectedRulesInverseMixModelWithMultiInput方法
```go
func (gp *GenginePool) ExecuteSelectedRulesInverseMixModelWithMultiInput(data map[string]interface{}, names []string) error
```
- 此方法允许用户输入任意多个临时参数
- 此方法逆混合模式执行用户选中的规则
- 1.3.7加入


#### ExecuteSelected方法
- 此方法是"选择式模式"下的4种执行模式四合一的方法,具体会选择哪种执行模式,由用户使用参数execModel来设定


### 其他可能重要的方法

#### 查看规则是否存在的方法
``` func (gp *GenginePool) IsExist(ruleName string) bool  ``` 检查某个规则是否存在于pool中

#### 获取规则的优先级
``` func (gp *GenginePool) GetRuleSalience(ruleName string) (int64, error) ``` 如果error!=nil 表示不存在此规则;否则可以获取到指定规则的优先级

#### 获取规则描述信息
``` func (gp *GenginePool) GetRuleDesc(ruleName string) (string, error)  ``` 如果error!=nil 表示不存在此规则;否则可以获取到指定规则的描述

#### 获取pool中不同规则的个数
``` func (gp *GenginePool) GetRulesNumber() int  ```












