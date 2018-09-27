### 读一段代码来理解chan和goroutine是如何配合使用的
```
|--------------------------------------------
  Author:       intro                        
  CreateAt:     2018-09-27 22:43:51          
|--------------------------------------------
  
```
分析一段简单的代码:
```
package main

import (
	"fmt"
	"time"
	"strconv"
)

type Person struct {
	name string
	age uint8
	address Address
}

type Address struct {
	city string
	district string
}

func SendMessage(person *Person, channel chan Person){
	go func(person *Person, channel chan Person) {
		fmt.Printf("%s send a message.\n", person.name)
		channel<-*person
		for i := 0; i < 5; i++ {
			person.name+=strconv.Itoa(i)
			fmt.Println("i=",i)
			channel<- *person
		}
		close(channel)
		fmt.Println("channel is closed.")
	}(person, channel)
}

func main() {
	channel := make(chan Person,1)
	harry := Person{
		"Harry",
		30,
		Address{"London","Oxford"},
	}
	 go SendMessage(&harry, channel)
	data := <-channel


	fmt.Printf("main goroutine receive a message from %s.\n", data.name)
	for {
		i, ok := <-channel
		time.Sleep(time.Second)
		if !ok {
			fmt.Println("channel is empty.")
			break
		}else{
			fmt.Printf("receive %s\n",i.name)
		}
	}
}
```
输出结果:

    Harry send a message.
    i= 0
    i= 1
    main goroutine receive a message from Harry.
    i= 2
    receive Harry0
    i= 3
    receive Harry01
    i= 4
    receive Harry012
    channel is closed.
    receive Harry0123
    receive Harry01234
    channel is empty.
 
 main方法中创建了一个chan,存储为Person类型的内容的变量,channel是带缓存的通道,其中 **带缓存**和  **不带缓存** 的区别在于,`带缓存`可以一次写入缓存长度个值,然后再阻塞至读出.相反的`不带缓存`通道指的是,只要往里面写入内容,就会阻塞,直到读出 
 
 程序的运行流程是:
 
*1*>    创建channel
   
*2*>    创建harray对象
   
*3*>    启动一个goroutine去准备执行SendMessage,注意,只是准备执行而并没有执行.因为

     go语言的goroutine或子线程调度是非抢占式的,
    意味着一个goroutine会一直执行,直到`[执行完成 \ 有io(包括fmt.Println这样的输出) \ 程序阻塞  \ 运行时间过长 ]` 才会交出执行权
    main程序,或是主线程,也可以理解为一个goroutine,所以此时的SendMessage并不会执行
    
*4*>    从channel中读出数据,如果此时channel中并无数据时,会进行阻塞,所以,这个时候执行SendMessage的goroutine拿到执行权,赶紧执行.

*5*>    从`SendMessage`方法中又创建了一个匿名的立即执行的子线程,并且,将SendMessage方法的形参作为它的实参进行传入后,此时SendMessage其实已经执行完了,可以交出控制权,返回到main方法中,而此时main方法还在阻塞中,main方法也让出控制权,所以控制权来到了执行匿名方法的goroutine上.于是,进行了程序的第一个输出:`Harray send a message`,

*6*>    为channel写入第一个 person.neme为`Harray`的值,此时,因为执行权仍未交出,所以并不会切换到main方法的读取部分来.程序继续往下走去进入for循环中

*7*>    将person的name和循环变量拼接起来:person.name的值为`Harray0`,进行了第二个输出:`i=0`

*8*>    将刚改过name的person对象写入channel中,再i++,之所以能写,是因为main中的读出部分,先已经挂起在等待了,所以person.name的值为`Harray0`的对象也能被成功写入

*9*>    重复执行第7步,只是此时的person.name的值为`Harray01`,接着有了第三个输出:`i=1`

*10*> 再试着将person.name的值为`Harray01`的对象写入channel中时,由于channel中已经有了一个person.name的值为`Harray0`的对象存在,缓存已经满了.于是,程序阻塞直到有读出.此时,交出执行权,此时main方法又拿到了执行权

*11*>   由于当前的channel已经有了内容,所以`	data := <-channel` 将不再继续阻塞,读出内容给data后,data里的person.name为`Harray`,所以下一句进行了输出: `main goroutine receive a message from Harry.`

*12*>   进入main方法里的for循环中,读到一个person.name的值为`Harray0`的内容后

*13*>   time.sleep,会导至进程交出执行权,所以此时,执行权又回到了第10步中,被阻塞起来的goroutine上.所以,将之前未写入的person.name的值为`Harray01`的对象写入channel,然后,i++,i=2,

*14*>   接着person.name的值改为`Harray012`,然后进行了输出:`i=2`,

*15*>   再试着写时channel时,又阻塞,让出执行权给main

*16*>   main中经过if判断,输出了`receive Harry0`,之前通道里拿到的值只是Harry0,再读入person,此时persion.name为`Harray01`,遇time.sleep,执行权又回到第15步.

*17*>   做着和第14步一样的事,只是 i+1后,内容有所改变

*18*>   交替输出,直到`SendMessage`中子进程里i=5时,跳出循环,此时
  persion.name的值为`Harray01234`了,而被送进main方法里等待输出的person.nmae的值已经为`receive Harry0123`
  
*19*>   close(channel),通道关闭,意味着,不能再往里面写东西,但是,不影响继续从channel通道中读出内容,输出:`"channel is closed."`后,子线程结束,让出执行权等待GC
*20*> 输出channel中person.nmae的值已经为`receive Harry0123`的内容:` receive Harry0123`,再sleep,这个时候的子线程已经全部结束,也就不存在再把执行权交出来的可能了.再输出 `receive Harry01234`

*21*>   再尝试着从通道里读出内容,由于通道已经空了,所以返回`false`,再经if判断后输出`"channel is empty."`,遇break,跳出循环,程序结束.



### 总结
    1.goroutine的执行与否,往往由执行权决定的.所以多个goroutine由channel控制进行并发时,并不一定就是按预期的顺序在执行.
    2.chan的读写,会产生阻塞而导出交出当前线程的执行权
    


