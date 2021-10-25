---
# 常用定义
title: "go知识巩固"           # 标题
date: 2021-01-01T16:01:23+08:00    # 创建时间
lastmod: 2021-01-02T16:01:23+08:00 # 最后修改时间
draft: false                       # 是否是草稿？
tags: ["go"]  # 标签
categories: ["golang"]              # 分类
author: "wind chime"                  # 作者

# 用户自定义
# 你可以选择 关闭(false) 或者 打开(true) 以下选项
comment: true   # 关闭评论
toc: true       # 关闭文章目录
# 你同样可以自定义文章的版权规则
contentCopyright: '<a rel="license noopener" href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>'
reward: true	 # 关闭打赏
mathjax: true    # 打开 mathjax


menu:
  main:
    parent: "docs"
    weight: 2
---

# go 知识巩固

## 指针

Go语言保留了指针，但与C语言指针有所不同，主要体现在

- 默认值为nil
 
- 操作符 "&" 取变量地址，"*" 通过指针访问目标对象
 
- 不支持指针运算，不支持 "_>" 运算符，直接用 "." 访问目标成员


		 指针就是地址，。指针变量就是存储地址的变量。

		 *p : 解引用，间接引用

		 栈帧存储：1.局部变量 2.形参 (形参与局部变量存储的地位等同) 3. 内存字段描述值
		
```go
func main() {
	var a int = 100

	fmt.Println("a = ", a)

	var p *int = &a
	//借助 a 变量的地址，操作 a 对应的空间
	*p = 1000 //*p 取值运算（解引用）

	fmt.Println("a = ", a)
	fmt.Println("*p = ", *p)

	a = 250

	fmt.Println("*p = ", *p)

	test(a)
}

func test(m int) {
	b := 100
	m += b
}
```
		一个函数就是一个栈帧，一个栈帧由栈基指针，栈顶指针分配空间，当main()函数调用另test()函数的时候，原本分配main()函数空间的指针去给test()函数分配空间，同时main()函数的栈帧存储着这两个指针的值，当test()函数执行结束之后，test()栈帧释放，指针回到main()函数栈帧记录的位置。

****
### new()
![内存存储图]("./png/memer.png")

```go
func main() {
	var p *string

	//在heap 上申请一片内存地址空间

	p = new(string)

	*p = "100"

	fmt.Printf("%q\n", *p) //默认类型的 默认值
}
```
    var p *string  在stack  上开辟一个空指针，p =new(string) 在heap 上开篇一个对象空间，并且将这个对象的地址值赋给 p ,此时*p 的默认值为默认类型的默认值(此处为""),*p = "100" 将 "100" 这个值写入到heap中开辟的对象里。

__指针使用注意:__

>-	空指针：未被初始化的指针。 var p  *int
>-  野指针：被一片无效的地址空间初始化

**变量存储:**

>-	左值：等号左边的变量，代表变量所指向的内存空间。 此时是写操作
>-  右值：等号右边的变量，代表变量内存空间存储的数据值。 此时是读操作

**注意:**

>  当方法结束后，栈帧被释放，但是在方法里创建的对象并没有被释放，因为他是存在于heap上的。(栈内存默认为1M大小(可以手动分配大小，linux环境下最大16M)，但是堆内存默认是1G以上的)

### 指针的函数传参

- 传地址(引用): 将地址值作为函数参数/返回值后传递

- 传值: 将实参的值拷贝一份给行参。

>>>所有函数传参都是值传递

```go
func main() {
	a, b := 10, 20
	swap1(a, b) //值传递，实参将自己的值拷贝一份，给形参

	fmt.Printf("main :a=%d,b=%d\n", a, b) //main :a=10,b=20

	c, d := 10, 20
	swap2(&c, &d) //值传递，实参将自己的值(地址值)拷贝一份，给形参

	fmt.Printf("main :c=%d,d=%d\n", c, d) //main :c=20,d=10
}

func swap1(a, b int) {
	a, b = b, a

	fmt.Printf("swap1:a=%d,b=%d\n", a, b) //swap1:a=20,b=10
}

func swap2(c, d *int) {
	*c, *d = *d, *c //等号左边的*c，*d 代表空间内存，等号右边的 *c,*d代表内容(值)

	fmt.Printf("swap2:*c=%d,*d=%d\n", *c, *d) //swap2:*c=20,*d=10
}
```
**传引用: 在A栈帧内部，修改B栈帧中的变量值**

## 切片(slice)

### 为什么用切片
1. 数组的内容是固定的，不能自动扩容
2. 值传递，数组作为函数参数时，将整个数组拷贝一份给行参

**在Go语言中，我们几乎可以在所有场景中，使用切片替换数组使用**

### 切片的本质

