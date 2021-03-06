## Go语言学习日志之defer机制
今天在学习Go语言中的函数时，遇到了用于延迟函数调用的defer关键字，觉得defer机制对于我日后用Go开发项目会有重大的意义，于是写这篇博客来记录我的学习心得。
话不多说，先来看defer关键字的用法：

```go
defer resp.Body.Close()
```
可以看出用法很简单，就是在正常的语句前加上一个defer关键字就行了。defer关键字修饰的语句会推迟到执行return语句或函数执行完毕以及发生错误之后才会执行。defer语句常用于成对的操作，例如打开和关闭，连接和断开，加锁和解锁。defer机制能够保证无论再复杂的控制流，在任何情况下，资源都能被正确地释放。我个人觉得和Java的try/catch中的finally有些相似，其意义在于在发生各种不可预料的错误时某个操作仍能被正确执行，这样可以提高系统的稳定性和可用性，避免出现过高的资源占用以及死锁问题。
在介绍defer的具体使用方式之前，我想先介绍一下defer的几个特性：
**特性一：defer关键字修饰的语句遵循后进先出的原则**
简单来说就是后定义的defer语句会先被执行，话不多说看例子：

```go
func fun()  {
	for i := 0; i < 5; i++ {
		defer fmt.Print(i)
	}
}
```
输出结果：

```bash
43210
Process finished with exit code 0
```
个人猜测这和内存的数据结构栈有一定的关系，因为栈就是一个后进先出的数据结构。（仅为猜测，如有懂的大佬烦请指导指导）

**特性二：defer被声明时，所含参数就会被实时解析**
上文也说了，defer关键字修饰的语句会推迟执行，那么可否认为defer关键字修饰的语句中的参数的值会是最终的值而不是过程中的值呢？这么说或许有些抽象，我们来看例子：

```go
func fun()  {
	var i=0
	defer fmt.Print(i)//此处i为初始值0
	i++
	defer fmt.Print(i)//此处i经过自增值为1
	return
}
```
来看输出结果：

```bash
10
Process finished with exit code 0
```
这个输出结果说明上文的猜测是错误的，如果defer关键字修饰的语句中的参数读取的是函数执行完成后得到的最终值，那么结果应该是11才对，而此处的结果是10（至于为什么1先输出上文的特性一已经介绍过了）。那么很显然defer关键字修饰的语句中的参数的值是实时解析的，即定义在哪就读取哪的值。
**特性三：defer关键字修饰的匿名内部函数可以读取其外层函数函数的返回值**
书上的描述是延迟执行的匿名函数能够改变外层函数返回给调用者的结果，这么说还是有点抽象，没关系我们直接来看例子：

```go
func fun() (result int) {
	defer func() {result++}()
	return 1
}

func main() {
	fmt.Print(fun())
}

```
输出结果：

```bash
2
Process finished with exit code 0
```
这个结果说明defer关键字修饰的匿名内部函数可以读取其外层函数函数的返回值且可以修改其外层函数返回给调用者的结果。
**总结：**
defer机制可以让我们优雅的释放我们所需要释放的资源而不需要担心中间环节，但这也要求我们对于运用defer机制后输出的结果有精准的认知。希望我个人的一点粗浅的理解能够对大家有所帮助，如有不当之处欢迎各位大佬指正！