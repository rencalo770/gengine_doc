# gengine语法

### gengine DSL语法
```go
const rule = `
rule "rulename" "rule-describtion" salience  10
begin

//规则体

end`
```
如上,gengine DSL完整的语法块由如下几个组件构成:
- 关键字rule,之后紧跟"规则名称"和"规则描述",并且它们都是必须的. 当****一个gengine实例中有多个规则时,"规则名"必须唯一****,否则当有多个相同规则名的规则时,编译好之后只会存在一个
- 关键字salience,之后紧跟一个整数,表示的规则优先级,它们为非必须.当用户没有显式的指明优先级时,规则的优先级未知
- 关键字begin和end包裹的是规则体,也就是规则的具体逻辑

### 规则体语法

 规则体的语法支持或执行顺序,与主流的计算计语言(如golang、java、C/C++等)一致

#### 规则体支持的运算:
- 支持完整数值之间的加(+)、减(-)、乘(*)、除(/)四则运算,以及字符串之间的加法
- 完整的逻辑运算(&&、 ||、 !)
- 支持比较运算符: 等于(==)、不等于(!=)、大于(\>)、小于(<)、大于等于(\>=)、小于等于(<=)
- 支持小括号
- 优先级:括号, 非, 乘除, 加减, 逻辑运算(&&,||) 依次降低  

#### 规则体支持的基础数据类型
- string
- bool
- int, int8, int16, int32, int 64
- uint, uint8, uint16,uint32, uint64
- float32, float64

#### 不支持的特例
- 不支持函数或方法返回nil,或者nil值处理,提倡用户设置默认值处理。另外，map中使用不存在的key获取value，会有nil异常

#### 规则体支持的语法
- 完整的if .. else if .. else 语法结构,及其嵌套结构

```go
const rule_else_if_test =`
rule "elseif_test" "test"
begin

a = 8
if a < 1 {
	println("a < 1")
} else          if a >= 1 && a <6 {
	println("a >= 1 && a <6")
} else if a >= 6 && a < 7 {
	println("a >= 6 && a < 7")
} else if a >= 7 && a < 10 {
	println("a >=7 && a < 10")
} else        {
	println("a > 10")
}
end`
```

- conc{} 并发语句块 

```go
const rule_conc_statement  = `
rule "conc_test" "test" 
begin
	conc  { 
		a = 3.0
		b = 4
		c = 6.8
		d = "hello world"
        e = "you will be happy here!"
	}
	println(a, b, c, d, e)
end`
```


#### @name语法
- 在规则体中使用@name,指代的是当前规则名,@name在规则执行时,会被解析为规则名字符串(规则内的名字感知)
- 测试:https://github.com/rencalo770/gengine/blob/master/test/at_name_test.go

#### 注释
- 支持规则内的单行注释,注释以双斜杠(//)开头 

#### 自定义变量
- 用户自定义变量无需申明类型
- ****规则内定义的变量,只对当前规则可见,对其他规则不可见****(局部变量)
- ****使用dataContext注入的(变量)数据,对加载到gengine中的所有规则均可见****(全局变量)

#### 其他语法

- **为了使用简单、便于记忆,规则内只支持A.B形式的最长的二级调用,不支持A.B.C形式的三级及三级以上的调用(迪米特法则)**
- 为了尽量保证规则的书写和golang语法的无关性，**规则体内可以调用具有多返回的函数或方法,但当使用自定义变量接受函数、方法返回值时,只支持但返回的函数和方法调用**(也是间接建议用户为自己的业务实现设置适当的默认值,而不是直接报错)


