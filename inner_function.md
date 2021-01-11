# 内置函数
- 为了处理golang语言的nil,简化用户使用,gengine目前提供了一个唯一的内置函数isNil()
- isNil的参数只有一个,且可以是任意类型的;其返回值是true或false
- 当你有不确定其返回值是否为nil,而导致规则不可正常执行时,都可以使用isNil函数来判别一下

## 一般规则
- 基础类型永远都不是nil,都是其对应的默认值
- 非基础类型比较可能为nil,具体测试详见如下
- array:  https://github.com/rencalo770/gengine/blob/master/test/map_slice_array/array_nil_value_test.go
- map:    https://github.com/rencalo770/gengine/blob/master/test/map_slice_array/map_nil_value_test.go
- slice:  https://github.com/rencalo770/gengine/blob/master/test/map_slice_array/slice_nil_value_test.go


