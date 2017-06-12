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

大体上，使用`LLDB`可以分为下面几个步骤（大致有先后顺序）：

- 确定调试的程序
- 设置调试断点
- 控制程序运行
- 输出线程和`stack frame`信息。

在这里将[文档](https://developer.apple.com/library/content/documentation/General/Conceptual/lldb-guide/chapters/C1-Introduction-to-Debugging.html#//apple_ref/doc/uid/TP40016717-CH6-SW1)实例照搬至此。

假设有一下的一个名为`Greeter.swift`的文件：

```swift
class Greeter {
    private var acquaintances: Set<String> = []

    func hasMet(personNamed name: String) -> Bool {
	return acquaintances.contains(name)
    }

    func greet(personNamed name: String) {
	if hasMet(personNamed: name) {
	    print("Hello again, \(name)!")
	} else {
	    acquaintances.insert(name)
	    print("Hello, \(name). Nice to meet you!")
	}
    }
}

let greeter = Greeter() 
greeter.greet(personNamed: "Anton")
greeter.greet(personNamed: "Mei")
greeter.greet(personNamed: "Anton")
```
在使用`swiftc`命令编译之后会产生一个名为`Greeter`的可执行文件：
```cmd
$ swiftc -g Greeter.swift
$ ls
Greeter.dSYM
Greeter.swift
Greeter*
```
#### 确定调试程序

首先，使用`lldb`来启动程序。

```cmd
$ lldb Greeter
(lldb) target create "Greeter"
Current executable set to 'Greeter' (x86_64).
```

该命令就会进入`lldb`的调试`session`，能够让你输入不同的`lldb`命令与被调试的程序交互。

#### 设置调试断点

在确定了调试的程序之后，通常需要设置调试断点。断点一般设置在自己需要特别关心的代码块上。

```cmd
(lldb) breakpoint set --line 18
Breakpoint 1: where = Greeter`main + 70 at Greeter.swift:18, address = 0x0000000100001996
```

#### 运行程序

使用`process launch`来运行需要调试的程序。前面`lldb <program>`只是开启了针对`program`的调试session，并没有启动被调试的程序进程。

```cmd
(lldb) process launch
Process 97209 launched: 'Greeter' (x86_64)
Process 97209 stopped
* thread #1: tid = 0x1288be3, 0x0000000100001996 Greeter`main + 70 at Greeter.swift:18, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
frame #0: 0x0000000100001996 Greeter`main + 70 at Greeter.swift:18
15          }
16      }
18
-> 18      let greeter = Greeter()
19
20      greeter.greet(personNamed: "Anton")
21      greeter.greet(personNamed: "Mei")
```
#### 控制程序运行

使用`thread step-over`跳过断点出的函数调用直接到下一个函数调用。

```cmd
(lldb) thread step-over
Process 97209 stopped
* thread #1: tid = 0x1288be3, 0x00000001000019bd Greeter`main + 109 at Greeter.swift:20, queue = 'com.apple.main-thread', stop reason = step over
  frame #0: 0x00000001000019bd Greeter`main + 109 at Greeter.swift:20
  17
  18      let greeter = Greeter()
  19
  -> 20      greeter.greet(personNamed: "Anton")
  21      greeter.greet(personNamed: "Mei")
  22      greeter.greet(personNamed: "Anton")
```

使用`thread step-in`可以进入函数内部。

#### 输出信息

使用`thread backtrace`输出程序执行至目前断点处之前的`栈帧`顺序。

```cmd
(lldb) thread backtrace
* thread #1: tid = 0x1288be3, 0x0000000100001a98 Greeter`Greeter.hasMet(name="Anton", self=0x0000000101200190) -> Bool + 24 at Greeter.swift:5, queue = 'com.apple.main-thread', stop reason = step in
   frame #0: 0x0000000100001a98 Greeter`Greeter.hasMet(name="Anton", self=0x0000000101200190) -> Bool + 24 at Greeter.swift:5
   * frame #1: 0x0000000100001be4 Greeter`Greeter.greet(name="Anton", self=0x0000000101200190) -> () + 84 at Greeter.swift:9
   frame #2: 0x00000001000019eb Greeter`main + 155 at Greeter.swift:20
   frame #3: 0x00007fff949d05ad libdyld.dylib`start + 1
   frame #4: 0x00007fff949d05ad libdyld.dylib`start + 1
```

使用`frame variable`输出当前栈帧上的所有变量。

```cmd
(lldb) frame variable
(String) name = "Anton"
(Greeter.Greeter) self = 0x0000000100502920 {
acquaintances = ([0] = "Anton")
}
```

## 参考资料

- [LLDB Quick Start Guide](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/Introduction.html)
- [LLDB Debugging Guide](https://developer.apple.com/library/content/documentation/General/Conceptual/lldb-guide/chapters/Introduction.html)
- [Advanced Debugging with LLDB - WWDC](https://developer.apple.com/videos/play/wwdc2013/413/)
- [The LLDB Debugger](https://lldb.llvm.org/)
