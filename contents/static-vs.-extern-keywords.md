# 变量 Static VS. Extern 修饰关键字

该讨论基于C语言标准。

C语言本身并不支持在一个函数中定义另外一个函数。那么一个变量的作用域（`scope`）就和C语言本身的块结构特点（`block structure`）紧密联系起来。

所谓块结构`block structure`简单地将就是通过```{```和```}```将作用域划分为一个个```{}```包围的块。使得在块中声明的变量（指`automatic variable`）生命周期就是其声明开始到函数执行离开该块；另外，在一个内部块中定义的于外部块中定义的同名变量相互并不影响。

例如：

```c
void sample_function(void) {
    int i = 0, n = 10; /* outter i */
    if (n > 0) {
	    int i;  /* declare a new i */
	    for (i = 0; i < n; i++) {
		    ...
	    }
    }
}
```

在```for```循环中使用变量`i`是一个内部块中的局部变量；它和进入函数时定义的变量`i`没有什么关系。

理解了C语言的`block structure`特征之后，就比较好理解`external variable`这个概念了。所谓的外部变量就是在任何的函数之外定义的变量。比如，将上面例子修改一下：

```c
int i; /* external variable i */
void sample_function(void) {
    int i = 0, n = 10; /* outter i */
    if (n > 0) {
	    int i;  /* declare a new i */
	    for (i = 0; i < n; i++) {
		    ...
	    }
    }
}
```
最外层定义的变量`i`就是外部变量（`external variable`）。该外部变量，同样道理，和```sample_function(void)```中重新定义的两个同名局部变量`i`是没有关系的，这两个局部变量会将外部变量`i`隐藏起来，使得在`sample_function(void)`内部无法使用外部变量`i`。因此，这其实也说明在里外代码块中使用同名变量并不是一个好的实践。

## `extern`关键字

理解了外部变量（`external variable`）这个概念，那外部变量和`extern`关键字；甚至，和静态变量（`static variable`）以及`static`关键字有什么联系？

- 静态变量`static variable`指的是在该变量已经申明就在内存中常驻；同时，每次改变该变量的值，该值都会保留下来。那么这个概念和通常所说的局部变量是几乎相对的，局部变量每次代码执行到其声明处时会重新申明，超过其申明的块时就被销毁。
- 外部变量`external variable`在内存属性上其实就是`static variable`。只不过，它定义处为所有函数的外部。
- `extern`关键字用来申明外部变量，使得该外部变量可以在该申明之后被使用。
- `static`关键字用来申明某个变量的内存属性为静态变量属性。同时，限定该变量的使用为其申明时所处的块范围或者文件范围（若其为外部变量的话）。

结合下面例子来说明这个上面几个结论。例子包括三个文件：
- `stack.h` 一个简单的栈结构头文件

    ```c
    /* contents of stack.h */

    #ifndef stack_h
    #define stack_h

    #define MAXVAL (100)

    void push(double);
    double pop(void);

    #endif /* stack_h */
    ```
- `stack.c` 该栈结构的实现

    ```c
    #include <stdio.h>
    #include "stack.h"

    int sp = 0;
    double val[MAXVAL];

    void push(double value) {
	    if (sp < MAXVAL) {
	    	val[sp++] = value;
	    }
	    else {
	    	printf("error: stack full. Can't push %g\n", value);
	    }
	}

	double pop(void) {
	    if(sp > 0) {
	    	return val[--sp];
	    }
	    else {
	    	printf("error: stack empty.");
		return 0.f;
	    }
	}
    ```

- `extern.h` 用于将外部变量的申明放在一起

    ```c
    /* contents of extern.h */
    #ifndef extern_h
    #define extern_h

    extern int sp;
    extern double val[];

    #endif /* extern_h */
    ```
由于函数```void push(double value)```和```void pop(void)```都需要公用这个栈的内部存储元素和现在栈指针（这里并非真正意义上的指针）的位置，所以需要将变量`val`和`sp`定义为外部变量，方便访问。

如果外部函数同样需要知道这个栈（尽管几乎没有这个必要）内部的数据结构实现，可以引入`extern.h`这个头文件；就能够使用`val`和`sp`这两个在`stack.c`中定义的外部变量。

因此，这里就能理解：`extern`关键字仅仅是用来在不可以见外部变量定义的情况下，使得该外部变量在它处可用。

所以，在```main```函数中，如果引入```extern.h```头文件。就可以可见`sp`和`val`。

```c
#include <stdio.h>
#include "stack.h"
#include "extern.h"

int main(int argc, const char * argv[]) {
    push(2.0);
    printf("print stack position - sp:%d \n", sp);
    pop();
    printf("print stack position - sp:%d \n", sp);

    return 0;
}
```
输出结果:
> print stack position - sp:1 <br> 
> print stack position - sp:0 

实际上，可以在多个使用需要使用该外部变量的文件中引入`extern.h`，也就是多个文件中保留了同样的多份申明；这是完全可以的。

但是，***对这两个外部变量的定义只能有一份***；在示例中，放在了`stack.c`文件中。（所谓定义，就是给变量申请分配了内存空间的地方）。

## `static`关键字

上面例子有一个很大的缺陷：一个栈的内部实现不应该暴露给外部。因此，尽管需要设计`sp`和`val`为全局变量使得他们`globally accessible`。另外一方面也需要将它们的可视范围限定在`stack.c`文件中。

这时候，就可以将`stack.c`中这两个全局变量的定义前（内存属性上也是静态变量）前面加上`static`关键字。

```c
#include <stdio.h>
#include "stack.h"

static int sp = 0;
static double val[MAXVAL];

... /* code not shown */
```
在不修改任何其他代码的前提下运行程序：

> Undefined symbols for architecture x86_64: <br>
>   "_sp", referenced from: <br>
>   _main in main.o <br>
>   ld: symbol(s) not found for architecture x86_64

也就是`main`函数中找不到`sp`的定义了。（仍然保留了`extern.h`头文件)。

当然，在一个函数的内部，也可以声明局部变量为静态变量。这个意义是有两层的：
- 该局部变量在内存表现上成为了静态变量。为保留修改值。
- 其可见范围为定义的这个块范围中（由于本身就是局部变量，所以这个并没有改变什么）。
