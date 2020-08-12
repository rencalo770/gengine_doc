# 规则内的继承调用
- golang的继承和实现不如java直接明白,golang的继承其实是组合模式,或者可以直接说golang没有继承这一概念
- 方法实现了某接口,删除接口定义时,对golang代码的编译执行没有任何影响.
- 前面已经提到,在gengine中,只要不超过A.B形式的两层调用写法,都是合法的语法.

```go

//define
type Man struct {
	Eat string
	Drink string
}

// define
type Father struct {
	*Man
	Son string
}

//rule
const ext_rule = `
rule "extends test" "extends test" 
begin
	Father.Son = "tom"
	Sout(Father.Son)
	Father.Eat= "apple"
	Sout(Father.Eat)
end
`
```
测试: https://github.com/rencalo770/gengine/blob/master/test/complex/extends_test.go
