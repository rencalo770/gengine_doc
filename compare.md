## 前言
- 最近总是有人在问我gengine和gopher-lua的区别,尤其是执行性能上的区别.其实,我觉得这种想法是没多大意义的:
因为你觉得哪个框架好用,能满足你的业务需求,你用哪个就行.另外,两框架的文档都挺全的,还不如先去好好看看文档,跑两个demo自己体会以下.如果连文档都不想看,那能怎么办呢？
- 但因为问的人越来越多了,所以我觉得还是有必要做一个详细的测试对比.

## 基准测试用例
- 为了对比gopher-lua和gengine的性能,我们使用完全相同的功能对两个框架进行测试
- 具体的测试功能为"求斐波拉契数列第n项的值" 
- 当前最优秀的 gopher-lua github地址 https://github.com/yuin/gopher-lua (知乎贴:https://zhuanlan.zhihu.com/p/33471484)
- gengine    github地址 https://github.com/rencalo770/gengine

### 使用的机器配置
- 系统 macOS  
- CPU intel i7
- 内存 16GB 2667MHz 

### go.mod依赖
```go
go 1.15

require (
	gengine v1.3.9
	github.com/yuin/gopher-lua v0.0.0-20200816102855-ee81675732da
)

replace gengine => github.com/rencalo770/gengine v1.3.9
```

### gopher-lua纯lua式写法
```go

import (
	"fmt"
	lua "github.com/yuin/gopher-lua"
	"testing"
	"time"
)

const lua_fib = `
function fib(n)
	if n < 2 then
		return 1
	end
	return fib(n-1) + fib(n-2)
end
`

func Test_lua(t *testing.T) {

	L := lua.NewState()
	defer L.Close()
	// 加载fib.lua
	if err := L.DoString(lua_fib); err != nil {
		panic(err)
	}

	start := time.Now()
	defer logTime(start)
	// 调用fib(n)
	n := 20 //此处输入参数
	err := L.CallByParam(lua.P{
		Fn:      L.GetGlobal("fib"), // 获取fib函数引用
		NRet:    1,                  // 指定返回值数量
		Protect: true,               // 如果出现异常，是panic还是返回err
	}, lua.LNumber(n)) // 传递输入参数n=10
	if err != nil {
		panic(err)
	}
	// 获取返回结果
	ret := L.Get(-1)
	// 从堆栈中扔掉返回结果
	//L.Pop(1)
	// 打印结果
	res, ok := ret.(lua.LNumber)
	if ok {
		println(int(res))
	} else {
		fmt.Println("unexpected result")
	}

}

func logTime( t time.Time,) {
	println( "cost_time:", time.Since(t).Microseconds())
}
```

### gopher-lua的汇编式写法
```go

import (
	lua "github.com/yuin/gopher-lua"
	"testing"
	"time"
)

func Test_inject(t *testing.T) {

	L := lua.NewState()
	defer L.Close()
	meta := L.NewTable()
	L.SetGlobal("Counter", meta)
	// 注册函数
	//L.SetField(meta, "new", L.NewFunction(newCounter))
	//L.SetField(meta, "incr", L.NewFunction(incrCounter))
	L.SetField(meta, "get", L.NewFunction(getCounter))
	L.SetField(meta, "fib", L.NewFunction(fibCounter))
	// 使用lua代码测试效果
	start := time.Now()
	defer logTime(start)
	err := L.DoString(`
		counter = Counter:fib(40)
		print(counter:get())
	`)
	if err != nil {
		panic(err)
	}
}


func fibCounter(L *lua.LState) int {
	c := L.NewTable()
	self := L.CheckTable(1)
	value := lua.LNumber(0)
	// 第二个为可选参数
	if L.GetTop() >= 2 {
		value = L.CheckNumber(2)
	}

	value = lua.LNumber(float64(fib(int(value))))
	L.SetField(c, "value", value)
	L.SetMetatable(c, self)
	L.SetField(self, "__index", self)
	// 返回值压栈
	L.Push(c)
	// 返回[函数返回值的个数]
	return 1
}


func fib(n int) int{
	if n < 2 {
		return 1
	}
	return fib(n-1) + fib(n-2)
}


func getCounter(L *lua.LState) int {
	self := L.CheckTable(1)
	value := L.GetField(self, "value").(lua.LNumber)
	// 返回值压栈
	L.Push(value)
	// 返回[函数返回值的个数]
	return 1
}


func logTime( t time.Time,) {
	println( "cost_time:", time.Since(t).Microseconds())
}
```


