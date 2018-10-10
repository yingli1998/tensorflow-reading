# Protocol Buffers 格式数据结构 

TensorFlow 用 Protocol Buffers 定义了很多特别重要的数据格式. 下面将会列出一些关键的数据结构. 



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



`attr` 以键值对方式记录与特定操作相关的属性集合. 其键类型是 string, 其取值为属性在 `OpDef` 中对应的 `AttrDef` 对象的 name 值.  



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



#### Ⅵ. AttrValue 

---

```C++
message AttrValue {
  message ListValue {
    repeated bytes s = 2;                        
    repeated int64 i = 3 [packed = true];        
    repeated float f = 4 [packed = true];        
    repeated bool b = 5 [packed = true];        
    repeated DataType type = 6 [packed = true];  
    repeated TensorShapeProto shape = 7;         
    repeated TensorProto tensor = 8;             
    repeated NameAttrList func = 9;              
  }

  oneof value {
    bytes s = 2;                 
    int64 i = 3;                 
    float f = 4;                 
    bool b = 5;                  
    DataType type = 6;           
    TensorShapeProto shape = 7;  
    TensorProto tensor = 8;      
    ListValue list = 1;          
    NameAttrList func = 10;
    string placeholder = 9;
  }
}
```



这是一个包含值的数据结构, 通过 `oneof` 型字段(类似 C/C++ 的 union) 支持多种不同类型的属性值. 



`list` 用于存储用户定义的 `function` , `func.attr.first` 是函数 `attr`  的名称, `func.attr.second` 是那个 `attr` 的初始化值. 



`placeholder` 用于定义在 `function` 中的节点, 指向当 `function` 被实例化的时候, 需要提供的值的属性. 



#### Ⅶ. OpDef

---

表示操作的数据结构, 该类的对象表达的是一类特定操作的公有特征, 而非一幅流图上的某个具体操作节点. 



```
message OpDef {
  // Op names starting with an underscore are reserved for internal use.
  // Names should be CamelCase and match the regexp "[A-Z][a-zA-Z0-9_]*".
  string name = 1;

  // For describing inputs and outputs.
  message ArgDef {
    // Name for the input/output.  Should match the regexp "[a-z][a-z0-9_]*".
    string name = 1;

    // Human readable description.
    string description = 2;

    // Describes the type of one or more tensors that are accepted/produced
    // by this input/output arg.  The only legal combinations are:
    // * For a single tensor: either the "type" field is set or the
    //   "type_attr" field is set to the name of an attr with type "type".
    // * For a sequence of tensors with the same type: the "number_attr"
    //   field will be set to the name of an attr with type "int", and
    //   either the "type" or "type_attr" field will be set as for
    //   single tensors.
    // * For a sequence of tensors, the "type_list_attr" field will be set
    //   to the name of an attr with type "list(type)".
    DataType type = 3;
    string type_attr = 4;    // if specified, attr must have type "type"
    string number_attr = 5;  // if specified, attr must have type "int"
    // If specified, attr must have type "list(type)", and none of
    // type, type_attr, and number_attr may be specified.
    string type_list_attr = 6;

    // For inputs: if true, the inputs are required to be refs.
    //   By default, inputs can be either refs or non-refs.
    // For outputs: if true, outputs are refs, otherwise they are not.
    bool is_ref = 16;
  };

  // Description of the input(s).
  repeated ArgDef input_arg = 2;

  // Description of the output(s).
  repeated ArgDef output_arg = 3;

  // Description of the graph-construction-time configuration of this
  // Op.  That is to say, this describes the attr fields that will
  // be specified in the NodeDef.
  message AttrDef {
    // A descriptive name for the argument.  May be used, e.g. by the
    // Python client, as a keyword argument name, and so should match
    // the regexp "[a-z][a-z0-9_]+".
    string name = 1;

    // One of the type names from attr_value.proto ("string", "list(string)",
    // "int", etc.).
    string type = 2;

    // A reasonable default for this attribute if the user does not supply
    // a value.  If not specified, the user must supply a value.
    AttrValue default_value = 3;

    // Human-readable description.
    string description = 4;

    // TODO(josh11b): bool is_optional?

    // --- Constraints ---
    // These constraints are only in effect if specified.  Default is no
    // constraints.

    // For type == "int", this is a minimum value.  For "list(___)"
    // types, this is the minimum length.
    bool has_minimum = 5;
    int64 minimum = 6;

    // The set of allowed values.  Has type that is the "list" version
    // of the "type" field above (uses the "list" field of AttrValue).
    // If type == "type" or "list(type)" above, then the "type" field
    // of "allowed_values.list" has the set of allowed DataTypes.
    // If type == "string" or "list(string)", then the "s" field of
    // "allowed_values.list" has the set of allowed strings.
    AttrValue allowed_values = 7;
  }
  repeated AttrDef attr = 4;

  // Optional deprecation based on GraphDef versions.
  OpDeprecation deprecation = 8;

  // One-line human-readable description of what the Op does.
  string summary = 5;

  // Additional, longer human-readable description of what the Op does.
  string description = 6;

  // -------------------------------------------------------------------------
  // Which optimizations this operation can participate in.

  // True if the operation is commutative ("op(a,b) == op(b,a)" for all inputs)
  bool is_commutative = 18;

  // If is_aggregate is true, then this operation accepts N >= 2
  // inputs and produces 1 output all of the same type.  Should be
  // associative and commutative, and produce output with the same
  // shape as the input.  The optimizer may replace an aggregate op
  // taking input from multiple devices with a tree of aggregate ops
  // that aggregate locally within each device (and possibly within
  // groups of nearby devices) before communicating.
  // TODO(josh11b): Implement that optimization.
  bool is_aggregate = 16;  // for things like add

  // Other optimizations go here, like
  //   can_alias_input, rewrite_when_output_unused, partitioning_strategy, etc.

  // -------------------------------------------------------------------------
  // Optimization constraints.

  // Ops are marked as stateful if their behavior depends on some state beyond
  // their input tensors (e.g. variable reading op) or if they have
  // a side-effect (e.g. printing or asserting ops). Equivalently, stateless ops
  // must always produce the same output for the same input and have
  // no side-effects.
  //
  // By default Ops may be moved between devices.  Stateful ops should
  // either not be moved, or should only be moved if that state can also
  // be moved (e.g. via some sort of save / restore).
  // Stateful ops are guaranteed to never be optimized away by Common
  // Subexpression Elimination (CSE).
  bool is_stateful = 17;  // for things like variables, queue

  // -------------------------------------------------------------------------
  // Non-standard options.

  // By default, all inputs to an Op must be initialized Tensors.  Ops
  // that may initialize tensors for the first time should set this
  // field to true, to allow the Op to take an uninitialized Tensor as
  // input.
  bool allows_uninitialized_input = 19;  // for Assign, etc.
};
```



主要成员如下:

|  成员  |    含义    |
| :----: | :--------: |
|  name  | 操作的名称 |
| ArgDef |            |
|        |            |
|        |            |





































