# 前言

第一单元的主题是表达式展开。学习目标聚焦于模块化设计。
## 最终UML类图
![最终UML类图](https://img-blog.csdnimg.cn/direct/5989a78c37994353bfc2540297a1bca5.png)

## 最终UML依赖图
![最终UML依赖图](https://img-blog.csdnimg.cn/direct/99064a9444d143fdae08f666942493ef.png)
## 最终代码规模图
![类总规模](https://img-blog.csdnimg.cn/direct/cc16ac4b614747fcb1630cabab77f783.png)
如图，三次迭代后最终程序有 
- 17 个`.java`文件
- 代码总长 823 行
- 文件中的最小行数为12（表达式因子类）
- 文件中的最大行数为197（最小单元类，因为优化输出长度进行了大量`if-else`判断 QAQ）
- 文件的平均行数为48

---

下面分享一下每次作业的设计思路~
# 一、第一次作业分析
第一次作业主要是关于单变量多项式的括号展开

## 1.代码UML类图
![简单包含继承关系](https://img-blog.csdnimg.cn/direct/e052e29d3a444dd896c815c3cc4aab61.png)

## 2.代码架构分析
仔细阅读教程，经过一定的思考分析，我们将表达式解析为三层——**表达式**-`Expr`，**项**-`Term`，**因子**-`Factor`。其中根据需求，将`Factor`分为三种——**变量因子（幂函数）**-`PowerFactor`，**常数因子**-`NumFactor`，**表达式因子**-`ExprFactor`。这里考虑到各种因子可能拥有各自的幂指数`index`，运用了继承。其中`Expr`类中使用`Arrylist`来容纳表达式中含有的项，`Term`类中使用`Arrylist`来容纳项中含有的因子。
### a、表达式预处理
我们设置了表达式处理类`Processor`，预处理包括
i)输入的表达式中**空白字符**删去
ii)将连续的`+-`号合并为一个`+`或`-`
### b、表达式解析
对第一次作业的表达式解析至少有两种方案，一种是利用**正则表达式**，另一组是利用课程组传授的**递归下降算法**。我们采用了老师和助教推荐的后者。
**递归下降算法**的详细思想可见**OO加油站**。主要包含两个部分——**词法分析器**-`Lexer`和**解析器**-`Paser`。
#### Lexer
将表达式分解为一系列语法单元（`Token`类）
第一次作业的基本语法单元有：加`ADD`, `SUB`, `MUL`, 乘方-`POWER`, `LPAREN`, `RPAREN`, `VAR`,`NUM`。我们考虑用枚举类型来记录。
可以直接用**循环**对字符串进行遍历，将解析出的**语法单元**放入`tokens`中。
#### Paser
依靠`Lexer`分解的`tokens`递归生成表达式、项和因子
我们将表达式的解析分为三层——`parseExpr`，`parseTerm`，`parseFactor`，每一部分的解析都遵循形式化文法。
`parseExpr`:考虑到第一项可能带有符号，进行特殊处理。解析完第一项后直接用循环进行`ADD`or`SUB` 与 `parseTerm`的解析（这里解析项实际上就是调用一次`parseTerm`）。
`parseTerm`:同理。`ADD`or`SUB` 变为 `MUL`
`parseFactor:`对三种不同因子进行**条件判断**并解析。
### c、表达式展开
单变量表达式的展开实际上是关于`x`的多项式。我们考虑引入**多项式类**-`Poly`和**单项式类**-`Mono`。
`Mono`:含有两个成员变量——**系数**-`coe`和**指数**-`index`。这里我们都采用了`BigInteger`（似乎`index`并不需要）。
`Poly`:有容纳一系列单项式的容器和各种多项式的运算方法（如多项式的加、乘、乘方），以及形成字符串的方法`toString()`。
自然地，我们对`Expr`，`Term`，`Factor`类中都引入**转换为表达式**-`toPoly(...)`方法，分别利用表达式的运算进行转换（这里要考虑因子或项的符号问题，可以把符号下发给因子或者集合到项中）。
## 3.优化
- 单项式的系数为`0`，结果为`0`
- 单项式的系数为`1`或`-1`，可以省略系数
- 单项式的指数为`0`，只输出系数
- 单项式的指数为`1`，可以省略指数
- 找到多项式为正的那一项并移到首位（否则多一个负号）
## 4.代码复杂度分析
**Method Metrics**

![Method Metrics](https://img-blog.csdnimg.cn/direct/26da6d36fcbb4f2a998715934ddb5d83.png)
- `Lexer`类中的构造方法使用了大量`if-else`语句对表达式语法单元进行解析
- `Lexer`类中的`toString()`方法使用了大量`if-else`语句特判进行优化
-  `Parser`类中的`parseFactor()`方法先对因子进行了判断，再解析
- `Poly`类中的`toString()`方法进行了输出优化
# 二、第二次作业分析
第二次作业在第一次作业的基础上新增了**指数函数因子**和**自定义函数因子**，允许**括号嵌套**，且自定义函数调用时**实参**可以使用**自己或其他的函数**。
## 1.代码UML类图
![UML类图](https://img-blog.csdnimg.cn/direct/5c9fd48801704f12b5e5d7a9ee4ffc02.png)
## 2.代码架构分析
第二次作业我们延续**递归下降算法**，自然实现了**括号嵌套**。需要增加新的语法单元（如`EXP`，`FUNC`，`COMMA`(逗号)）
### a.指数函数解析
我们新建`ExpFactor`类，该类拥有两个属性——`Factor base`-表示指数函数括号内的因子，以及继承父类的`index`。
同时，我们在`Parser`类中新增`paresExpFactor()`方法。当`parseFactor()`中解析到当前因子为`exp`时，调用此方法解析指数函数。
```java
//Paser.java
public Factor paresExpFactor() {
        //向后解析
        ExpFactor expFactor = new ExpFactor(parseFactor());
        //向后解析
        return setIndex(expFactor); //确定幂指数index
    }
```
### b.自定义函数的解析
我们新建`FuncFactor`类，该类拥有三个属性——`String actualFunc`-自定义函数实参替换形参后的结果，`Expr base`-解析成表达式后的结果，以及继承父类的`index`。
此外，我们可以新建一个**工具类**来实现自定义函数的**定义**和**解析**。
#### 工具类
- 该类具有两个属性
```java
//工具类
private static final HashMap<String, String> funcMap = new HashMap<>(); //函数名-函数定义式
private static final HashMap<String, ArrayList<String>> paraMap = new HashMap<>(); //函数名-函数形参表
```
- 具有两个方法
```java
//工具类
public static void addFunc(String input) {...}; //将函数定义式和形参加入属性
public static String formFunc(String funcName, ArrayList<Factor> actualParas) {...}; //形参替换成实参
```
我们还是在`Parser`类处理自定义函数（也有的方法是在**预处理**时就将形参替换为实参）。`Parser`类中新增`paresFuncFactor()`方法。当`parseFactor()`中解析到当前因子为自定义函数时，调用此方法解析指数函数。
```java
//Parser.java
public Factor parseFuncFactor(String funcName) {
        //...
        ArrayList<Factor> actualParas = new ArrayList<>();
        while (lexer.now().getType() != Token.Type.RPAREN) {
            //跳过左括号、逗号
            actualParas.add(parseFactor());
        }
        //...
        return new FuncFactor(funcName, actualParas);
    }
```
我们先把函数名和参数解析出来，然后在`FuncFactor`类中调用`Parser`类的方法完成解析
```java
//FuncFactor.java
public Expr formExpr() {
        Lexer lexer = new Lexer(actualFunc);
        Parser parser = new Parser(lexer);
        return parser.parseExpr();
    }
```
> tips: `常数因子`可能带有符号，解析时要注意~
### c.表达式展开
第二次作业中多项式的**最小单元**不再是单项式，我们考虑将之前的`Mono`类重构为`Unit`类（IDEA重构功能十分好用）。形如：
$$ax^n(e^{<expr>})^b$$
利用此，我们可以构造`Unit`类的属性。
#### 合并同类项
我们需要重写之前多项式中的**加**、**乘**、**乘方**方法。其中我们需要实现**合并同类项**的功能。
对于`Hashmap`的`value`-`Unit`我们可以以`Unit`为`key`；也可以以`x`的幂指数为`key`，以另一个`Hashmap`为`value`，后者以指数函数括号内的表达式为`key`（这里可以是表达式的字符串形式（需要一定的次序）），最终`value`-`Unit`。
接着我们改写`equals()`和`hashCode()`方法（这两个方法一定要配套改写，保证相等时具有相同的hash值。hash也决定了hash表的查找性能）
## 3.代码复杂度分析
![复杂度分析](https://img-blog.csdnimg.cn/direct/be8f21c191b047b0a115173166735ab8.png)
- `Lexer`类中的构造方法使用了大量`if-else`语句对表达式语法单元进行解析
- `Poly`类中的`toString()`方法进行了输出优化
# 三、第三次作业分析
第三次作业在之前的基础上增加了**求导因子**，且函数表达式中支持调用其他**已定义的函数**。
## 1.代码UML类图
![1.3UML](https://img-blog.csdnimg.cn/direct/e63bfdee08974826a44c2ca739654fd8.png)
## 2.代码架构分析
第三次作业我们延续**递归下降**算法和第二次作业中对自定义函数的处理，自然实现了自定义函数表达式中调用其他函数的功能。需要增加新的因子-**求导因子**
### a.求导因子解析
我们新建`DxFactor`类，该类拥有两个属性——`Expr base`-表示需要求导的表达式，以及继承父类的`index`。
同时，我们在`Parser`类中新增`paresDxFactor()`方法。当`parseFactor()`中解析到当前因子为`dx`时，调用此方法解析指数函数。
### b.求导计算
这里对表达式的求导不限于两种方法
- **先化简，再求导**，即对求导因子内表达式先调用`toPoly()`方法化简为许多 $coe·x^n·e^{expr}$ 和的形式，利用公式化的方法
$$(coe·x^n·e^{expr})'=coe·n·x^{n-1}·e^{expr}+coe·x^n·e^{expr}·(expr)'$$
转化为每一个`Unit`的求导。当然可以分情况讨论（如`coe`, `coe·x^n`, `coe·exp(expr)`直接借助求导公式节约计算量）
- **递归求导**，即在`Expr`,`Term`以及`Factor`的各个子类中都实现一个`toDerivative()`方法，且均返回`Poly`类型的对象。
	- 表达式求导是各个项求导的和
	- 项求导是因子求导的乘积+求导的乘法法则
	- 各个因子分别对应各种求导公式（如常数求导为`0`）。表达式因子的求导再递归回去。需要注意，若因子为**求导因子**，即对表达式求**二阶导**，我们应当先求内层的导数，然后再求外层的导数。这可能涉及`Poly`类的转换为字符串再重走词法解析流程。

我们这里采取了第一种方案。
```java
// Poly.java
public Poly derivatePoly() {
        ArrayList<Unit> unitsDx = new ArrayList<>();
        for (Unit unit : units) {
            // ...
        }
        return new Poly(unitsDx);
    }
```
```java
// Unit.java
public Poly derivateUnit() {
        ArrayList<Unit> units = new ArrayList<>();
        if (coe.equals(BigInteger.ZERO)
                || (index.equals(BigInteger.ZERO) && expBase.getUnits().isEmpty())) {
            // ...
        }
        if (expBase.getUnits().isEmpty()) {
            // ...
        }
        if (index.equals(BigInteger.ZERO)) {
            // ...
        }
        // ...
    }
```
## 3.优化
这里主要讨论**提取公因数**以减小长度的问题
因为指数函数 $e^x$ 具有独特的数学性质：$$(e^x)^a = e^{ax}$$
而正确性判定中并没有对该处加以限制，所以我们在本次作业**输出结果的有效长度**中可能且不仅限于考虑：当指数函数`exp(<因子>)`中的因子是表达式因子时是否需要**找到系数中合适的公因子并提出**，转换为`exp(<因子>)^a`的形式。
稍微思考后我们发现，很难有一个统一的方案去处理这里的性能。这里我们采取**提取最大公因数**的方式。即借用`BigInteger`自带的`gcd`方法来寻找最大公因数并提取。
## 4.代码复杂度分析
![复杂度分析](https://img-blog.csdnimg.cn/direct/a58dd96ee9b541d49c7c733303433dad.png)
- `Lexer`类中的构造方法使用了大量`if-else`语句对表达式语法单元进行解析
- `Parser`类中的`parseFactor`方法使用了大量`if-else`语句区分不同的表达式因子
- 优化输出长度时需要大量判断

---
# 总结
## 新的迭代？
参考了外界学长学姐的博客，本届第一单元作业更加人性化些。如果在之前程序的基础上继续进行迭代，加入**求和函数**、**三角函数**等因子，可以延续之前的做法在`Factor`类后进行继承，`Parser`类加入解析方法，思想是一致的。难点可能聚焦于**优化**。而这里的优化目前思路就是不断的`if-else`判断，而这样会增加程序**复杂度**。希望能探索更好的方法~

## BUG小结
三次作业很幸运通过都通过了强测、互测。这很大程度上归功于**好的架构**和**评测机**的检测。
- 第二次作业曾遇到解析**常数因子**未解析符号。第一次作业因为把符号提出去了，没有影响。第二次作业常数因子作为**实参**就不能避免这个问题了，加个符号判断就好
- 第三次作业在**优化-提取公因数**时因为**深浅拷贝**出了 bug
## 浅谈HACK
**互测**给予我们搭建评测机的动力，让我们有机会读取别人代码参考他们的架构，同时让别人帮我们找 bug（虽然会失分）。希望能在互测中成长~
和别人交流能收获许多好的测试点，这里浅浅列举几个~
```c
(((((((((((x^8)^8)^8)^8)^8)^8)^8)^8)^8)^8)^8)^8	// 一个来自强测的程序性能测试

dx(exp(exp(exp(exp(exp(exp(exp(exp(x^2))))))))) // 程序性能测试

exp((-x)) // 正确性测试——不能去括号~

3
g(z)=exp(exp(exp(exp(z))))
f(y)=exp(exp(exp(exp(g(y)))))
h(x)=exp(exp(exp(exp(f(x)))))
h(f(g(exp(exp(exp(exp(x^8)))))))
// emm~
```
## 心得体会
第一单元作业将**递归**这一思想的魅力体现得淋漓尽致，掌握好的设计能力，体会`java`编程的原理，看着如此复杂的表达式被一点点拆分处理，最终合并化简，觉得蛮奇妙呢~
