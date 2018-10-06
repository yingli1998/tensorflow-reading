# Protocol Buffers 格式数据结构 

TensorFlow 用 Protocol Buffers 定义了很多特别重要的数据格式. 下面将会列出一种重要的数据结构. 



### 与图相关的数据结构

---

下面介绍与图相关的一些数据结构

#### Ⅰ. GraphDef 

***

定义图的数据结构是 ` GraphDef`

```c++
message GraphDef{
    repeated NodeDef node = 1;
    VersionDef versions = 4;
    int32 version = 3 [deprecated = true];
    FunctionDefLibrary library = 2;
}
```



|   成员   |           含义            |
| :------: | :-----------------------: |
|   node   | 表示图节点的 NodeDef 列表 |
| versions |           版本            |
| version  |       版本 (已弃用)       |
| library  |      用户定义的功能       |



#### Ⅱ. NodeDef

***

`NodeDef` 是表示图形节点的数据结构. 

```C++
message NodeDef{
    string name = 1;
    string op = 2;
    repeated string input = 3;
    string device = 4;
    map<string, AttrValue> attr = 5;
}
```



|  成员  |          含义          |
| :----: | :--------------------: |
|  name  | 用于唯一标识节点的名称 |
|   op   |  与节点关联的操作名称  |
| input  |      输入节点列表      |
| device | 用户请求执行操作的设备 |
|  attr  |        属性信息        |



#### Ⅲ. FunctionDefLibrary

***

定义了一组命名函数

```
message FunctionDefLibrary{
    repeated FunctionDef function = 1;
    repeated GradientDef function = 2;
}
```



|   成员   |        含义        |
| :------: | :----------------: |
| function |   一组定义的函数   |
| gradient | 一组定义的梯度函数 |



#### Ⅳ. FunctionDef 

***

TensorFlow 中有一个叫做 function 的函数. 函数是一个允许将多个操作显示为单个节点的 `tensorflow.python.framework.function.Defun()` 函数, 用户可以使用装饰器定义任意函数. 







#### Ⅴ. GradientDef 

***



























