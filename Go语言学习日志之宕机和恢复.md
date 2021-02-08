## Go语言学习日志之宕机和恢复
首先来看**宕机的定义**：GO语言的类型系统会捕获许多编译时的错误，但有些其他的错误，例如数组越界访问或直接引用空指针，都需要在运行时进行检查。当Go语言运行时检测到这些错误，它就会发生**宕机**。一个典型的宕机发生时，正常的程序执行会终止，goroutine中的所有延迟函数会执行，然后程序会异常退出并留下一条日志消息。日志消息包括宕机的值，这往往代表某种错误消息，每一个goroutine都会在宕机时显示一个函数调用的栈跟踪消息。我们可以根据这条消息在不需要重新运行一次该程序的情况下诊断发生的问题。
并不是所有宕机都是在运行时发生的。我们可以直接调用Go语言内置的宕机函数。例如我们可以使用`panic()`函数来引发异常从而引起宕机，从而处理一些“不可能发生的情况”，例如一个人既不是男的也不是女的，这个时候就应该异常退出：

```go
switch sex:=people.sex; sex{
case "male":
case "female":
default:
	panic(fmt.Sprintf("Wrong sex :%s",sex))
}
```

还有一点就是`panic()`函数可以接收任何值作为参数，且可以用后面要介绍到的`recover()`函数进行接收。
当宕机发生时，会以倒序执行所有的延迟函数，下面来看例子：

```go
func main() {
	f(3)
}

func f(x int)  {
	fmt.Printf("f(%d)\n",x+0/x)
	defer fmt.Printf("defer %d\n",x)
	f(x-1)
}
```
输出结果：

```bash
f(3)
f(2)
f(1)
defer 1
defer 2
defer 3
```
上述的代码可以明显看出调用到f(0)的时候会发生`integer divide by zero`异常，所以接下来还会输出异常信息：

```bash
panic: runtime error: integer divide by zero

goroutine 1 [running]:
main.f(0x0)
        D:/WorkSpace/Go/GoLearning/src/defer1.go:10 +0x1e8
main.f(0x1)
        D:/WorkSpace/Go/GoLearning/src/defer1.go:12 +0x189
main.f(0x2)
        D:/WorkSpace/Go/GoLearning/src/defer1.go:12 +0x189
main.f(0x3)
        D:/WorkSpace/Go/GoLearning/src/defer1.go:12 +0x189
main.main()
        D:/WorkSpace/Go/GoLearning/src/defer1.go:6 +0x31
```
之后会介绍到一个`recover()`函数可以让函数从宕机状态恢复到正常运行状态而不让程序退出。
runtime包提供了转储存栈的方法使我们可以诊断错误：

```go
defer printStack()
```
这样就可以延迟`printStack()`函数执行并输出已经不存在的栈的信息。
宕机讲完了，下面来讲讲怎么**恢复**：退出程序通常是正确处理宕机的方式，但在一些情况下宕机是可以恢复的，或者说至少在理清当前的混乱情况后再退出。`recover()`函数在延迟函数的内部调用，且这个包含`defer`语句的函数发生宕机，那么`recover()`函数会终止当前宕机的状态并返回宕机的值。原函数不会从宕机的地方继续运行而是正常返回。`recover()`函数在其他任何情况下使用均没有效果且返回`nil`。这么说可能还是有点抽象，来看一下例子：

```go
func main() {
	defer func(){
		fmt.Print("First in main()\n")
		if err:=recover();err!=nil{
			fmt.Println(err)
		}
		fmt.Print("Second in main()\n")
	}()
	f()
}

func f()  {
	fmt.Print("First in f()\n")
	panic("Panic!")
	fmt.Print("Second in f()\n")
}
```
来看输出结果：

```bash
First in f()
First in main()
Panic!
Second in main()
```
这个结果说明了两点，第一点就是上文所说的`recover()`函数在延迟函数的内部调用可以接收到`panic()`函数中的参数。第二点就是原函数不会从宕机的地方继续运行而是正常返回。我们可以发现函数`f()`中在`panic("Panic!")`语句前的语句都被正常执行了，而`fmt.Print("Second in f()\n")`并没有被执行。在`main()`函数中`fmt.Print("Second in main()\n")`语句被执行了，证明`recover()`函数确实使宕机恢复并输出了错误信息。综上所述就可以证明第二点的正确性。
以上均为个人的一点粗浅的理解，如有不当之处请各位大佬指正，十分感谢！