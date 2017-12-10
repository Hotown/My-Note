# Golang —— goroutine（协程）和channel（管道）

## 协程（goroutine）

`协程（goroutine）`是Go中应用程序并发处理的部分，它可以进行高效的并发运算。

+ 协程是轻量的，比线程更廉价。使用4K的栈内存就可以在内存中创建。
+ 能够对栈进行分割，动态地增加或缩减内存的使用。栈的管理会在协程退出后自动释放。
+ 协程的栈会根据需要进行伸缩，不出现栈溢出。


### 协程的使用

```Golang
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("In main()")
	go longWait()
	go shortWait()
	fmt.Println("About to sleep in main()")

	//time.Sleep(4 * 1e9)
	time.Sleep(10 * 1e9)
	fmt.Println("At the end of main()")
}

func longWait() {
	fmt.Println("Beginning longWait()")
	time.Sleep(5 * 1e9)
	fmt.Println("End of longWait()")
}

func shortWait() {
	fmt.Println("Beginning shortWait()")
	time.Sleep(2 * 1e9)
	fmt.Println("End of shortWait()")
}
```

Go中用`go`关键字来开启一个协程，其中`main`函数也可以看做是一个协程。

不难理解，上述代码的输出为：

```Golang
In main()
About to sleep in main()
Beginning shortWait()
Beginning longWait()
End of shortWait()
End of longWait()
At the end of main()
```

但是，当我们将`main`的睡眠时间设置成4s时，输出发生了改变。

```Golang
In main()
About to sleep in main()
Beginning shortWait()
Beginning longWait()
End of shortWait()
At the end of main()
```

程序并没有输出`End of longWait()`，原因在于，`longWait()`和`main()`运行在不同的协程中，两者是异步的。也就是说，早在`longWait()`结束之前，main已经退出，自然也就看不到输出了。

## 通道（channel）

`通道（channel）`是Go中一种特殊的数据类型，可以通过它们发送类型化的数据在协程之间通信，避开内存共享导致的问题。

通道的通信方式保证了同步性，并且同一时间只有一个协程能够访问数据，不会出现`数据竞争`。

以工厂的传输带为例，一个机器放置物品（生产者协程），经过传送带，到达下一个机器打包装箱（消费者协程）。