**不是一个数组的指针，是一种数据结构，用来操作数组内部元素**
```go
		type notInHeapSlice struct {
		array *notInHeap        //底层数组的指针
		len   int              //切片的长度
		cap   int              //切片的容量
		}
```
### 切片与数组的区别

	创建数组时 [] 内指定数组长度
	创建切片时 [] 内为空，或者为 ...

__截取数组建切片：__ 切片名称[low:high:max]
		
		low: 起始下标位置
		high: 结束下标位置;  len = high - low
		max: 容量结束下标位置; cap = max - low

**截取数组，初始化切片时，没有指定切片容量，切片容量跟随原数组(切片)**

		s[:high:max] :从0开始，到high结束;  「左闭右开」
		s[low:] :从low开始，到末尾
		s[:high] :从0开始，到high结束，容量跟随原先的容量。「常用」
```go
func main() {
	arr := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

	s := arr[2:5]

	fmt.Println("s=", s)            //s= [3 4 5]
	fmt.Println("s.len()=", len(s)) //s.len()= 3
	fmt.Println("s.cap()=", cap(s)) //s.cap()= 8

	s2 := s[2:7]
	fmt.Println("s2=", s2)            //s2= [5 6 7 8 9]
	fmt.Println("s2.len()=", len(s2)) //s2.len()= 5
	fmt.Println("s2.cap()=", cap(s2)) //s2.cap()= 6
}
```
### 切片的创建

	1. 自动推导类型创建切片。slice :=[]int{1,2,3,4}
	2. slice := make([]int,len(),cap())
	3. slice := make([]int,len())    创建切片时，没有指定容量，容量= 长度。 「常用」

````go
func main() {
	slice1 := []int{1, 2, 3, 4}
	//slice1=[1 2 3 4],len()=4,cap()=4
	fmt.Printf("slice1=%v,len()=%d,cap()=%d\n", slice1, len(slice1), cap(slice1))

	slice2 := make([]int, 2, 5)
	//slice2=[0 0],len()=2,cap()=5
	fmt.Printf("slice2=%v,len()=%d,cap()=%d\n", slice2, len(slice2), cap(slice2))

	slice3 := make([]int, 2)
	//slice3=[0 0],len()=2,cap()=2
	fmt.Printf("slice3=%v,len()=%d,cap()=%d\n", slice3, len(slice3), cap(slice3))
}
````
>  **注:切片作为参数---传引用(传地址)**

### append基本使用

````go
	append(切片对象,追加的元素)
````

```go
func main() {
	slice := []int{1, 2, 3, 4}
	slice = append(slice, 888)
	slice = append(slice, 888)
	slice = append(slice, 888)
	slice = append(slice, 888)
	slice = append(slice, 888)
	fmt.Println("slice=", slice)   //slice= [1 2 3 4 888 888 888 888 888]
}
```
>  **append函数会智能将底层的容量自动扩容，一旦超过底层数组容量，通常以2倍(1024以下)容量重新分配底层数组。因此，使用append给切片扩容时，切片的地址可能发生变化，但，数据重新保存了，不影响使用**

### copy的基本使用

> copy(目标位置切片，源切片)

```go
func main() {
	data := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
	s1 := data[8:]
	s2 := data[:5]

	copy(s2, s1)

	fmt.Printf("s2=%v\n", s2)  //s2=[9 2 3 4 5]
}
```

练习: 删除slice 中间的某个元素并保存原有的元素顺序
```go
func deleteNode() {
	data := []int{1, 2, 3, 4, 5} //删除元素 3

	copy(data[2:], data[2+1:])

	data = data[:len(data)-1]

	fmt.Printf("data=%v\n", data)
}
```


## map(字典，映射)

>  **key：无序、唯一。不能是引用类型数据***
### map声明、初始化、赋值

```go
func main() {
	//申明但是没有初始化，不能赋值，= nil
	var map1 map[int]string
	fmt.Printf("map1=%v\n", map1)

	//声明并且初始化且赋值, key  不能重复
	var map2 = map[int]string{1: "wnag", 2: "duo", 3: "cong"}
	fmt.Printf("map2=%v\n", map2)

	//申明且初始化,长度为0, key 重复会覆盖
	map3 := map[int]string{}
	map3[1] = "wind"
	map3[4] = "chime"
	fmt.Printf("map3=%v\n", map3)

	//make  初始化 不指定长度, key 重复会覆盖

	map4 := make(map[int]string)
	map4[1] = "li"
	map4[4] = "qiao"
	fmt.Printf("map4=%v\n", map4)

	//make  初始化并指定长度, key 重复会覆盖

	map5 := make(map[int]string, 5)
	map5[4] = "li"
	map5[6] = "yu"
	map5[1] = "mei"
	fmt.Printf("map5=%v\n", map5)
}
```
> **map 没有容量的概念，只有长度。当到 map 中添加数据时，超出map长度，map会自动扩容**





