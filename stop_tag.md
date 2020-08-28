# 基于标志位来控制规则是否继续执行

### 背景
某些业务场景有这种需求: <br/>
- 如在使用顺序模式执行规则的场景,有规则"1","2","3","4","5","6",当用户执行了规则"2"之后,不想继续执行后面的规则.<br/>
- 如在使用混合模式执行规则的场景,有规则"1","2","3","4","5","6",其中规则"1"的优先级最高,剩下的规则优先级都比"1"低,当用户执行了"1"之后,不想继续执行后面的规则.<br/>
- 针对以上需求,或者基于性能考量,**gengine支持用户使用StopTag的方式,来满足以上两种场景的需求**.

### 示例

#### 顺序模式场景
```go
func (g *Gengine) ExecuteWithStopTagDirect(rb *builder.RuleBuilder, b bool, sTag *Stag) error
```
- 测试用例:https://github.com/rencalo770/gengine/blob/master/test/stop_tag_test/stop_tag_in_sort_model_test.go

对应的规则引擎pool的方法支持:

```go
func (gp *GenginePool)ExecuteRulesWithStopTag(reqName string, req interface{}, respName string, resp interface{}, stag *Stag) error 

//and
func (gp *GenginePool)ExecuteRulesWithMultiInputAndStopTag(data map[string]interface{}, stag *Stag) error
```

#### 混合模式场景
```go 
func (g *Gengine) ExecuteMixModelWithStopTagDirect(rb * builder.RuleBuilder, sTag *Stag)
```

- 测试用例: https://github.com/rencalo770/gengine/blob/master/test/stop_tag_test/stop_tag_in_mix_model_test.go
对应的规则引擎pool的方法支持:

```go
func (gp *GenginePool)ExecuteRulesWithStopTag(reqName string, req interface{}, respName string, resp interface{}, stag *Stag) error 

//and
func (gp *GenginePool)ExecuteRulesWithMultiInputAndStopTag(data map[string]interface{}, stag *Stag) error
```
- 注意,在规则引擎pool中,顺序模式和混合模式是相同的两个接口,是因为执行模式的控制分离到了```func (gp *GenginePool)SetExecModel(execModel int) error```