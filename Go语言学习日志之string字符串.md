## Go语言学习日志之string字符串

学习任何一门编程语言，再在自己能力范围内去阅读源码是非常必要的，下面就是我对Go中的字符串类型的一些粗浅的理解。
先来看看官方是怎么描述string的：

```go
// string is the set of all strings of 8-bit bytes, conventionally but not
// necessarily representing UTF-8-encoded text. A string may be empty, but
// not nil. Values of string type are immutable.
type string string
```
注意这句话：

>  Values of string type are immutable.

翻译过来就是string类型的值是不可改变的。刚学Go的时候觉得它和C语言很像，而在我的印象中C和C++中的字符串是可以改变的，本着科学严谨的态度我简单写了一段代码印证了我的想法：

```cpp
#include<bits/stdc++.h>
using namespace std;
int main(){
	string s="abc";
	printf("Unchanged s[1]:%c\n",s[1]);
	printf("Unchanged s:%s\n",s.c_str());
	s[1]='a';
	printf("Changed s[1]:%c\n",s[1]);
	printf("Changed s:%s\n",s.c_str());
}
```
输出结果如下：

```bash
Unchanged s[1]:b
Unchanged s:abc
Changed s[1]:a
Changed s:aac

--------------------------------
Process exited with return value 0
Press any key to continue . . .
```
很明显C++中的字符串是可以改变值的。
而在Go语言中字符串是不可变的，字符串不可变可以保证线程安全，所有进程使用的都是只读对象，无须加锁；再者，方便内存共享，而不必使用写时复制等技术；字符串 hash 值也只需要制作一份。先来看看Go语言中如何“改变”字符串吧：

```go
str := "I am a string"
strBytes := []byte(str)
```
通过将字符串转换为数组进行修改，然后再将修改完后的数组转化为字符串进行保存。
将这一点弄清楚之后我又想到一个问题：在Go语言中字符串是可以重新赋值的：

```go
func main() {
	s1 :="I am String1"
	s2 :="I am String2"
	s1=s2
	fmt.Print(s1)
}
```
输出的结果是：

```bash
I am String2
Process finished with exit code 0
```
此时我猜测，会不会是Go将s1的地址空间替换为s2的地址空间从而这两者共享一个地址空间呢？

```go
func main() {
	s1 :="I am String1"
	s2 :="I am String2"
	s1=s2
	fmt.Print(&s1==&s2)
}
```

输出的结果是：

```bash
false
Process finished with exit code 0
```
这说明在将s2的值赋给s1之后这哥俩并没有变成一家人而是仍然分房住，只不过将s2中的内容复制过去了而已。不放心，又试了一下s1的地址到底变没变：

```go
func main() {
	s1 :="I am String1"
	fmt.Print(&s1)
	s2 :="I am String2"
	s1=s2
	fmt.Print("\n")
	fmt.Print(&s1)
}
```
输出的结果是：

```bash
0xc0000881e0
0xc0000881e0
Process finished with exit code 0
```
由此得知我上文的推测是正确的。
以上内容均为个人浅薄的理解，如有不当之处烦请各位大佬不吝赐教，批评指正，十分感谢！