---
title: 【翻译】LPeg编程指南
tags: [lua,lpeg]
---

原文：[http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#ex)

#### 译者序：

这个是官方的LPeg的文档。这段时间学习LPeg的时候发现国内关于LPeg的文章很少，所以决定把文档翻译一下。

翻译的不是很完整，只是常用的一部分，会慢慢的翻译下去，有同学能帮我补全的话就太感谢了。

#### 介绍：

LPeg是lua中一个新的模式匹配（pattern-matching）的库，基于 [Parsing Expression Grammars](http://pdos.csail.mit.edu/~baford/packrat/) (PEGs)。本文是一个关于LPeg库的参考手册。关于更详细的文档，请看see [A Text Pattern-Matching Tool based on Parsing Expression Grammars](http://www.inf.puc-rio.br/~roberto/docs/peg.pdf).，这里有关于实现的更详细的讨论。

根据 Snobol的传统，LPeg定义patterns作为第一级别对象，也就是说 patterns 可以作为常规的lua变量(represented by userdata)。

这个库提供了多种方式来创建和组合patterns。通过使用元方法，个别的一些函数可以提供类似中缀运算符或前缀运算符。一方面，相对于一般的正则表达式，LPeg匹配的结果通常更为详细。另一方面，第一级别的patterns可以更好的描写和扩展正则关系，我们可以定义函数来创建和组合patterns。

| Operator | Description |
| ------------- |:-------------:|
| [`lpeg.P(string)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-p) | 匹配字符串 |
| [`lpeg.P(n)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-p) | 匹配n个字符串 | 
| [`lpeg.S(string)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-s) | 匹配字符串中任意一个字符 (Set) | 
| [`lpeg.R("xy")`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-r) | 匹配x和y之间的任意一个字符(Range) | 
| [`patt^n`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-pow) | `匹配至少n个``patt` | | [`patt^-n`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-pow) | `匹配最多n个``patt` | 
| [`patt1 * patt2`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-mul) | 先匹配`patt1` 然后接着匹配 `patt2` | 
| [`patt1 + patt2`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-add) | 匹配满足`patt1` 或者满足`patt2` (二选一) | 
| [`patt1 - patt2`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-sub) | 匹配满足patt1而且不满足patt2 | 
| [`-patt`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-unm) | 和 `("" - patt)一样` | | [`#patt`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-len) | Matches `patt` but consumes no input | 
| [`lpeg.B(patt)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-behind) | Matches `patt` behind the current position, consuming no input |

举一个很简单的例子， lpeg.R("09")^1创建了一个pattern，这个pattern的作用是匹配一个非空的数字序列。再举一个稍微复杂一点的例子，-lpeg.P(1)匹配一个不能有任何字符的空字符串，这个通常用在匹配规则的最后。

### Functions

#### lpeg.match (pattern, subject [, init])

匹配函数。它试图通过一个给定的pattern来对目标字符串进行匹配。如果匹配成功， 则返回匹配成功子串的第一个字符的位置，或者返回捕获的值（如果成功捕获到值的话）。

一个可选的数字参数 init 作为匹配目标字符串的起始位置。和通常的Lua库一样，如果参数是一个负数，则从目标字符串的最后一个字符开始向前计算，得到起始位置。

和典型的匹配函数不同， match 仅仅在一个固定的模式下工作； 也就是说，它试着从目标字符串的前缀字符开始匹配，而不是匹配任意的子串。.所以，如果我们想匹配一个任意位置的子串，就必须用Lua写一个循环来把目标字符串的每一个位置作为起始位置匹配，或者写一个pattern来匹配任意字符。两种方法对比来说，第二种非常方便、快捷和高效，可以以看看下面的例子。
#### lpeg.type (value)

如果value是一个pattern，则返回一个字符串 "pattern".，否则返回nil。

#### lpeg.version ()

返回LPeg的字符串版本号。

#### lpeg.setmaxstack (max)

设置堆栈的上限，默认是400。

### Basic Constructions

####lpeg.P (value)

用下面的规则将一个给定的值转换成一个合适的pattern：
* 如果参数是一个pattern，则返回参数pattern。 
* 如果参数是一个string，则返回匹配这个字符串的pattern。 
* 如果参数是一个非负整数 _n_, 则返回一个匹配正好是n个字符的字符串的pattern。 
* 如果参数是一个负整数 _-n_, 则只有在输入的字符串还剩下不到n个字符才会成。 lpeg.P(-n) 等同于 -lpeg.P(n) (see the **[unary minus operation](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-unm)**). 
* 如果参数是一个 boolean, the result is a pattern that always succeeds or always fails (according to the boolean value), without consuming any input. 
* 如果参数是一个table, 则被解读为一个grammar (see **[Grammars](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#grammar)**)。 
* 如果参数是一个function, 则返回一个pattern，等价于一个 **[match-time capture](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#matchtime)** 用一个空字符串匹配.

#### lpeg.B(patt)

Returns a pattern that matches only if the input string at the current position is preceded by patt. Pattern patt must match only strings with some fixed length, and it cannot contain captures.
Like the **[and predicate](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#op-len)**, this pattern never consumes any input, independently of success or failure.

#### lpeg.R ({range})

返回一个在给定的范围内任何一个字符。范围是一个长度为2的字符串xy，返回的所有字符都是x和y对应ASCII编码之间（包括x和y）。
 举个例子， pattern lpeg.R("09") 匹配所有的数字，lpeg.R("az", "AZ") 匹配所有的ASCII字母。
 
#### lpeg.S (string)
返回一个pattern匹配一个字符，这个字符是给定的string中的任何一个字符。 (The S stands for _Set_.)

举个例子， pattern lpeg.S("+-*/") 匹配任何一个算术运算符。

注意， 如果s是一个字符，那么 lpeg.P(s) 等价于 lpeg.S(s)。

#### lpeg.V (v)

This operation creates a non-terminal (a _variable_) for a grammar. The created non-terminal refers to the rule indexed by v in the enclosing grammar. (See **[Grammars](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#grammar)** for details.)
**lpeg.locale ([table])**
Returns a table with patterns for matching some character classes according to the current locale. The table has fields named alnum, alpha, cntrl, digit, graph, lower, print, punct, space, upper, and xdigit, each one containing a correspondent pattern. Each pattern matches any single character that belongs to its class.
If called with an argument table, then it creates those fields inside the given table and returns that table.

#### #patt
Returns a pattern that matches only if the input string matches patt, but without consuming any input, independently of success or failure. (This pattern is called an _and predicate_ and it is equivalent to _&patt_ in the original PEG notation.)
This pattern never produces any capture.

#### -patt
返回一个pattern，这个pattern要求输入的字符串不匹配patt。 它不消耗任何的输入，只是成功或者失败。 (This pattern is equivalent to _!patt_ in the original PEG notation.)
举个例子，pattern -lpeg.P(1) 匹配字符串的末尾。

这个pattern 从来不产生任何捕获，因为不是 patt失败就是 -patt 失败。 (一个失败的 pattern 从来不产生任何捕获 )

#### patt1 + patt2

返回一个符合 patt1 或者 patt2的pattern。
如果 patt1 和 patt2 都是字符集合, 则得到的结果是两个的并集。
lower = lpeg.R("az")
upper = lpeg.R("AZ")
letter = lower + upper

#### patt1 - patt2
相当于 _!patt2 patt1_。 这个pattern 意思是不匹配 patt2 且匹配 patt1。
如果成功了，则最后捕获到的是patt1的内容。这个pattern不会从patt2中捕获任何信息 (as either patt2 fails or patt1 - patt2 fails).
如果 patt1 和 patt2 都是字符集合，那么这个运算就相当于集合差。 注意 -patt等价于 "" - patt (or 0 - patt). 如果 patt 是一个字符集合， 1 - patt是它的补集。

#### patt1 * patt2
返回一个pattern，这个pattern先匹配patt1，patt1匹配完成之后，从匹配完成的下一个字符开始匹配patt2。 The identity element for this operation is the pattern lpeg.P(true), which always succeeds.
     (LPeg uses the * operator [instead of the more obvious ..] both because it has the right priority and because in formal languages it is common to use a dot for denoting concatenation.)
     
#### patt^n
如果 n 是一个非负数， 这个pattern等价于 _patt__n__ patt*_。它匹配的条件是至少n个 patt。
另外， 如果n 是负数， 这个 pattern 等价于 _(patt?)__-n_: 它匹配的条件是最多 |n| 个 patt。
在个别情况下， 在原始的 PEG 中 ，patt^0 等价于 _patt*_, patt^1 等价于 _patt+，_ patt^-1 等价于 _patt?_。
在所有的情况下， the resulting pattern is greedy with no backtracking (also called a _possessive_ repetition).注意，patt^n只会匹配最长的序列。

#### Grammar

在lua的环境下，可以自定义一些patterns，让新定义的pattern可以使用已经定义过的旧的pattern，然而，这些技巧不允许定义循环的patterns。 For recursive patterns, we need real grammars.
LPeg通过使用table来定义gramar， table的每个条目是一条规则。

#### Captures

_capture_ 是一个pattern匹配成功之后捕获的值。 LPeg提供多种捕获方式， 基于pattern的匹配和组合来产生不同的捕获值。
下面是捕获的基本概述：

| **Operation** | **What it Produces** |
| ------------- |:-------------:|
| [`lpeg.C(patt)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-c) | 所有pattern捕获的子串 | 
| [`lpeg.Carg(n)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-arg) | the value of the nth extra argument to `lpeg.match` (matches the empty string) | 
| [`lpeg.Cb(name)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-b) | the values produced by the previous group capture named `name` (matches the empty string) | 
| [`lpeg.Cc(values)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-cc) | the given values (matches the empty string) | 
| [`lpeg.Cf(patt, func)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-f) | 捕获的结果将作为参数依次被func调用 | 
| [`lpeg.Cg(patt [, name])`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-g) | 把patt所有的返回值作为一个返回值并指定一个名字 | | [`lpeg.Cp()`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-p) | 捕获的位置 | 
| [`lpeg.Cs(patt)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-s) | 创建一个替代捕获 | 
| [`lpeg.Ct(patt)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-t) | 把patt中所有的返回值按照父子关系放到一个数组里返回 |
| [`patt / string`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-string) | `string`, with some marks replaced by captures of `patt` | 
| [`patt / number`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-num) | the n-th value captured by `patt`, or no value when `number` is zero. | 
| [`patt / table`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-query) | `table[c]`, where `c` is the (first) capture of `patt` | 
| [`patt / function`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-func) | the returns of `function` applied to the captures of `patt` | 
| [`lpeg.Cmt(patt, function)`](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#matchtime) | the returns of `function` applied to the captures of `patt`; the application is done at match time |

#### lpeg.C (patt)
返回匹配到的子字符串以及patt内部子patt的返回值。

#### lpeg.Carg (n)
Creates an _argument capture_. This pattern matches the empty string and produces the value given as the nth extra argument given in the call to lpeg.match.

#### lpeg.Cb (name)
Creates a _back capture_. This pattern matches the empty string and produces the values produced by the _most recent_ **[group capture](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html#cap-g)** named name (where name can be any Lua value).
_Most recent_ means the last _complete_ _outermost_ group capture with the given name. A _Complete_ capture means that the entire pattern corresponding to the capture has matched. An _Outermost_ capture means that the capture is not inside another complete capture.

#### lpeg.Cc ([value, ...])
Creates a _constant capture_. This pattern matches the empty string and produces all given values as its captured values.

#### lpeg.Cf (patt, func)

创建一个折叠的捕获，假设patt有n个返回值,C1,C2,C3,那么Cf返回 f(f( f(C1),C2), C3)。
举个例子，一个用逗号隔开的数字序列，计算出数字串中每个数字相加的结果：
-- matches a numeral and captures its numerical value
number = lpeg.R"09"^1 / tonumber

-- matches a list of numbers, capturing their values
list = number * ("," * number)^0

-- auxiliary function to add two numbers
function add (acc, newvalue) return acc + newvalue end

-- folds the list of numbers adding them
sum = lpeg.Cf(list, add) -- example of use
print(sum:match("10,30,43"))  --> 83

#### lpeg.Cg (patt [, name])
创建一个捕获的集合，这组返回的所有值型成一个捕获。集合可能是匿名(如果没有名字)或命名的(可以是任何非nil值Lua值)。

#### lpeg.Cp ()
Creates a _position capture_. It matches the empty string and captures the position in the subject where the match occurs. The captured value is a number.

#### lpeg.Cs (patt)

Creates a _substitution capture_, which captures the substring of the subject that matches patt, with _substitutions_. For any capture inside patt with a value, the substring that matched the capture is replaced by the capture value (which should be a string). The final captured value is the string resulting from all replacements.

#### lpeg.Ct (patt)
创建一个捕获的数组。  创建一个表捕获;这个捕获将创建一个表,将匿名的捕获保存到表中,索引从1开始.对于命名组捕获,以组名为key。
**注：下面的内容由于开源中国已经翻译完成故不再翻译**[http://www.oschina.net/translate/lpeg-syntax](http://www.oschina.net/translate/lpeg-syntax)