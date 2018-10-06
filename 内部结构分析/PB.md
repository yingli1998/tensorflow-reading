# Protocol Buffers 格式数据结构 

TensorFlow 用 Protocol Buffers 定义了很多特别重要的数据格式. 下面将会列出一种重要的数据结构. 



### 数据流图相关的数据结构

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

自定义函数用于封装并复用一组以固定模式组合的操作. 如果图中某个 `NodeDef`对象的 op 值不是 TensorFlow 固有的 `OpDef` 类所定义的操作名称, 而是 `GraphDef` 对象的 `library` 字段所管理的自定义函数名称, 那么当该节点的输入就绪时, 自定义函数会被调用. 自定义函数由 `FunctionDefLibrary` 类管理. 

```
message FunctionDefLibrary{
    repeated FunctionDef function = 1;
    repeated GradientDef function = 2;
}
```



|   成员   |             含义             |
| :------: | :--------------------------: |
| function |      自定义函数的元信息      |
| gradient | 自定义函数对应的梯度计算函数 |



#### Ⅳ. FunctionDef 

***

TensorFlow 中有一个叫做 function 的函数. 函数是一个允许将多个操作显示为单个节点的 `tf.contrib.eager.defun()` 函数, 用户可以使用装饰器定义任意函数. 



`reserved` 是过去存在的字段, 但已经被删除, 但为了兼容性, 仍然保留. 



而且, `node_def` 中的 op 首先认为是用户自定义的, 如果不是, 则假设为是内建的.



```C++
message FunctionDef{
    OpDef signature = 1;
    map<string, AttrValue> attr = 5;
    reserved 2;
    repeated NodeDef node_def = 3;
    map<string, string> ret = 4;
}
```



|   成员    |        含义        |
| :-------: | :----------------: |
| signature | 签名以唯一确定函数 |
|   attr    |      属性信息      |
| node_def  |   构成函数的节点   |
|    ret    |       返回值       |



#### Ⅴ. GradientDef 

***

定义了 `FunctionDef` 相关的梯度函数

```
message GradientDef{
    string function_name = 1;
    string gradient_func = 2;
}
```



























