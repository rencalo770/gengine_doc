# 规则内的并发实现
- 在很多业务场景下,一个规则,需要很多的数据指标支持,因为数据量的缘故,这些数据指标往往不可能存储在本地.
- 因此,如果在规则内顺序地从网络上取很多个指标时,会严重拖慢规则的执行性能.
- 基于此考量,**gengine支持规则内的并发函数调用或赋值语句**.其语法相当简单,使用conc{}语句块来包裹需要并发调用的接口即可.

```go
const rule_conc_statement  = `
rule "conc_test" "test" 
begin
	conc  { // this should be used in time-consuming operation, such as the operation contains network connection (get data from remote based on network) 
		println("hihihi")
		a = 3.0
		b = 4
		c = 6.8
		d = "hello world"
        e = "you will be happy here!"
		sout("heheheheh")
	}
	println(a, b, c, d, e)
end`

//define
func Sout(str string)  {
	println("----",str)
}

//inject
dataContext.Add("println",fmt.Println)
dataContext.Add("sout", Sout)
```

测试:https://github.com/rencalo770/gengine/blob/master/test/conc_statement_test.go
- conc{}语句块内支持放置方法、函数、赋值语句
- conc{}语句块内可以为空,或有一个或多个方法、函数或赋值语句
- conc{}语句块为空时,不产生任何影响,当仅有一条语句时,退化为顺序执行,当多2条或2条以上的语句时,会进行并发执行
- 用户需要自己来保证在conc{}中的调用的api是线程安全的
- 多执行几次测试,你会发现conc{}内的输出顺序每次都不一样













