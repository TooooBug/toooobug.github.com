# 《JavaScript语言精粹》读书笔记 第三章 对象
_2011-04-27 16:47_

### 3.0绪

简单类型：数字、字符串、布尔、null、undefined

对象：数组、函数、正则表达式、对象

对象是属性的窗口，每个属性有名和值。

$$solo_more$$

### 3.1对象字面量

对象字面量是包围在一对花括号中的零或多个名值对。

属性名允许空值。

如果属性名不是保留字，可以不用引号。

对象可嵌套。

### 3.2检索

方括号或者点语法，优先考虑用点语法，因为好看。

不存在的成员元素，返回undefined。

多级对象时，检索undefined会抛出TypeError。用&&解决：flight.equipent && flight.equipment.model

### 3.3更新

赋值语句。

### 3.4引用

对象通过引用来传递，它们永远不会被拷贝！

### 3.5原型

每个对象都连接到一个原型对象，并可以从中继承属性。

所有通过字面量创建的对象都连接到Object.prototype。