# 规则引擎池
"规则引擎池",指的是一个具有多个gengine实例的池子,你可以将"规则引擎池"类比于"数据库连接池",都是为了解决性能和并发安全的问题.

### 背景

- 加载到一个gengine实例中的一系列规则,在执行的时候,是有状态的.
- 当一个gengine实例作为服务核心中调用的的模块的时候,当规则少的时候,一个请求数据执行所有规则的时间非常短,基于当前请求执行的规则的状态维持的时间也非常短,因此不会引发任何问题;
- 但当一个gengine实例中加载了几十个甚至上百个规则的时候,一个请求执行完所有的规则的时间就会变长,尤其是处于高QPS的情况下,当前请求还未执行完并返回时,下一个请求就已经到来,这个时候下一个请求就极有可能破坏当前请求执行规则的状态,并最终导致不可预知的结果
- 为了解决这个问题,gengine框架仿照"数据库连接池",实现了一个"gengine池".

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

### 动态化更新规则
```go 
//更新规则引擎池中gengine实例中的规则
func (gp *GenginePool)UpdatePooledRules(ruleStr string) error
```
此方法允许用户在"规则引擎池"对外提供服务的时候,实时的更新规则,且线程安全

#### 参数说明
- ruleStr 新的用户需要加载的规则的字符串

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

#### 参数说明
- execModel 执行模式编号,只能为1、2或3中的一个值
- 当使用```ExecuteSelectedRulesWithMultiInput```和```ExecuteSelectedRulesConcurrentWithMultiInput```方法执行规则时,此参数自动失效
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

#### ExecuteRulesWithMultiInput方法
```go
func  (gp *GenginePool)ExecuteRulesWithMultiInput(data map[string]interface{}) error
```
 此方法是为了解决```ExecuteRules```方法的局限的一个新方法,其他的内部实现和其完全一样.用户可以传入任意多个临时参数到gengine中,但传入的参数要先放到map中

#### ExecuteRulesWithStopTag方法
```go
func (gp *GenginePool)ExecuteRulesWithStopTag(reqName string, req interface{}, respName string, resp interface{}, stag *Stag) error
```
此方法和```ExecuteRules```唯一不同的地方在于,在顺序执行模式下,用户可以使用stag来控制是否继续执行后续规则

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

### 测试代码
- https://github.com/rencalo770/gengine/blob/master/test/pool_test.go



















