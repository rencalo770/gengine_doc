# N-M执行模式支持
- N-M执行模式, 将规则集中的规则基于优先级从高到底,依次排序,然后将规则集分成两阶段,第一阶段为前N个最高优先级的规则,第二个阶段为剩下的M个规则,即第一阶段的规则优先级一定比第二阶段的规则优先级高,由此产生了如下执行子模式


### 单实例说明与API

- 说明

| 执行模式 | 解释 | 
| -------- | -------- | 
| nSort - mConcurrent | 前n个规则顺序执行,执行完毕,并发执行剩下的m个规则|  
| nConcurrent - mSort | 前n个规则并发执行,执行完毕,顺序执行剩下的m个规则|
| nConcurrent - mConcurrent | 前n个规则并发执行,执行完毕,并发执行剩下的m个规则|
| nSort - mSort       | 前n个规则顺序执行,执行完毕,顺序执行剩下的m个规则;此模式退化为顺序执行模式,也就是普通的顺序模式 |

- 对应的API

| API | 解释 | 
| -------- | -------- | 
| ``` (g *Gengine) ExecuteNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool) error ```| 前n个规则顺序执行,执行完毕,并发执行剩下的m个规则;bool控制顺序执行阶段,有规则执行出错时,是否继续执行,true表示继续执行 |
| ``` (g *Gengine) ExecuteNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool)  error``` | 前n个规则并发执行,执行完毕,顺序执行剩下的m个规则;bool控制并发执行完毕之后,有规则执行出错时,是否继续执行,true表示继续执行  |
| ``` (g *Gengine) ExecuteNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool)  error ``` | 前n个规则并发执行,执行完毕,并发执行剩下的m个规则;bool控制第一阶段并发执行完毕,有规则出错时,是否继续执行第二个并发阶段,true表示继续执行|

- 注意事项:
1. 参数 n 和 m 必须都大于0;
2. 规则集中的规则个数必须大于等于 n+m;
3. 详细的测试,请看: https://github.com/rencalo770/gengine/blob/master/test/n_m_model_test.go

### 基于选择式的N-M执行模式
- 我们可以先从规则集中基于指定的规则名选出一批规则,然后再使用N-M模式执行,对应的API如下:

| API | 解释 | 
| -------- | -------- | 
| ``` (g *Gengine) ExecuteSelectedNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) error ```| 基于指定的规则名称集合names,选出一批规则,前n个规则顺序执行,执行完毕,并发执行剩下的m个规则;bool控制顺序执行阶段,有规则执行出错时,是否继续执行,true表示继续执行 |
| ``` (g *Gengine) ExecuteSelectedNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool, names []string) error``` | 基于指定的规则名称集合names,选出一批规则,前n个规则并发执行,执行完毕,顺序执行剩下的m个规则;bool控制并发执行完毕之后,有规则执行出错时,是否继续执行,true表示继续执行  |
| ``` (g *Gengine) ExecuteSelectedNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) error ``` | 基于指定的规则名称集合names,选出一批规则,前n个规则并发执行,执行完毕,并发执行剩下的m个规则;bool控制第一阶段并发执行完毕,有规则出错时,是否继续执行第二个并发阶段,true表示继续执行|

- 注意事项:
1. 参数 n 和 m 必须都大于0;
2. 选定的规则集中的规则个数必须等于 n+m;
3. 指定的规则名称集中的所有名称对应的规则必须存在
4. 详细的测试,请看:  https://github.com/rencalo770/gengine/blob/master/test/n_m_model_test.go


### 基于gengine pool的N-M模式
- API

| API | 解释 | 
| -------- | -------- | 
| ``` (gp *GenginePool) ExecuteNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool)  (error, map[string]interface{}) ```| 前n个规则顺序执行,执行完毕,并发执行剩下的m个规则,bool控制顺序执行阶段,有规则执行出错时,是否继续执行,true表示继续执行 |
| ``` (gp *GenginePool) ExecuteNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool)   (error, map[string]interface{})``` | 前n个规则并发执行,执行完毕,顺序执行剩下的m个规则,bool控制并发执行完毕之后,有规则执行出错时,是否继续执行,true表示继续执行  |
| ``` (gp *GenginePool) ExecuteNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool)  (error, map[string]interface{}) ``` | 前n个规则并发执行,执行完毕,并发执行剩下的m个规则,bool控制第一阶段并发执行完毕,有规则出错时,是否继续执行第二个并发阶段,true表示继续执行|
| ``` (gp *GenginePool) ExecuteSelectedNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string)  (error, map[string]interface{}) ```| 基于指定的规则名称集合names,选出一批规则, 前n个规则顺序执行,执行完毕,并发执行剩下的m个规则,bool控制顺序执行阶段,有规则执行出错时,是否继续执行,true表示继续执行 |
| ``` (gp *GenginePool) ExecuteSelectedNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool, names []string)  (error, map[string]interface{})``` | 基于指定的规则名称集合names,选出一批规则,前n个规则并发执行,执行完毕,顺序执行剩下的m个规则,bool控制并发执行完毕之后,有规则执行出错时,是否继续执行,true表示继续执行  |
| ``` (gp *GenginePool) ExecuteSelectedNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string)  (error, map[string]interface{}) ``` | 基于指定的规则名称集合names,选出一批规则,前n个规则并发执行,执行完毕,并发执行剩下的m个规则,bool控制第一阶段并发执行完毕,有规则出错时,是否继续执行第二个并发阶段,true表示继续执行|

其中 ```map[string]interface{}``` 是收集每个策略的返回值(如果策略有返回值,map中的key是策略的名称, value是具体的返回值)

 -注意事项
 1. 参数 n 和 m 必须都大于0;
 2. 规则集中的规则个数必须大于等于 n+m;
 3. 选定的规则集中的规则个数必须等于 n+m;
 4. 指定的规则名称集中的所有名称对应的规则必须存在;
 5. 详细的测试,请看: https://github.com/rencalo770/gengine/blob/master/test/n_m_model_pool_test.go

