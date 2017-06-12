# `LLDB`调试技巧

从`XCode5`之后，所有`XCode`的项目，[调试器自动配置为`lldb`(LLDB Quick Start Guide)](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/Introduction.html)，替代了原来的`GDB`。`LLDB`本身作为一个功能完整的调试器，既可以在通过`XCode`的`GUI`调试模块来工作，也可以单独工作。

如果能够了解`LLDB`本身的常用命令，那么在`XCode`的环境中使用它或者在`XCode`的`debug console`中使用命令都会更加得心应手。

## `LLDB`命令基本语法

如果有`GDB`的经验的话，会发现`LLDB`的命令很相似。所有的`LLDB`命令都是以下的结构：

```cmd
<command> [<subcommand> [<subcommand>...]] <action> [-options [option-value]] [argument [argument...]]
```
具体示例：

```cmd
(lldb) breakpoint set --file test.c --line 12
```

- `<command>`即上图中`breakpoint`部分。使用`help`命令可以看到`LLDB`定义的基本命令集。
- `subcommand`通常是由动词构成，它表示一个`command`内的一类相关操作。比如上例中的`command`，也就是`breakpoint`还有`delete`、`disable`等`subcommand`。使用`help <command>`就能查看相关`command`的`subcommand`。
- `argument`一个`command`或者`command [subcommand]`组合可能需要跟不同参数（`argument`）。针对不同的`option`还可以跟不同的参数。比如上例中`test.c`和`12`就是跟在`breakpoint set`不同的`option`之后的参数。
- `option`用来修饰需要进行的操作。常常在其起前面用`--`分割，一些`option`也提供简约表达形式使用`-`开头。比如上例中，`--file`这个`option`的简约表达式为`-f`。

#### 为命令设置别名（`command alias`）

如果觉得命令的输入十分冗长、不方便，可以为常用的命令设置便于记住的别名。这样就方便使用，而不用每次输入长段的命令。比如：

```cmd
(lldb) command alias bfl breakpoint set -f %1 -l %2
```

将在某个文件（-f 之后的参数）某一行（-l 之后的参数）设置一个断点的命令用别名`bfl`代替。在使用：

```cmd
(lldb) bfl foo.c 12
```

实际上就是在执行：

```cmd
(lldb) breakpoint set --file foo.c --line 12
```

常见的一些如`next`、`step`和`continue`都是`LLDB`初始配置的命令别名。想要查看所有的命令，包括自定义的命令别名，在现版本的`LLDB`（lldb-370.0.37）中，只需要键入`help`命令：

```
...  /* 命令 */
Current command abbreviations (type 'help command alias' for more info):
... /* 命令别名 */
```

第二段就列出了所有的命令别名。

如果想要去掉自己设置的或者是`LLDB`预设的命令别名，只需要执行下列命令就可以了：

```cmd
(lldb) command unalias <command-alias> 
```

#### 使用`LLDB`的`help`功能

- 使用`help help`查看`help`命令的使用；
- 使用`help <command>`查看某个`command`的使用；
- 使用`help <command> [subcommand]`查看某个`command`下的子命令的使用；

#### 命令不同形式

以在`XCode`中最常用的`po`命令来举例。`LLDB`的命令是存在多种形式的，实际在上面已经零散提到。

|命令形式|命令内容|
|:-----|:-----:|
|标准形式|expression --object-description -- someVariable|
|简写形式|e -O -- someVariable|
|别名形式|po someVariable|

需要解释一下的是这里`--object-description`这个`option`之后用`--`分割了`argument`部分。[文档](https://developer.apple.com/library/content/documentation/General/Conceptual/lldb-guide/chapters/Introduction.html)解释：

>  Commands that accept options as well as freeform arguments, such as the expression command, must place a space-delimited double dash (--) between the last option and the first argument. This ensures that arguments resembling an option by starting with a dash (-) are interpreted as an argument.

## 独立使用`LLDB`流程

绝大部分时候，都是通过`XCode`间接地使用`LLDB`的调试功能。理解如何独立使用`LLDB`调试程序的流程，也能更好地理解在`XCode`中调试程序的技巧。

大体上，使用`LLDB`可以分为下面几个步骤（不一定具有先后顺序）：

- 确定调试的程序
- 设置调试断点
- 

## 参考资料

- [LLDB Quick Start Guide](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/Introduction.html)
- [LLDB Debugging Guide](https://developer.apple.com/library/content/documentation/General/Conceptual/lldb-guide/chapters/Introduction.html)
- [Advanced Debugging with LLDB - WWDC]()
- [The LLDB Debugger]()
