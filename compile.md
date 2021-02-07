# 规则更新与删除
## 规则更新方式
- 也叫规则编译方式
- 在一个场景中,当规则仅有几十个或上百个的时候,改动一个规则时,重新编译所有规则并加载,这不会引起什么问题。但这种模式对于只有少量规则的场景是适合的;
- 但随着业务推进,一个场景可能规则多到成千上万个,如果还是以每次全量的方式来编译更新,则会严重损耗服务器性能(CPU资源);并且我们在修改规则的时候,常常也是一个个的修改,因此,对于那些没有改动的规则,进行重新编译更新,是一种完全不必要服务器资源浪费.
- 基于以上两点考虑,gengine支持2种规则更新方法,一种是全量更新,一种是增量更新.
- 规则更新模式借鉴Java的CopyOnWriteArrayList和CopyOnWriteArraySet, 因此是线程安全的

#### 全量更新
- 言下之意就是,当一批规则加载到gengine中,无论是新增一个规则,还是改动其中一个规则,我们都会移除上一次的所有规则,然后重新编译这一批传过来的所有规则,并且将新编译的这些规则到gengine实例中.
- 接口如下

```go
//单实例时
ruleBuilder := builder.NewRuleBuilder(dataContext)
e1 := ruleBuilder.BuildRuleFromString(rulesString) //这里全量更新

//pool中全量更新
//第一处
pool初始化的时候是全量更新的
//第二处
func (gp *GenginePool) UpdatePooledRules(ruleStr string) error


```

#### 增量更新
- 言下之意就是, 当有一批规则加载到gengine中时,如果新增规则或改动规则,gengine仅编译新增的或者改动的规则,对于那些没有变化的规则,则不进行任何操作.这种操作的优势非常明显,我们只为改动付出"代价",对于"不变",我们则不必付出任何代价.始终确保在服务期间,编译过程只消耗极少量的资源.
- 但这里需要用户对于传入的规则做一点小改动,那就是用户仅传入被改动(或新增)的规则的字符串,对于未改动的规则字符串,则不传入.
- 接口如下

```go
//单实例 rule_builder.go
func (builder *RuleBuilder)BuildRuleWithIncremental(ruleString string) error

//pool中的增量更新接口
func (gp *GenginePool) UpdatePooledRulesIncremental(ruleStr string) error 

```

#### 测试用例
- https://github.com/rencalo770/gengine/blob/master/test/increment_update_test.go

## 规则删除
#### 批量删除
- 当我们不需要某个规则的时候,我们想直接删除某个规则,gengine也提供了对应的规则删除方法

```golang
//ruleBuilder中
func (builder *RuleBuilder) RemoveRules(ruleNames []string) error 

//pool中
func (gp *GenginePool) RemoveRules(ruleNames []string) error 

```
#### 测试
- https://github.com/rencalo770/gengine/blob/master/test/remove_rules_test.go



