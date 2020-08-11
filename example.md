# gengine示例
先看一个gengine使用的小示例

#### 一个小示例
```go

package test

import (
	"bytes"
	"fmt"
	"gengine/base"
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
	knowledgeContext := base.NewKnowledgeContext()
	ruleBuilder := builder.NewRuleBuilder(knowledgeContext,dataContext)


	start1 := time.Now().UnixNano()
    //构建规则
	err := ruleBuilder.BuildRuleFromString(rule1) //string(bs)
	end1 := time.Now().UnixNano()

	logrus.Infof("rules num:%d, load rules cost time:%d", len(knowledgeContext.RuleEntities), end1-start1 )

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
- User是需要被注入到gengine中的结构体；结构体在注入之前需要被初始化；结构体需要以指针的形式被注入，否则无法在规则中改变其属性值
- rule1是以字符串定义的具体规则
- dataContext用于接受注入的数据(结构体、方法等)
- ruleBuilder用于编译字符串形式的规则
- engine接受ruleBuilder，并以某一种执行模式执行其中的规则

#### 小技巧
 通过示例可以发现，****规则的编译构建和执行是异步的****. 因此, 用户可使用此特性,在不停服的情况下进行更新规则.通常做法是,创建一个新的rulebuilder接受新的规则并编译构建,当编译构建完毕,使用新的rulebuilder指针来替换老的rulebuilder指针，进而达到更新gengine中执行的规则的目的




