#### LR分析器

##### 介绍

**LR分析器**是一种自底向上（bottom-up）的[上下文无关语法](https://zh.wikipedia.org/wiki/上下文無關語法)分析器。**LR**意指由左（**L**eft）至右处理输入字符串，并以最右边优先派生（**R**ight derivation）的推导顺序（相对于[LL分析器](https://zh.wikipedia.org/wiki/LL剖析器)）建构语法树。能以此方式分析的语法称为**LR语法**。而在**LR(k)**这样的名称中，**k**代表的是分析时所需**前瞻符号**（lookahead symbol）的数量，也就是除了当前处理到的输入符号之外，还得再向右引用几个符号之意；省略 **（k）**时即视为LR(1)，而非LR(0)。

由于LR分析器尝试由分析树的叶节点开始，向上一层层透过文法规则的化简，最后规约回到树的根部（起始符号），所以它是一种自底向上的分析方法。许多[编程语言](https://zh.wikipedia.org/wiki/程式語言)使用LR(1)描述文法，因此许多编译器都使用LR分析器分析源代码的文法结构。LR分析的优点如下：

- 众多的编程语言都可以用某种LR分析器（或其变形）分析文法。（C++是个著名的例外）
- LR分析器可以很有效率的建置。
- 对所有“由左而右”扫描源代码的分析器而言，LR分析器可以在最短的时间内侦测到文法错误（这是指文法无法描述的字符串）。

然而LR分析器很难以人工的方式设计，一般使用“分析产生器（parser generator）”或“编译器的编译器（compiler-compiler，产生编译器的工具）”来建构它。LR分析器可根据分析表（parsing table）的建构方式，分类为“简单LR分析器（SLR, Simple LR parser）”、“前瞻LR分析器（LALR, Look-ahead LR parser）”以及“正统LR分析器 (Canonical LR parser)”。这些解析器都可以处理大量的文法规则，其中LALR分析器较SLR分析器强大，而正统LR分析器又比LALR分析器能处理更多的文法。著名的Yacc即是用来产生LALR分析器的工具。

##### 分析器的结构

以表格为主（table-based）自底向上的分析器可用图一描述其结构，它包含：

- 一个输入[缓冲器](https://zh.wikipedia.org/wiki/緩衝區)，输入的源代码存储于此，分析将由第一个符号开始依序向后扫描。
- 一座[堆栈](https://zh.wikipedia.org/wiki/堆疊)，存储过去的状态与化简中的符号。
- 一张*状态转移表*（goto table），决定状态的移转规则。
- 一张*动作表*（action table），决定当前的状态碰到输入符号时应采取的文法规则，输入符号指的是终端符号（Terminals）与非终端符号（Non-terminals）。

#### 接口扫描器

hotspot提供的接口扫描位于`com.sun.java_cup.internal.runtime.Scanner`，申明一个@next_token方法，需要被扫描器实现，这个方法由LR分析器进行扫描

#### 类符号

```markdown
  解析器需要从词法解析器中获取符号名称，token的标识如下
  sym 符号类型
  parse_state 转换状态
  value 词法分析器的对象类型
  left  原始输入文件的左指针
  right 原始输入文件的右指针
```

| 属性           | 介绍                                                       |
| -------------- | ---------------------------------------------------------- |
| sym            | 标记结束与否的符号名称                                     |
| parse_state    | 转换状态,这个值仅仅解析器可以修改,在转换栈中用于排序关键字 |
| used_by_parser | 是否用于转换                                               |
| left           | 输入文本的左指针                                           |
| right          | 输入文本的右指针                                           |
| value          | 实际值                                                     |

#### 虚拟机解析栈

```markdown
这个类实现了临时的或者虚拟的解析栈。这个解析栈可以对实际解析栈的内容作出替换。同时维护本身的内容.
这个数据接口在转换头决定是否让那个错误恢复请求继续执行的时候使用.一旦转换头成功或者是失败,就会转换
到原始的转换栈上(原始的转换栈没有发生修改).由于转换头没有执行动作,仅仅在虚拟机栈中维护了转换状态.
```

##### 属性

| 属性名称   | 类型  | 介绍                      |
| ---------- | ----- | ------------------------- |
| real_stack | Stack | 实际栈,通常情况下不能修改 |
| real_next  | int   | 实际栈栈指针              |
| vstack     | Stack | 虚拟栈                    |

注意到常规的栈操作（出栈，入栈）都是对虚拟栈进行操作，实际栈不会进行元素直接操作。