### gengine代码
```go

import (
	"fmt"
	"gengine/builder"
	"gengine/context"
	"gengine/engine"
	"testing"
	"time"
)

const gengine_fib = `
rule "1" "2" 
begin
r.R = fib(20) //此处输入参数
end
`

func Fib(n int) int{
	if n < 2 {
		return 1
	}
	return Fib(n-1) + Fib(n-2)
}


func Test_gengine(t *testing.T) {

	type Result struct{
		R int
	}

	r := &Result{}

	dataContext := context.NewDataContext()
	dataContext.Add("fib", Fib)
	dataContext.Add("r", r)
	dataContext.Add("println", fmt.Println)

	ruleBuilder := builder.NewRuleBuilder(dataContext)
	e1 := ruleBuilder.BuildRuleFromString(gengine_fib)
	if e1 != nil {
		panic(e1)
	}

	start := time.Now()
	defer logTime(start)

	gengine := engine.NewGengine()
	e := gengine.Execute(ruleBuilder, true)
	if e != nil {
		panic(e)
	}
	println(r.R)
}

func logTime( t time.Time,) {
	println( "cost_time:", time.Since(t).Microseconds())
}

```

### 对比结果
求斐波拉契数列第n项值的结果

| 第n项执行耗时 |gopher-lua纯lua写法(平均耗时) |gopher-lua汇编式写法(平均耗时) |gengine执行耗时(平均耗时) |
| -------- | -------- | --------- |----|
|n = 1 |0.008ms|0.15ms|0.045ms|
|n = 2 |0.01ms|0.15ms|0.045ms|
|n = 3 |0.012ms|0.15ms|0.045ms|
|n = 4 |0.015ms|0.15ms|0.045ms|
|n = 5 |0.02ms|0.15ms|0.045ms|
|n = 10 |0.033ms|0.15ms|0.045ms|
|n = 11 |0.05ms|0.15ms|0.045ms|
|n = 20 |2.9ms|0.2ms|0.08ms|
|n = 30 |365ms|4.3ms|4ms|
|n = 40 |43800ms |500ms|490ms|95|
|n = 50 |执行时间太长无法获知|61000ms|61000ms|-|

### 基于以上统计数据得到的结论
- 因为gopher-lua和gengine都提供了池模式,所以实例个数均不是两者性能的限制条件, 因此我们把性能对比限制单实例上,代码相同,传入参数不同时的情况,
最终可以看出:

|对比|gopher纯lua写法和gengine对比 | gopher-lua汇编写法和gengine对比|
|---|---|---|
|1 | n < 11, gopher纯lua写法执行性能高于gengine | n < 42时, gopher汇编式写法执行性能低于gengine|
|2 |n = 11, gopher-lua和gengine的执行性能持平  | 当n较大时，gopher-lua的汇编写法性能才能和gengine持平|
|3| n > 11, gopher-lua的执行性能开始低于gengine,随着n越大,纯lua写法的执行性能和gengine的差距越来越大,n=40时已达到95倍| 大n情况下, gopher-lua汇编写法才能和gengine持平|

- 另外,从gopher-lua的两种写法可以看出, gopher-lua的汇编写法虽然与gengine获得了相差不多的性能, 但与golang的交互相当复杂(API较多),获得结果的方式很不友好,且容易出错.(适合大佬级人物)
- 当使用gopher纯lua写法时, 当逻辑较复杂以后, 又难以获得与gengine相似的高性能.

### 两者的设计实现
| 对比项 | gopher-lua | gengine | 说明 |
| -------- | -------- | --------- |----|
|底层依赖|golang、Yacc、lua | golang、Antlr | Yacc 和 Antlr都是编译领域的前端著名框架 |
|侧重点|追求对lua在golang上的完整的生态实现| 追求简单的语法和对各种业务场景的细粒度规则执行模式的支持|两者侧重点不一样|
|使用选择|适合熟悉lua + golang的大佬 | 适合仅了解golang的大佬|gopher-lua实现业务逻辑,有业务逻辑从golang"逃逸"到lua的风险; gengine语法是golang最简单的那部分语法的子集,不想用了,可以随时抛弃,无任何业务迁移与反向迁移的难度与风险|


### 最后
-还是回到开头, 选择你喜欢的就好, 不必纠结于哪一种框架


















