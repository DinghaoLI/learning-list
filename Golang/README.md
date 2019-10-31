# Golang

## 1. 书籍


[golang圣经](https://github.com/golang-china/gopl-zh)

[effective go](https://golang.org/doc/effective_go.html)


## 2. 优质内容作者 & 专栏


[go语言中文网](https://studygolang.com/)

[legendtkl Golang](http://legendtkl.com/categories/golang/page/2/)

[EDDYCJY Golang](https://github.com/EDDYCJY/blog)

[golang 源码阅读之路](https://zhuanlan.zhihu.com/c_1010470599088594944)

[For-learning-Go-Tutorial](https://github.com/KeKe-Li/For-learning-Go-Tutorial)

[Golang 面试汇总](https://github.com/KeKe-Li/data-structures-questions/blob/master/src/chapter05/golang.01.md)


## 3. Golang 优质仓库


[awesome-go: A curated list of awesome Go frameworks, libraries and software](https://github.com/avelino/awesome-go)

[go-patterns 设计模式](https://github.com/tmrts/go-patterns)


## 4. Golang相关知识点、面试问题与相关文章


### 1. Golang简介&特点：

- 谷歌开发的一种系统编程语言
- 内置GC
- 支持高并发
- 可编译成单个可执行二进制文件，不需要添加库或运行时环境就可以在服务器上直接执行

### 2. Golang中的类以及类型断言：

[golang 类型断言](https://blog.csdn.net/MaxCoderLlj/article/details/80105040)

### 3. Golang是否支持泛型编程？为什么不支持？

- 目前暂时不支持泛型，Golang 2.0 可能会加入
- 泛型编程方便，但是在类型系统和运行时的复杂性方面付出代价 => C++ template 在运行时会多出判断类型+拷贝函数的动作，会增加内存开销，且拷贝出来的函数占用的内存无法释放

### 4. 编译命令

```
go run xxx.go: 直接编译并运行go文件，可执行文件在临时文件夹中
go build xxx.go: 在当前目录产生同名可执行程序
go get mypack: 下载包源码到当前GOPATH/src目录下
go install mypack: 对指定的包进行编译安装：
	- 如果mypack是一个类库包，则在当前GOPATH/pkg下生成对应的包文件(.a文件是编译过程中生成的，每个package都会生成对应的.a文件，Go在编译的时候先判断package的源码是否有改动，如果没有的话，就不再重新编译.a文件，这样可以加快速度)
	- 如果mypack是一个含有main主程序的包，则当前GOPATH/bin下生成可执行程序
```

### 5. 值传递/引用传递

- Go中的引用类型: slice, map, channel, pointer, interface
- 其余都是值类型
- 在变量传递的时候（函数传参）时，指类型是拷贝传递，引用类型是地址传递


### 6. 如何提前释放 make()分配的内存(slice, map, channel)/其他的引用类型(struct指针，interface{}) => 想让其提前被gc回收

- make()用于go语言中三种引用类型的内存分配：slice、map、channel
- 引用类型创建出来的对象：值的内存分配在heap中，并在stack中使用一个指针指向这个值
- 需要回收只需要将栈中的指针/引用指向nil即可，
- 例如：**如果一个函数运行很久，但是想提前释放引用类型所占用的内存。此时可以通过 buffer = nil 去接触解除引用，gc会通过三色标记法将heap中没有被引用的内存回收，buffer（local变量）本身需要等这个函数运行结束后才自动释放。**

### 7. slice和数组的区别，slice的结构，双倍动态扩容

- 数组大小固定，slice不是
- slice运行时动态增加slice大小，数组不行
- len()是slice中元素个数
- cap()返回slice的容量，即可以容纳的元素数量
- 数组也有cap()方法，只不过是恒定的
- **注意slice扩容前和扩容后(分配一个大一倍的新地址)的指向地址变化**

**相关文章:**

[Go slice](https://jsharkc.github.io/2017/07/10/Slice机制/)

[Golang中slice与二级指针的陷阱](https://zhuanlan.zhihu.com/p/51203586)

### 8. 哈希表/哈希映射 => Golang中的map

**相关文章:**

[为什么遍历 Go map 是无序的？](https://gocn.vip/article/1704)

[Golang map源码详解](https://juejin.im/entry/5a1e4bcd6fb9a045090942d8)

### 9. select作用

- 除 default 外，如果只有一个 case 语句评估通过，那么就执行这个case里的语句
- 除 default 外，如果有多个 case 语句评估通过，那么通过伪随机的方式随机选一个
- 如果 default 外的 case 语句都没有通过评估，那么执行 default 里的语句
- 如果没有 default，那么 代码块会被阻塞，指导有一个 case 通过评估；否则一直阻塞
- 如果 case 语句中 的 receive 操作的对象是 nil channel，那么也会阻塞

**相关文章:**

[Go select](https://segmentfault.com/a/1190000006815341)

### 10. 主协程如何等待子协程

- WaitGroup: 开n个协程就总共Add(n)，每个协程结束需要Done()，主协程Wait()
- slice: 10子协程，每个协程结束时向公共slice写入true，主协程循环查看slice长度，为10就退出

### 11. 什么时候会死锁，如何避免？

- 主协程被阻塞
- 不能单个协程自渎自写一个无缓冲channel
- A要求B先写入自己在读出，B要求A先读出自己在写入，此时AB死锁
- range channel时，注意channel的写入关闭，如果不关闭，range channel会永远阻塞

### 12. channel

- 有缓冲的管道允许，即使没人读出，也能写入若干值才阻塞，读出读到空时阻塞

**相关文章:**

[图解Go的channel底层实现](https://i6448038.github.io/2019/04/11/go-channel/)

[深入理解 Go Channel](http://legendtkl.com/2017/07/30/understanding-golang-channel/)

[Go Channel 源码剖析](http://legendtkl.com/2017/07/30/understanding-golang-channel/)


### 13. Golang如何实现同步调度

- 无缓冲channel的读写阻塞
- sync包下的同步:
	- 读写锁的加锁解锁阻塞
	- 等待组，WaitGroup.Wait()知道WaitGroup中所有协程全部WaitGroup.Done()
	- 条件变量，cond.Wait()知道有人cond.Signal()或者cond.Broadcast()
	- 原子操作：CAS

**相关文章:**

[条件变量](https://cloud.tencent.com/developer/article/1400153)


### 14. 调度模型 & Goroutine

[Head First of Golang Scheduler](https://zhuanlan.zhihu.com/p/42057783)

[Go并发调度器解析之实现一个协程池](https://zhuanlan.zhihu.com/p/37754274)

[golang 抢占式调度](https://www.jianshu.com/p/469d0c7a7936)

[Go语言模型：Linux线程调度 vs Goroutine调度](https://blog.csdn.net/thisinnocence/article/details/80474698)

[Golang - 调度剖析【第一部分】](https://segmentfault.com/a/1190000016038785)

[Golang - 调度剖析【第二部分】](https://segmentfault.com/a/1190000016611742)

[Go语言——goroutine并发模型](https://www.jianshu.com/p/f9024e250ac6)

[Golang - 调度模型](http://wudaijun.com/2018/01/go-scheduler/)

[goroutine 调度策略](https://my.oschina.net/renhc/blog/2221426)

[Golang 的 goroutine 是如何实现的？ - Yi Wang的回答 - 知乎](https://www.zhihu.com/question/20862617/answer/27964865)


### 15. GC

[Golang - GC](http://legendtkl.com/2017/04/28/golang-gc/)

[golang GC回收策略](https://github.com/tiancaiamao/go.blog/blob/master/content/gc.md)

[Golang - 垃圾回收](https://cloud.tencent.com/developer/article/1072602)

[Golang - 内存管理](http://legendtkl.com/2017/04/02/golang-alloc/)


### 其他

[深度解析 Go 语言中「切片」的三种特殊状态 - 「零切片」、「空切片」和「nil 切片」](https://juejin.im/post/5bea58df6fb9a049f153bca8)

[golang interface原理](https://tiancaiamao.gitbooks.io/go-internals/content/zh/07.2.html)

[golang derfer = 赋值指令 + CALL defer指令 + RET指令](http://www.zenlife.tk/golang-defer.md)

[golang defer实现原理](https://studygolang.com/articles/16067)

[Go语言并发模型：使用 select](https://segmentfault.com/a/1190000006815341)

[golang - context](https://www.flysnow.org/2017/05/12/go-in-action-go-context.html)


[Golang 互斥锁内部实现](https://zhuanlan.zhihu.com/p/27608263)

[Golang Mutex实现 trylock](https://colobu.com/2017/03/09/implement-TryLock-in-Go/#%E5%AE%9E%E7%8E%B0%E8%87%AA%E6%97%8B%E9%94%81)

[golang 自旋锁](https://studygolang.com/articles/16480)

[Golang 互斥锁内部实现](https://zhuanlan.zhihu.com/p/27608263)

[Golang 剖析锁](http://legendtkl.com/2016/10/13/about-lock/)

[Golang 互斥锁](http://legendtkl.com/2016/10/23/golang-mutex/)


[Golang同步：原子操作使用](https://www.kancloud.cn/digest/batu-go/153537)

[Go sync.Map原理&使用](http://www.gogodjzhu.com/index.php/code/basic/397/)

[Golang简单socket编程](https://hiberabyss.github.io/2018/03/14/unix-socket-programming/)

[Golang 闭包](https://juejin.im/post/5c850d035188257ec629e73e)



