# gengine语法

### gengine DSL语法
```go
const rule = `
rule "rulename" "rule-describtion" salience  10
begin

//logic

end`
```
如上，gengine DSL完整的语法块由如下几个组件构成：
- 关键字rule之后, 紧跟"规则名称"和"规则描述",且它们都是必须的. 当一个gengine实例中有多个规则时，"规则名"必须唯一，否则当有多个相同规则名的规则时，最终只会存在一个
- 关键字salience之后, 紧跟一个整数，表示的规则优先级，它们为非必须。当没有显式的指明优先级时，规则的优先级未知
- 关键字begin和end包裹的是规则的具体逻辑，也叫规则体

### 规则体语法
规则体支持的语法如下：
- 完整的if .. else if .. else 语法结构
- 支持完整的加(+)、减(-)、乘(*)、除(/)四则运算
- 

 



