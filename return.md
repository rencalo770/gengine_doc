# 规则支持return语法
- 从gengine v1.4.1开始支持return语法,支持return语法的重要原因是进一步增强gengine中DSL与golang的交互能力. 
- gengine的return语法含义和常见的编程语言含义基本一致,即含义是: 一条规则(可理解为一个函数),遇上return语句时就返回,同时,如果在return后面还有处于return相同的可达路径上的代码,则编译出错.
- 但gengine的return带有一定的特色性, 具体请看如下实例

## gengine单实例
- 测试框架代码

```
//测试的框架代码
import (
	"gengine/builder"
	"gengine/context"
	"gengine/engine"
	"testing"
)

//rules是注入的规则, map[string]interface{}是每个规则的返回值
func framework(rules string) map[string]interface{} {
	dataContext := context.NewDataContext()
	dataContext.Add("getInt", getInt)
	ruleBuilder := builder.NewRuleBuilder(dataContext)
	e1 := ruleBuilder.BuildRuleFromString(rules)
	if e1 != nil {
		panic(e1)
	}

	eng := engine.NewGengine()
	e2 := eng.Execute(ruleBuilder, true)
	if e2 != nil {
		println(e2)
	}

	resultMap, _ := eng.GetRulesResultMap()
	return resultMap
}

func getInt() int {
	return 666
}
```

- 以下的所有测试代码(规则),都是上面测试框架样板代码上实现的
1.直接返回nil,此return用法仅用于中断当前规则执行
```
func Test_return_nil_1(t *testing.T) {

	//无返回值,返回nil
	ruleName := "return_in_statements"
	rule := `rule  "` + ruleName + `"
			 begin
			  return //无返回值,主要用于中断当前规则执行
      		 end	
			`
	returnResultMap := framework(rule)

	//无返回值,或者说返回nil
	r := returnResultMap[ruleName]
	if r == nil {
		println("return is nil")
	}
}
```

2.在一个规则中,gengine的return语句可以基于条件返回不同类型的值,这正是gengine的return语句的特色之处！ 
```go
//test 这里的表现和golang return略有不同
func Test_return_complex_if_return_int64(t *testing.T)  {

	ruleName := `return_in_statements`
	rule := `rule  "` + ruleName + `"
			 begin
			  a = 290	//修改这里,就可以得到不同的结果
			  if a < 100 {
				  return 5 + 6 * 6			
    		  }else if a >= 100 && a < 200  {
  				  return "hello world"	
              }else {
				  return true	
			  }
      		 end	
			`
	returnResultMap := framework(rule)
	r := returnResultMap[ruleName]

	// a < 100时 返回int64
	//i := r.(int64)

	//a >= 100 && a < 200时 返回string
	//i := r.(string)

	//a >= 200
	i := r.(bool)

	println("return--->", i)
}
```

- 另外,return语句不能出现在conc{}并发语句块中.如果这么写了,但在规则构建的时候,会报编译错误.
- return和StopTag的区别是,return仅能终止当前所在规则执行,stopTag等于true时,会终止此规则链中当前规则之后的所有规则的执行,但stopTag仅能用于最后
- 所有的测试用例位置,请看这里:https://github.com/rencalo770/gengine/blob/master/test/return_statement_test.go

## gengine pool对应的支持
- 在v1.4.1之前, gengine pool中的执行规则的方法,只有一个返回值,那就是标识规则是否执行出错的error对象
- 从v1.4.1开始,gengine pool中的执行规则的方法,有两个返回值,一个是标识规则是否执行出错的error对象,另一个是收集本次规则集执行之后,每个规则的返回值结果的对象```map[string]interface{}```,如果用户不想关注此返回值,忽略就行.
- 具体测试如下,测试代码位置：https://github.com/rencalo770/gengine/blob/master/test/pool_return_statements_test.go

```
import (
	"fmt"
	"gengine/engine"
	"testing"
	"time"
)

func random() int {
	return time.Now().Nanosecond()
}


func Test_pool_return_statments(t *testing.T) {

	ruleName := "test_pool_return"
	rule := `rule "` +ruleName +  `"  
			begin
				return random()
			end
			`

	apis := make(map[string]interface{})
	apis["print"] = fmt.Println
	apis["random"] = random
    //实例化pool时,最好仅注入无状态函数,供所有pool内实例执行,这样可以很好的避免线程安全问题
	pool, e1 := engine.NewGenginePool(1, 2, 1, rule, apis)
	if e1 != nil {
		println(fmt.Sprintf("e1: %+v", e1))
	}

    //具体执行时,注入与此次规则相关的具体结构体或方法,这样可以很好的避免线程安全问题
	data := make(map[string]interface{})
	e2, rrm1 := pool.ExecuteRulesWithMultiInput(data)
	if e2 != nil {
		panic(e2)
	}

	i1 := rrm1[ruleName]
	ix1 := i1.(int)
	println("ix1--->", ix1)

	e3, rrm2 := pool.ExecuteRulesWithMultiInput(data)
	if e3 != nil {
		panic(e2)
	}

	i2 := rrm2[ruleName]
	ix2 := i2.(int)
	println("ix2--->", ix2)

	i3 := rrm1[ruleName]
	ix3 := i3.(int)
	println("ix3--->", ix3)
}
```