![](https://raw.githubusercontent.com/Unknwon/the-way-to-go_ZH_CN/master/images/14.2_fig14.1.png)

### 通道的使用

在学习使用管道之前，我们先来看一个“悲剧”。

```Golang
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("Reveal romantic feelings...")
	go sendLove()
	go responseLove()
	waitFor()
	fmt.Println("Leaving ☠️....")
}

func waitFor() {
	for i := 0; i < 5; i++ {
		fmt.Println("Keep waiting...")
		time.Sleep(1 * 1e9)
	}
}

func sendLove() {
	fmt.Println("Love you, mm ❤️")
}

func responseLove() {
	time.Sleep(6 * 1e9)
	fmt.Println("Love you, too")
}
```

用上面学习的知识，不难看出。。。真的惨啊

```Golang
Reveal romantic feelings...
Love you, mm ❤️
Keep waiting...
Keep waiting...
Keep waiting...
Keep waiting...
Keep waiting...
Leaving ☠️....
```

明明收到了暗恋女孩的回应，然而却以为对方不接受自己的情感，含泪离去。【TAT】

可见，协程之间没有互相通信将会引起多么大的误解。幸好，我们有了`channel`，现在就来一起改写故事的结局吧~

```Golang
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan string)
	var answer string

	fmt.Println("Reveal fomantic feelings...")
	go sendLove()
	go responseLove(ch)
	waitFor()
	answer = <-ch

	if answer != "" {
		fmt.Println(answer)
	} else {
		fmt.Println("Dead ☠️....")
	}

}

func waitFor() {
	for i := 0; i < 5; i++ {
		fmt.Println("Keep waiting...")
		time.Sleep(1 * 1e9)
	}
}

func sendLove() {
	fmt.Println("Love you, mm ❤️")
}

func responseLove(ch chan string) {
	time.Sleep(6 * 1e9)
	ch <- "Love you, too"
}
```

输出为：

```Golang
Reveal fomantic feelings...
Love you, mm ❤️
Keep waiting...
Keep waiting...
Keep waiting...
Keep waiting...
Keep waiting...
Love you, too
```

皆大欢喜。

这里我们用`ch := make(chan string)`创建了一个string类型的管道，当然我们还可以构建其他类型比如`ch := make(chan int)`，甚至一个函数管道`funcChan := chan func()`。

我们还用到了一个通信操作符`<-`。

+ 流向通道：`ch <- content`，用管道ch发送变量content。
+ 从通道流出：`answer := <- ch`，变量answer从通道ch接收数据。 
+ `<- ch`可以单独调用，以获取通道的下一个值，当前值会被丢弃，但是可以用来验证，比如：

    ```Golang
    if <- ch != 100 {
        /* do something */
    }
    ```

### 通道阻塞

+ 对于同一通道，发送操作在接受者准备好之前是不会结束的。这就意味着，如果一个无缓冲通道在没有空间接收数据的时候，新的输入数据无法输入，即发送者处于阻塞状态。
+ 对于同一通道，接收操作是阻塞的，直到发送者可用。如果通道中没有数据，接收者会保持阻塞。

以上两条性质，反映了`无缓冲通道`的特性：`同一时间只允许至多一个数据存在于通道中`。

我们通过例子来感受一下：

```Golang
package main

import "fmt"

func main() {
	ch1 := make(chan int)
	go pump(ch1)
	fmt.Println(<-ch1)
}

func pump(ch chan int) {
	for i := 0; ; i++ {
		ch <- i
	}
}
```

程序输出：

```
0
```

这里的`pump()`函数被称为`生产者`。

### 解除通道阻塞

```Golang
package main

import "fmt"
import "time"

func main() {
	ch1 := make(chan int)
	go pump(ch1)
	go suck(ch1)
	time.Sleep(1e9)
}

func pump(ch chan int) {
	for i := 0; ; i++ {
		ch <- i
	}
}

func suck(ch chan int) {
	for {
		fmt.Println(<-ch)
	}
}
```

这里我们定义了一个`suck`函数，作为`接收者`，并给`main`协程一个1s的运行时间，于是，便产生了70W+的输出【TAT】。

### 通道死锁

通道两段互相阻塞对方，会形成死锁状态。Go运行时会检查并panic，停止程序。无缓冲通道会被阻塞。

```Golang
package main

import "fmt"

func main() {
	out := make(chan int)
	out <- 2
	go f1(out)
}

func f1(in chan int) {
	fmt.Println(<-in)
}
```

```
fatal error: all goroutines are asleep - deadlock!
```

显然在`out <- 2`的时候，由于没有接受者，主线程被阻塞。

### 同步通道

除了普通的无缓存通道外，还有一种特殊的带缓存通道——`同步通道`。

```Golang
buf := 100
ch1 := make(chan string, buf)
```

`buf`是通道可以同时容纳的元素个数，即`ch1`的缓冲区大小，在`buf`满之前，通道都不会阻塞。

如果容量大于0，通道就是异步的：在缓冲满载或边控之前通信不会阻塞，元素会按照发送的顺序被接收。

同步：`ch := make(chan type, value)`

+ value ==0 --> synchronous, unbuffered（阻塞）
+ value > 0 --> asynchronous, buffered（非阻塞）取决于value元素

使用通道缓冲能使程序更具有伸缩性（scalable）。

尽量在首要位置使用无缓冲通道，只在不确定的情况下使用缓冲。

```Golang
package main

import "fmt"
import "time"

func main() {
	c := make(chan int, 50)
	go func() {
		time.Sleep(15 * 1e9)
		x := <-c
		fmt.Println("received", x)
	}()
	fmt.Println("sending", 10)
	c <- 10
	fmt.Println("send", 10)
}

```

## 信号量模式

```Golang
func compute(ch chan int) {
    ch <- someComputation()
}

func main() {
    ch := make(chan int)
    go compute(ch)
    doSomethingElaseForAWhile()
    result := <-ch
}
```

协程通过在通道`ch`中放置一个值来处理结束信号。`main`线程等待`<-ch`直到从中获取到值。

我们可以用它来处理切片排序：

```
done := make(chan bool)

doSort := func(s []int) {
    sort(s)
    done <- true
}
i := pivot(s)
go doSort(s[:i])
go doSort(s[i:])
<-done
<-done
```

### 带缓冲通道实现信号量

信号量时实现互斥锁的常用同步机制，限制对资源的访问，解决读写问题。

+ 带缓冲通道的容量要和同步的资源容量相同
+ 通道的长度（当前存放的元素个数）与当前资源被使用的数量相同
+ 容量减去通道的长度等于未处理的资源个数

```Golang
//创建一个长度可变但容量为0的通道
type Empty interface {}
type semaphore chan Empty
```

初始化信号量

```Golang
sem = make(semaphore, N)
```

对信号量进行操作，建立互斥锁

```
func (s semaphore) P (n int) {
    e := new(Empty)
    for i := 0; i < n; i++ {
        s <- e
    }
}

func (a semaphore) V (n int) {
    for i := 0; i < n; i++ {
        <- s
    }
}

/* mutexes */
func (s semaphore) Lock() {
	s.P(1)
}

func (s semaphore) Unlock(){
	s.V(1)
}

/* signal-wait */
func (s semaphore) Wait(n int) {
	s.P(n)
}

func (s semaphore) Signal() {
	s.V(1)
}
```

#### 通道工厂模式

不将通道作为参数传递，而是在函数内生成一个通道，并返回。

```Golang
package main

import (
	"fmt"
	"time"
)

func main() {
	stream := pump()
	go suck(stream)
	time.Sleep(1e9)
}

func pump() chan int {
	ch := make(chan int)
	go func() {
		for i := 0; ; i++ {
			ch <- i
		}
	}()
	return ch
}

func suck(ch chan int) {
	for {
		fmt.Println(<-ch)
	}
}
```

#### 通道使用for循环

`for`循环可以从`ch`中持续获取值，直到通道关闭。（这意味着必须有另一个协程写入`ch`，并且在写入完成后关闭）

```Golang
for v := range ch {
    fmt.Println("The value is", v)
}
```

```Golang
package main

import (
	"fmt"
	"time"
)

func main() {
	suck(pump())
	time.Sleep(1e9)
}

func pump() chan int {
	ch := make(chan int)
	go func() {
		for i := 0; ; i++ {
			ch <- i
		}
	}()
	return ch
}

func suck(ch chan int) {
	go func() {
		for v := range ch {
			fmt.Println(v)
		}
	}()
}
```

### 通道的方向

通道可以表示它只发送或者只接受：

```Golang
var send_only chan<- int    // channel can only send data
var recv_only <-chan int    // channel can only receive data
```

只接收的通道（<-chan T）无法关闭，因为关闭通道是发送者用来表示不再给通道发送值，所以对只接收通道是没有意义的。

#### 管道和选择器模式

借鉴一个经典的例子`筛法求素数`来学习这一内容。

![](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/images/14.2_fig14.2.png?raw=true)

这个算法的主要思想是，引入`筛法`（一种时间复杂度为O(x * ln(lnx))的算法），对一个给定返回的正整数从大到小排序，然后从中筛选掉所有的非素数，那么剩下的数中最小的就是素数，再去掉该数的倍数，以此类推。

假设一个范围为1~30的正整数集，已经从大到小排序。

第一遍筛掉非素数1，然后剩余数中最小的是2。

由于2是一个素数，将其取出，然后去掉所有2的倍数，那么剩下的数为：

`3 5 7 9 11 13 15 17 19 21 23 25 27 29`

剩下的数中3最小，且为素数，取出并去除所有3的倍数，循环直至所有数都筛完。

代码如下：

```Golang
// 一般写法
package main

import (
	"fmt"
)

func generate(ch chan int) {
	for i := 2; i < 100; i++ {
		ch <- i
	}
}

func filter(in, out chan int, prime int) {
	for {
		i := <-in
		if i%prime != 0 {
			out <- i
		}
	}
}

func main() {
	ch := make(chan int)
	go generate(ch)
	for {
		prime := <-ch
		fmt.Print(prime, " ")
		ch1 := make(chan int)
		go filter(ch, ch1, prime)
		ch = ch1
	}
}
```

```Golang
// 习惯写法
package main

import (
	"fmt"
)

func generate() chan int {
	ch := make(chan int)
	go func() {
		for i := 2; ; i++ {
			ch <- i
		}
	}()
	return ch
}

func filter(in chan int, prime int) chan int {
	out := make(chan int)
	go func() {
		for {
			if i := <-in; i%prime != 0 {
				out <- i
			}
		}
	}()
	return out
}

func sieve() chan int {
	out := make(chan int)
	go func() {
		ch := generate()
		for {
			prime := <-ch
			ch = filter(ch, prime)
			out <- prime
		}
	}()
	return out
}

func main() {
	primes := sieve()
	for {
		fmt.Println(<-primes)
	}
}
```







