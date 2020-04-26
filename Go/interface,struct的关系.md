### 如何理解go的interface,struct？

```
|--------------------------------------------
  Author:       intro                        
  CreateAt:     2018-09-27 23:57:28          
|--------------------------------------------
  
```

#### 如何定义？

 ```
 //结构体定义
 type strs struct{
 
     ID int
     Name string
     
 }
 
 //一个名为 `strs`的structo类型 包含两个元素，首字母大写，是为了可以导出
 
 ```
 ```
 //接口定义
 
 type ints interface{
 
     funcOne(int) int
     funcTwo(string) bool
 }
 
 //定义了一个ints的接口，里面包含两个方法。
 ```

 #### 什么是 struct？

 struct是go语言中的一种数据结构或数据类型,之所以说是数据类型，是因为，可以通过这种类型，生产出一些变量，或是对象出来`var myStruct strs`,表示，声明了一个类型为strs的变量【但些时并未赋值】。此时，可以使用`myStruct.ID`来输出和赋值。哪怕没有赋值，也可以使用，因为会有 **零值** 的存在

    struct除了有一些成员属性外，还有什么？
    
    1、成员属性是可以嵌套（组合）的，即可以包含另一个结构体类型作为它的成员
    
    2、可以为struct 绑定方法
    
     func (s strs) funcOne(i int) int{
         return i*2
     }
     
    对于struct绑定方法的调用，可以操作 myStruct.funcOne(i)
#### 什么是interface？

  interface是go语言的精华所在，go语言，可以理解为` 万物可接口 `，很厉害的角色

  首先，在go语言中，和其它语言一样interface是可以被实现的类型，其次，所有类型都实现了空接口

  务实一点去理解是，接口是定义了一系列方法的结构，没有方法，叫空接口， 可以声明变量为 接口的类型，如：`var s ints `，和struct不一样的是，struct是可以直接去使用里面的元素的，而当使用 `s.funcOne(1)`时，是会编译报错,因为此时的`s`,还只是一个nil，空指针。如果要用里面的方法，必须，为s赋值。

    go语言将会如何判断什么样的变量或类型，可以赋值给一个接口类型的变量呢？

只有**实现了该接口**的类型，才可以进行赋值

实现一个接口，即某struct有绑定过这个接口里的所有方法 ，如 ints里包含 funcOne 和funcTwo两方法
只有当某个struct里同时有绑定过funcOne 和funcTwo方法才可以进行赋值
```
//声明s为ints的类型
var s ints
//将s赋值为一个已经实现过ints的所有方法的类型 strs
 s=strs{
     ID:1,
     Name:"hello"
 }
 
 //s此时调用的funcOne(1)，即调用的是strs中实现的funcOne方法
 s.funcOne(1)
 
 
```
**当然** 由于strs类型,并未绑定funcTwo,所以并不能说strs实现了ints接口,所以s=strs的赋值,会出现**语法错误**

interface类型,必然是依赖于那个自定义的 type 类型结构,如上面代码中,ints的变量,必然是要作用于strs 这个struct的.
当然,一个struct可以实现了多个方法,某些interface的方法在它的struct绑定过的方法中,就说,strcut实现了这个interface,也可以说,一个sturct是可以实现多个接口的
