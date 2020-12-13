# gengine架构


- gengine从v1.2.0开始,进一步简化架构,用户只需要很少的改动,便可从老版本迁移到新版本上来.
- 从v1.2.0开始,用户也仅需关注gengine的```dataContext```, ```ruleBuilder``` , ```engine``` 和```GenginePool```这4个API
- 从v1.2.0开始, gengine框架简化为4部分组成,调整之后,代码更易理解,更容易使用,具体直接表现在代码结构上,具体说明如下:

### dataContext
- 代码位置: https://github.com/rencalo770/gengine/blob/master/context/data_context.go
- ```dataContext```允许用户注入将要在规则代码中使用的API
- 举例:
```go
	dataContext := context.NewDataContext()
	//用户注入需要在规则代码中使用的API
	dataContext.Add("println",fmt.Println)
```


### ruleBuilder
- 代码位置: https://github.com/rencalo770/gengine/tree/master/builder
- ruleBuilder接受一个dataContext实例作为参数,并将用户传入的字符串构建出可执行的代码(规则)
- 举例:
```go
const rule = `
rule "1" "rule-des" salience 10
begin
println("hello world, gengine!")
end
`

	dataContext := context.NewDataContext()
	//用户注入需要在规则代码中使用的API
	dataContext.Add("println",fmt.Println)
	
	ruleBuilder := builder.NewRuleBuilder(dataContext)
	err := ruleBuilder.BuildRuleFromString(rule)
```


### engine
- 代码位置:https://github.com/rencalo770/gengine/blob/master/engine/gengine.go
- ```engine``` 允许用户构建gengine实例,并传入实例化好的```ruleBuilder```使用不同的模式执行规则代码
- 举例:
```go
const rule = `
rule "1" "rule-des" salience 10
begin
println("hello world, gengine!")
end
`

	dataContext := context.NewDataContext()
	//用户注入需要在规则代码中使用的API
	dataContext.Add("println",fmt.Println)
	
	ruleBuilder := builder.NewRuleBuilder(dataContext)
	err := ruleBuilder.BuildRuleFromString(rule)
	
	eng := engine.NewGengine()
	err := eng.Execute(ruleBuilder, true)
```

### GenginePool
- gengine实例池,提供给用户在高QPS下使用,为了解决高并发和线程安全问题的API,具体用法在本文档的"引擎池"章节有详细叙述

### internal
- internal文件夹中的代码是gengine的核心,用户可以不必关心其实现



  
  
  


