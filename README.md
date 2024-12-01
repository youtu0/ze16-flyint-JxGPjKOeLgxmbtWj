
了解 JavaScript 引擎在执行代码过程中所做的一些行为是非常必要的，这有助于我们在遇到莫名其妙的调用时，能够大致定位问题所在。在我学习了预编译的相关知识，并基于[该文章](https://github.com)，引用其中的一段代码，结合“变量提升”、“函数提升”的小示例，对其进行详细的分析，算是留作一份笔记巩固记忆、加深理解。


### 代码



```
console.log(a)
fn1(1)
var a = 123
console.log(a)

var fn1 = () => {
  console.log(a)
}

function fn1(a) {
  console.log(a)
  var a = 666
  console.log(a)
  function a() {}
  console.log(a)
  var b = function () {}
  console.log(b)
  function c() {}
}

fn1(1)

```

错误的推导会让你认为上述代码的打印如下：



```
如果你判断首行报错，那么需要了解变量提升
或者你这样认为
undefined
undefined
666
[Function: a]
[Function: b]
123
undefined
666
[Function: a]
[Function: b]

```

实际上，上方的代码打印如下：



```
undefined
[Function: a]
666
666
[Function: b]
123
123

```

### 详细分析


#### 1\. 创建全局对象 GO


在全局执行上下文中，创建全局对象 `GO`。


#### 2\. 加载当前 JS 文件


加载并解析当前的 JavaScript 文件。


#### 3\. 脚本语法分析


进行语法分析，确保代码没有语法错误。


#### 4\. 当前 JS 文件预编译


##### 4\-1\. 查找变量声明



```
GO = {
  a: undefined
}

```

##### 4\-2\. 查找函数声明（除了函数表达式）



```
GO = {
  a: undefined,
  fn1: function fn1(a) {}
}

```

#### 5\. 正常执行（执行到函数调用前）



```
console.log(a) // 打印 undefined
fn1(1) // 执行到这里了，小心，函数也有预编译，执行前一刻完成

```

#### 6\. 函数预编译


##### 6\-1\. 创建活跃对象 AO



```
AO = {}

```

##### 6\-2\. 查找变量和形参



```
AO = {
  a: undefined,
  b: undefined
}

```

##### 6\-3\. 实参值和形参统一



```
AO = {
  a: 1,
  b: undefined
}

```

##### 6\-4\. 查找函数（非函数表达式）



```
AO = {
  a: function a() {},
  b: undefined,
  c: function c() {}
}

```

#### 7\. 正常执行函数（根据 AO）



```
console.log(a)  // 打印 function a() {}
var a = 666  // a 改变，AO.a = 666
console.log(a)  // 打印 666
function a() {}  // 该声明已提升过，不会覆盖
console.log(a)  // 打印 666
var b = function () {}  // b 改变，AO.b = function () {}
console.log(b)  // 打印 function () {}
function c() {}  // 该声明已提升过，不会覆盖

```

#### 8\. 接着执行函数外代码，执行到下个函数调用前



```
fn1(1) // 已讲述，上续
var a = 123  // GO 对象中的 a 改变为 123（undefined > 123）
console.log(a)  // 打印 123

var fn1 = () => {  // fn1 改变，GO.fn1 = () => {...}
  console.log(a)
}

function fn1(a) {  // 该声明已提升过（函数提升），不会覆盖
  ...
}

fn1(1)  // 执行到这里时，预编译

```

#### 9\. 函数预编译


##### 9\-1\. 创建活跃对象 AO



```
AO = {}

```

##### 9\-2\. 查找变量和形参



```
AO = {
  a: undefined
}

```

##### 9\-3\. 实参值和形参统一



```
AO = {
  a: 1
}

```

##### 9\-4\. 查找函数（非函数表达式）



```
AO = {
  a: 1
}

```

#### 10\. 正常执行函数（根据 AO）



```
console.log(a)  // a 不存在当前函数作用域，往上级查找，找到 GO.a，打印 123

```

### 总结


* **全局预编译**：创建 GO 对象，查找变量声明和函数声明。
* **函数预编译**：创建 AO 对象，查找变量和形参，实参值和形参统一，查找函数声明。
* **执行阶段**：按照代码顺序执行，变量赋值和函数调用。


 本博客参考[悠兔机场](https://xinnongbo.com)。转载请注明出处！
