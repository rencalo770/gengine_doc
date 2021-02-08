# 高级扩展部分

### 完全热加载实现-背景
- 到现在,我们可以看到,我们在gengine中使用的API, 都是在gengine启动之前预先定义好的,然后初始化完成时,注入到gengine当中才能使用.这种使用模式当然是好的,也基本能满足所有业务场景的需求:如用户可以事先定义好几个接口,然后使用这几个定义好的接口,通过变换参数的方法,来获得规则在执行过程中所需要的不同的数据(基于名称的方式来访问数据指标).
- 但是有这种可能:当前的定义的所有函数均不能满足需求,用户必须要定义新的方法或函数来实现新的功能.

### 现实状况
golang是一种静态类型语言,不如java般灵活,想要实现动态加载代码并执行,只能使用插件的形式。gengine包装了plugin的调用实现,使用plugin的方式进行代码热加载并执行。

### 说明
- gengine包装的动态加载代码的方法,只需要在遵守少量的规定,使用将极其简单
- gengine从v1.4.9便支持了基于golang plugin的完全动态加载实现

### golang plugin使用
- 插件编写

```go
//package必须是main
package main


//为了演示完整与gengine无关的go plugin使用实现，需要一个接口
type Man interface {
	SaveLive() error
}

type SuperMan struct {

}

func (g *SuperMan) SaveLive() error {

	println("execute finished...")
	return nil
}

//go build -buildmode=plugin -o=plugin_M_m.so plugin_superman.go

// exported as symbol named "M",必须大写开头
var M = SuperMan{}
```

- 插件编写注意事项
1. package 名必须为main
2. 导出变量必须是大写开头,且不能与其他struct或接口定义重名
3. 执行```go build -buildmode=plugin -o=plugin_M_m.so plugin_superman.go```命令,生成.so插件文件


- 插件使用测试

```go
func Test_pligin(t *testing.T) {

	dir, err := os.Getwd()
	if err!=nil {
		panic(err)
	}

	// load module 插件您也可以使用go http.Request从远程下载到本地,在加载做到动态的执行不同的功能
	// 1. open the so file to load the symbols
	plug, err := plugin.Open(dir + "/plugin_M_m.so")
	if err != nil {
		panic(err)
	}
	println("plugin opened")

	// 2. look up a symbol (an exported function or variable)
	// in this case, variable Greeter
	m, err := plug.Lookup("M") //大写
	if err != nil {
		panic(err)
	}

	// 3. Assert that loaded symbol is of a desired type
	man, ok := m.(Man)
	if !ok {
		fmt.Println("unexpected type from module symbol")
		os.Exit(1)
	}

	// 4. use the module
	if err := man.SaveLive(); err != nil {
		println("use plugin man failed, ", err)
	}
}

```

### gengine基于plugin的热加载实现
1. 由上面的插件使用可看出,使用插件需要这么几个步骤: a.定义插件go文件, b.命令生成.so文件, c.Open加载插件 d.Lookup查找插件定义中导出的api, e.接口类型转换 f.使用插件提供的功能
2. 在gengine中使用插件,需要步骤a和b,然后告诉gengine插件的.so文件位置, 就可以在gengine规则配置中使用了
3. 但是,要能在gengine正确的使用plugin,还需要遵守一点规范(要求): a.生成的.so文件,必须是plugin_exportName_apiName.so这种形式,其中exportName就是插件实现中的导出名,必须大写开头, apiName是在gengine规则配置中使用的插件名称
4. 请看下面的测试

- 单实例plugin加载测试
```go
func Test_plugin_with_gengine(t *testing.T)  {

	dir, err := os.Getwd()
	if err!=nil {
		panic(err)
	}

	dc := context.NewDataContext()
	//3.load plugin into apiName, exportApi
	_, _, e := dc.PluginLoader( dir + "/plugin_M_m.so")
	if e != nil {
		panic(e)
	}

	dc.Add("println", fmt.Println)
	ruleBuilder := builder.NewRuleBuilder(dc)
	err = ruleBuilder.BuildRuleFromString(`
	rule "1"
	begin
	 
	//this method is defined in plugin
	err = m.SaveLive()

	if isNil(err) {
	   println("err is nil")
	}
	end
	`)

	if err != nil {
		panic(err)
	}
	gengine := engine.NewGengine()
	err = gengine.Execute(ruleBuilder, false)

	if err!=nil {
		panic(err)
	}
}
```

- pool plugin测试

```go
func Test_plugin_with_pool(t *testing.T)  {

	rule :=`
	rule "1"
	begin
	 
	//this method is defined in plugin
	err = m.SaveLive()

	if isNil(err) {
	   println("err is nil")
	}
	end`

	apis := make(map[string]interface{})
	apis["println"] = fmt.Println
	pool, e := engine.NewGenginePool(1, 2, 1, rule, apis)
	if e != nil {
		panic(e)
	}

	dir, err := os.Getwd()
	if err!=nil {
		panic(err)
	}

	e = pool.PluginLoader( dir + "/plugin_M_m.so")
	if e != nil {
		panic(e)
	}
	data := make(map[string]interface{})
	e, _ = pool.Execute(data, true)
	if e != nil {
		panic(e)
	}

	//twice execute
	e, _ = pool.Execute(data, true)
	if e != nil {
		panic(e)
	}
}

```

### 测试位置
- 测试代码位置 https://github.com/rencalo770/gengine/tree/master/test/plugin








