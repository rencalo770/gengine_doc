# gengine支持的数据结构

gengine主要支持4种数据结构,分别是golang的struct、map、array和slice, 为了保持简洁、以及语言的无关性, 支持这4种结构的如下操作:

### 注入struct
注入struct必须是以指针的形式注入,否则无法在规则中改变结构体的属性值,注入结构体示例如下:

```go
	user := &User{
		Name: "Calo",
		Age:  0,
		Male: true,
	}

	dataContext := context.NewDataContext()
    //inject
	dataContext.Add("User", user)
```
测试: https://github.com/rencalo770/gengine/blob/master/test/mutli_rules_test.go 

### 注入map

gengine只能基于key对map取值或设置值,且注入的map,必须是指针map,或者附着于结构体指针的map; 当以非指针形式的map注入dataContext,则只能用key取值,而不能设置值
```go
    //define
    type MS struct {
	    MII *map[int]int
	    MSI map[string]int
	    MIS map[int]string
    }
    
    //init
	MS := &MS{
		MII: &map[int]int{1: 1},
		MSI: map[string]int{"hello": 1},
		MIS: map[int]string{1: "helwo"},
	}
    
    //define
	var MM map[int]int
	MM = map[int]int{1:1000,2:1000}
    
    //inject
	dataContext := context.NewDataContext()
	dataContext.Add("MS", MS)
	//single map inject, must be ptr
	dataContext.Add("MM", &MM)
```
测试:https://github.com/rencalo770/gengine/blob/master/test/map_slice_array/map_test.go 

### 注入数组
gengine只能基于index对数组取值或设置值,且注入的数组,必须为指针数组,或者附着与结构体指针的数组,当以非指针形式的数组注入dataContext.则只能用index取值,而不能设置值
```go
    
    //define
    type AS struct {
	    MI *[3]int
	    MM [4]int
    }
    
    //init
    AS := &AS{
   		MI: &[3]int{},
   		MM: [4]int{},
   	}
    
    define
   	var AA [2]int
   	AA = [2]int{1, 2}
    
   	dataContext := context.NewDataContext()
   	dataContext.Add("PrintName",fmt.Println)
   	dataContext.Add("AS", AS)
   	//single array inject, must be ptr
   	dataContext.Add("AA", &AA)
```
测试: https://github.com/rencalo770/gengine/blob/master/test/map_slice_array/array_test.go

### 注入slice
gengine只能基于index对切片取值或设置值,且注入的切片,必须为指针切片,或者附着与结构体指针的切片,当以非指针形式的切片注入dataContext.则只能用index取值,而不能设置值
```go
    
    //define
    type SS struct {
	    MI []int
	    MM *[]int
    }

    //init
	SS := &SS{
		MI: []int{1,2,3,4},
		MM: &[]int{9,1,34,5},
	}
    
    //define
	var S []int
	S = []int{1, 2, 3}

	dataContext := context.NewDataContext()
	dataContext.Add("PrintName",fmt.Println)
	dataContext.Add("SS", SS)
   	//single slice inject, must be ptr
	dataContext.Add("S", &S)

```
测试: https://github.com/rencalo770/gengine/blob/master/test/map_slice_array/slice_test.go 


