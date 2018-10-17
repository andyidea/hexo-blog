---
title: lua和c的亲密接触
date: 2017-02-11 14:40:10
tags: [lua,c]
---

## 介绍

>lua和c的亲密接触，靠的是一个虚拟栈。lua通过这个虚拟栈来实现和c之间值的互传。栈上的每一个元素是一个lua值（nil，number，string...）。

当lua调用c函数的时候，这个函数会得到一个新的栈，这个栈独立于c函数本身的栈，也独立于lua自己的栈。它里面包含了lua要传给c的所有参数，然后c函数会把返回的结果放入这个栈中返回给调用者。
对于栈的查询操作，如果按照栈的规则，只能拿到栈顶的元素。但这里和常规的栈有一些差异。就是可以用一个索引来指向栈上的任何元素。正数的索引`（1...n）`指向从栈底到栈顶元素，`1`就是最先入栈的元素，`n`就是栈顶的元素，负数的索引`（-1...-n）`指向从栈顶到栈底的元素，`-1`就是栈顶元素，`-n`就是最先入栈的元素。通过这两种索引方式可以很方便的获取栈中的元素。

## 基本操作

lua和c之间的交互的桥梁是一个虚拟栈，这个虚拟栈在lua的c api中为`lua_State`，下面的代码展示了从创建栈，元素入栈，根据索引获取栈中元素的值的过程，这也是`lua_State`的最基本的操作。

```c
lua_State *L = luaL_newstate();//创建一个新的栈

lua_pushstring(L, "muzixiaoxin"); //把一个字符串压入栈
lua_pushnumber(L, 875);//把一个整型压入栈

//现在栈内有两个元素，栈底是字符串"muzixiaoxin"，栈顶是整型875
//"muzixiaoxin"的索引就是1，或者-2
//855的索引就是2，或者-1

if (lua_isstring(L, 1)){//判断栈底的元素是不是字符串
    printf("%s\n",lua_tostring(L, 1));//如果是字符串就转换成字符串输出
}

if (lua_isnumber(L, -1)){//判断栈顶元素是不是number类型
    printf("%d", lua_tonumber(L, 2));//如果是就转换成number类型输出
}

lua_close(L); //记得不需要的时候要释放掉
```

>提示：更多的相关函数请参考[lua中文手册](http://cloudwu.github.io/lua53doc/manual.html)

## c调用lua

c调用lua这种情况我见到的比较少，一般都是用lua虚拟机直接跑脚本。也有一些把lua作为配置文件给c用的。
举个例子，新建一个lua文件test.lua

```lua
name = "muzixiaoxin"
version = 1003
```

c需要通过lua c api把这个文件加载进来，然后执行，执行的结果存在一个栈中， 去这个栈中拿到变量的值。
看下面的c代码：

```c
lua_State *L = luaL_newstate();

int err = luaL_loadfile(L, "test.lua"); //把lua文件加载成代码块，只加载不运行
if (err){
    return;
}

err = lua_pcall(L, 0, 0, 0);//运行加载的代码块
if (err){
    return;
}

lua_getglobal(L, "name"); //把全局变量name的值压入栈顶
printf("%s\n", lua_tostring(L, -1));//取出栈顶元素打印结果为:muzixiaoxin

lua_close(L); //记得不需要的时候要释放掉
```

## lua调用c方法

lua调用c有些麻烦，要写一个固定格式的方法来供lua调用。
我们先简单的写个求和的c方法：

```c
//计算求和的方法
static int
sum(int a, int b){
    return a + b;
}
```

这个方法就是求两个整型的和。我们要让lua使用这个方法，就要先把这个方法注册给lua的状态机，但注册给lua状态机的方法要求有固定的参数和固定的返回值，参数要是一个`lua虚拟栈`，这个虚拟栈存放着lua传过来的参数，lua调用的返回值也要通过这个虚拟栈返回给lua，最后的返回值要求是一个`int值`，存着返回给lua变量的个数。我们看写好的方法：

```c
//lua调用的方法
static int
lsum(lua_State *L){
    int a = (int)lua_tonumber(L, -1);//lua调用的参数之一
    int b = (int)lua_tonumber(L, -2);//lua调用的参数之一
    lua_pushnumber(L, sum(a, b));//把计算的加过压栈
    return 1;//返回返回值的个数
}
下一步是吧lsum这个方法注册给lua状态机：
lua_State *L = luaL_newstate();

luaL_openlibs(L);//打开L中的所有标准库，这样就可以使用print方法

lua_register(L, "sum", lsum);//把c函数lsum注册为lua的一个全局变量sum

int err = luaL_dofile(L, "test.lua"); //把lua文件加载成代码块，并运行
if (err){
    return;
}

lua_close(L);
```

test.lua的内容是：

```lua
print("1 + 2 = " .. sum(1,2))
```

最后的输出结果：
![image](http://images2015.cnblogs.com/blog/229056/201601/229056-20160115173124225-858704453.png)

总结一下，就是，你要通过一个中间函数（像lsum这种）对lua虚拟栈进行操作来实现lua调用c的方法。

>提示：更多的lua c api请参考[lua中文手册](http://cloudwu.github.io/lua53doc/manual.html)
