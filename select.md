# 基于用户选择的规则执行方式
### 背景
我们在gengine使用的过程中观察到这种需求:假设现在gengine中有10个规则,规则名编号从"1"到"10",当一个请求过来的时候,用户是明确知道自己需要执行哪几个规则的.

### 当前的执行模式
如果我们在用户知道仅需要执行哪几个规则的情况下,还是直接去以顺序、或者并发、或者混合模式去执行所有的规则的话,用户很可能得不到想要的结果,因为不同的规则间很可能是互斥的,或者完全不相关的.因此,gengine支持基于用户选择具体的规则的方式进行执行.

### 具体的方法

#### ExecuteSelectedRules方法
```go
func (g *Gengine)ExecuteSelectedRules(rb * builder.RuleBuilder, names []string)
```
- names参数,传入的是多个规则名称组成的数组,如当用户传入```name :=[]string{"1", "7", "3"}```时,gengine会基于用户选择的这3个规则的优先级顺序执行这3个规则.  
- 如果用户传入的规则名不存在,gengine在执行的过程中会日志记录下来.

#### ExecuteSelectedRulesConcurrent方法
```go
func (g *Gengine)ExecuteSelectedRulesConcurrent(rb * builder.RuleBuilder, names []string) 
```
- names参数,传入的是多个规则名称组成的数组,如当用户传入```name :=[]string{"1", "7", "3"}```时,gengine不会考虑规则的优先级,并发执行这3个规则.  
- 如果用户传入的规则名不存在,gengine在执行的过程中会日志记录下来.

### 具体实现
- https://github.com/rencalo770/gengine/blob/master/engine/gengine.go

### 测试用例
- https://github.com/rencalo770/gengine/blob/master/test/exceute_selected_rules_test.go
