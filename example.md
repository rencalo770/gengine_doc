# gengine示例
先看一个gengine使用的小示例

#### 示例
```go

package test

import (
	"bytes"
	"fmt"
	"gengine/builder"
	"gengine/context"
	"gengine/engine"
	"github.com/sirupsen/logrus"
	"io/ioutil"
	"strconv"
	"strings"
	"testing"
	"time"
)

//定义想要注入的结构体
type User struct {
	Name string
	Age  int64
	Male bool
}

func (u *User)GetNum(i int64) int64 {
	return i
}

func (u *User)Print(s string){
	fmt.Println(s)
}

func (u *User)Say(){
	fmt.Println("hello world")
}

//定义规则
const rule1 = `
rule "name test" "i can"  salience 0
begin
		if 7 == User.GetNum(7){
			User.Age = User.GetNum(89767) + 10000000
			User.Print("6666")
		}else{
			User.Name = "yyyy"
		}
end
`
func Test_Multi(t *testing.T){
	user := &User{
		Name: "Calo",
		Age:  0,
		Male: true,
	}

	dataContext := context.NewDataContext()
	//注入初始化的结构体
    dataContext.Add("User", user)

	//init rule engine
	ruleBuilder := builder.NewRuleBuilder(dataContext)


	start1 := time.Now().UnixNano()
    //构建规则
	err := ruleBuilder.BuildRuleFromString(rule1) //string(bs)
	end1 := time.Now().UnixNano()

	logrus.Infof("rules num:%d, load rules cost time:%d", len(ruleBuilder.Kc.RuleEntities), end1-start1 )

	if err != nil{
		logrus.Errorf("err:%s ", err)
	}else{
		eng := engine.NewGengine()

		start := time.Now().UnixNano()
        //执行规则
		err := eng.Execute(ruleBuilder,true)
		println(user.Age)
		end := time.Now().UnixNano()
		if err != nil{
			logrus.Errorf("execute rule error: %v", err)
		}
		logrus.Infof("execute rule cost %d ns",end-start)
		logrus.Infof("user.Age=%d,Name=%s,Male=%t", user.Age, user.Name, user.Male)
	}
}
```

#### 示例解释
- User是需要被注入到gengine中的结构体；结构体在注入之前需要被初始化；结构体需要以指针的形式被注入,否则无法在规则中改变其属性值
- rule1是以字符串定义的具体规则
- dataContext用于接受注入的数据(结构体、方法等)
- ruleBuilder用于编译字符串形式的规则
- engine接受ruleBuilder,并以用户选定的执行模式执行加载好的规则

#### 小技巧
- 通过示例可以发现,****规则的编译构建和执行是异步的****. 因此,用户可使用此特性,在不停服的情况下进行更新规则.
- 通常做法是,创建一个新的ruleBuilder接受新的规则并编译构建,当编译构建完毕,使用新的ruleBuilder指针来替换老的ruleBuilder指针,进而达到更新gengine中执行的规则的目的.
 另外,用户还可以通过ruleBuilder来进行异步的语法检测.
- 需要注意的是,编译构建规则是一个CPU密集型的事情,通常只有规则被用户更新的时候才去编译构建更新;
- 当规则较少的时候，可以选择全量更新，当规则多的时候，最好使用增量更新的方式
- 为了确保状态隔离,如果你不愿意使用gengine pool,最好是一个ruleBuilder对应一个gengine实例,同时注入的API确保是并发安全的;如果你使用gengine pool,那么你就不必担心状态问题,但依旧需要你确保你注入的api是并发安全的.
