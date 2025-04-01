```go
  // 输出什么?
  package main

  import (
      "fmt"
	  "strings"
  )

  func main() {
	  s := ""
	  fmt.Println(len(strings.Split(s, ",")))
  }

```
输出1
运行结果1
```go
package main

import (
	"fmt"
)

type person struct {
	name string
	m    map[int]string
}

func (p person) new() {
	p.name = "name1"
	p.m[1] = "key1"
}

func main() {
	p := person{
		name: "name0",
		m:    make(map[int]string),
	}
	p.new()
	fmt.Println(p) // 输出什么？
}
```
输出 name0, map[1:key1]
```go
  package main

  import (
      "fmt"
  )

  func main() {
	  // 写出下面代码输出的内容
	  defer_call()
  }

  func defer_call() {
      defer func() { fmt.Println("打印前") }()
      defer func() { fmt.Println("打印中") }()
      defer func() { fmt.Println("打印后") }()

      panic("触发异常")
  }

```
输出：打印后 打印中 打印前 触发异常 原因 defer是先入后出的形式，倒叙执行
```go
  // 以下代码有什么问题,说明原因 
  type student struct {
      Name string
      Age  int
  }

  func pase_student() {
      m := make(map[string]*student)
      stus := []student{
          {Name: "zhou", Age: 24},
          {Name: "li", Age: 23},
          {Name: "wang", Age: 22},
      }
      for _, stu := range stus {
          m[stu.Name] = &stu
      }

  }

```
问题：stu作为变量遍历时候，stu的地址是不会改变的，而map的value是一个指针指向的会是同一块地址，会出现数据覆盖的情况,最终的value都是wang 22
```go
// 下面代码会输出什么,并说明原因
func main() {
      runtime.GOMAXPROCS(1)
      wg := sync.WaitGroup{}
      wg.Add(20)
      for i := 0; i < 10; i++ {
          go func() {
              fmt.Println("A: ", i)
              wg.Done()
          }()
      }
      for i := 0; i < 10; i++ {
          go func(i int) {
              fmt.Println("B: ", i)
              wg.Done()
          }(i)
      }
      wg.Wait()
  }
```
输出结果是 A ：10 循环输出10次，B ：0~9 原因：因为设置了runtime.GOMAXPROCS(1)，只有一个CPU核数意味着只能按序执行，有用第一个for循环时并没有传递参数所以最后使用的都是相同的i值。第二个for循环传入i值，并且按需执行



```go
 //下面代码会输出什么?
type People struct{}

func (p *People) ShowA() {
    fmt.Println("showA")
    p.ShowB()
}
func (p *People) ShowB() {
    fmt.Println("showB")
}

type Teacher struct {
    People
}

func (t *Teacher) ShowB() {
    fmt.Println("teacher showB")
}

func main() {
    t := Teacher{}
    t.ShowA()
}

```
输出 showA showB  ***
```go
 // 下面代码会触发异常吗?请详细说明?
func main() {
    runtime.GOMAXPROCS(1)
    int_chan := make(chan int, 1)
    string_chan := make(chan string, 1)
    int_chan <- 1
    string_chan <- "hello"
    select {
        case value := <-int_chan:
            fmt.Println(value)
        case value := <-string_chan:
            panic(value)
    }
}

```
可能触发异常也可能不触发异常，原因是 select 语句是随机的，可能在第一个case中就返回了，而第二个case中的value是未赋值的，所以会panic。
```go
 // 下面代码输出什么?
  func calc(index string, a, b int) int {
      ret := a + b
      fmt.Println(index, a, b, ret)
      return ret
  }

  func main() {
      a := 1
      b := 2
      defer calc("1", a, calc("10", a, b))
      a = 0
      defer calc("2", a, calc("20", a, b))
      b = 1
  }

```
输出： 10 1 2 3

20 0 2 2

2 0 2 2

1 1 3 4

```go
 // 请写出以下输出内容
  func main() {
      s := make([]int, 5)
      s = append(s, 1, 2, 3)
      fmt.Println(s)
  }

```
输出 0 0 0 0 0 1 2 3 
```go
  // 下面的代码有什么问题?
  type UserAges struct {
      ages map[string]int
      sync.Mutex
  }

  func (ua *UserAges) Add(name string, age int) {
      ua.Lock()
      defer ua.Unlock()
      ua.ages[name] = age
  }

  func (ua *UserAges) Get(name string) int {
      if age, ok := ua.ages[name]; ok {
          return age
      }
      return -1
  }

```
问题：Get方法没加锁  ？？？

```go
  // 下面的迭代会有什么问题?
  func (set *threadSafeSet) Iter() <-chan interface{} {
      ch := make(chan interface{})
      go func() {
          set.RLock()

          for elem := range set.s {
              ch <- elem
          }

          close(ch)
          set.RUnlock()

      }()
      return ch
  }

```
问题：主协程没有等待协程退出，就直接退出了
```go
  // 以下代码能编译过去吗？为什么？
  package main

  import (
      "fmt"
  )

  type People interface {
      Speak(string) string
  }

  type Stduent struct{}

  func (stu *Stduent) Speak(think string) (talk string) {
      if think == "bitch" {
          talk = "You are a good boy"
      } else {
          talk = "hi"
      }
      return
  }

  func main() {
      var peo People = Stduent{}
      think := "bitch"
      fmt.Println(peo.Speak(think))
  }

```
答案：不能编译 

原因：Stduent 的方法 Speak 是一个指针接收者，而在赋值时使用的是 Stduent{}，这是一个值类型，这样var peo People = &Stduent{}就可以

```go
  // 以下代码打印出来什么内容，说出为什么?
  package main

  import (
      "fmt"
  )

  type People interface {
      Show()
  }

  type Student struct{}

  func (stu *Student) Show() {

  }

  func live() People {
      var stu *Student
      return stu
  }

  func main() {
      if live() == nil {
          fmt.Println("AAAAAAA")
      } else {
          fmt.Println("BBBBBBB")
      }
  }

```
输出结果 “BBBBBB”

原因 ：接口实例的具体实现类型（如果没有具体类型，则为 nil）

```go
  // 是否可以编译通过?如果通过,输出什么?
  func main() {
      i := GetValue()

      switch i.(type) {
      case int:
          println("int")
      case string:
          println("string")
      case interface{}:
          println("interface")
      default:
          println("unknown")
      }

  }

  func GetValue() int {
      return 1
  }

```
答案 ：不可以编译通过 ***

原因 :type switch 只能用于 接口类型 的变量，因为 type switch 是用来检查接口的 动态类型 的。而在代码中，i 的类型是 int，它不是接口类型，因此无法使用 type switch

```go
  // 下面函数有什么问题?
  func funcMui(x,y int)(sum int,error){
      return x+y,nil
  }

```
答案：返回参数被命名,第二个也要命名，不然就把第一个命名也取消掉
```go
  // 是否可以编译通过,如果通过,输出什么?
  package main

  func main() {

      println(DeferFunc1(1))
      println(DeferFunc2(1))
      println(DeferFunc3(1))
  }

  func DeferFunc1(i int) (t int) {
      t = i
      defer func() {
          t += 3
      }()
      return t
  }

  func DeferFunc2(i int) int {
      t := i
      defer func() {
          t += 3
      }()
      return t
  }

  func DeferFunc3(i int) (t int) {
      defer func() {
          t += i
      }()
      return 2
  }

```
***输出结果 4 1 3

```go
  // 是否可以编译通过,如果通过,输出什么?
  func main() {
      list := new([]int)
      list = append(list, 1)
      fmt.Println(list)
  }

```
答案：不能编译，new返回的是指针用make就可以了
```go
  // 是否可以编译通过,如果通过,输出什么?
  package main

  import "fmt"

  func main() {
      s1 := []int{1, 2, 3}
      s2 := []int{4, 5}
      s1 = append(s1, s2)
      fmt.Println(s1)
  }

```
答案 不能通过 加三个点 ...
```go
  // 是否可以编译通过,如果通过,输出什么?
  func main() {

      sn1 := struct {
          age  int
          name string
      }{age: 11, name: "qq"}
      sn2 := struct {
          age  int
          name string
      }{age: 11, name: "qq"}

      if sn1 == sn2 {
          fmt.Println("sn1 == sn2")
      }

      sm1 := struct {
          age int
          m   map[string]string
      }{age: 11, m: map[string]string{"a": "1"}}
      sm2 := struct {
          age int
          m   map[string]string
      }{age: 11, m: map[string]string{"a": "1"}}

      if sm1 == sm2 {
          fmt.Println("sm1 == sm2")
      }
  }

```
答案 ： 第一个是可以比较的只要两个结构体每个字段都相同就可以比较，但是第二个不可以比较的原因是map整个结构体不可以比较
```go
  // 是否可以编译通过,如果通过,输出什么?
  func Foo(x interface{}) {
      if x == nil {
          fmt.Println("empty interface")
          return
      }
      fmt.Println("non-empty interface")
  }
  func main() {
      var x *int = nil
      Foo(x)
  }

```
答案：输出 第二个Println 

原因： 整个x变量不是nil，它指向的内存是nil，相当于初始化了
```go
  // 是否可以编译通过,如果通过,输出什么?
  func GetValue(m map[int]string, id int) (string, bool) {
      if _, exist := m[id]; exist {
          return "存在数据", true
      }
      return nil, false
  }
  func main()  {
      intmap:=map[int]string{
          1:"a",
          2:"bb",
          3:"ccc",
      }

      v,err:=GetValue(intmap,3)
      fmt.Println(v,err)
  }

```
答案：不能编译通过 

原因:不存在数据时候的return返回值不应该为nil 
```go
  // 是否可以编译通过,如果通过,输出什么?
  const (
      x = iota
      y
      z = "zz"
      k
      p = iota
  )

  func main()  {
      fmt.Println(x,y,z,k,p)
  }

```
输出：0 1 zz zz 4
```go
  // 编译执行下面代码会出现什么?
  package main
  var(
      size :=1024
      max_size = size*2
  )
  func main()  {
      println(size,max_size)
  }

```
错误，取消:
```go
 // 下面函数有什么问题?
  package main
  const cl  = 100

  var bl    = 123

  func main()  {
      println(&bl,bl)
      println(&cl,cl)
  }

```
常量不能取地址
```go
  // 编译执行下面代码会出现什么?
  package main

  func main()  {

      for i:=0;i<10 ;i++  {
      loop:
          println(i)
      }
      goto loop
  }

```
goto loop 会导致程序跳转到 loop 标签，但 loop 标签位于 for 循环内部，无法直接跳出 for 循环，因此会导致编译错误。
```go
  // 编译执行下面代码会出现什么?
  package main
  import "fmt"

  func main()  {
      type MyInt1 int
      type MyInt2 = int
      var i int =9
      var i1 MyInt1 = i
      var i2 MyInt2 = i
      fmt.Println(i1,i2)
  }

```
编译会报错，原因是 MyInt1 和 int 是不同的类型，无法直接赋值给 MyInt1，即使它们底层类型相同。而 MyInt2 是类型别名，因此可以直接赋值给 int 类型的 i。
```go
  // 编译执行下面代码会出现什么?
  package main
  import "fmt"

  type User struct {
  }
  type MyUser1 User
  type MyUser2 = User
  func (i MyUser1) m1(){
      fmt.Println("MyUser1.m1")
  }
  func (i User) m2(){
      fmt.Println("User.m2")
  }

  func main() {
      var i1 MyUser1
      var i2 MyUser2
      i1.m1()
      i2.m2()
  }

```
输出 ：MyUser1.m1 User.m2

```go
  // 编译执行下面代码会出现什么?
  package main

  import "fmt"

  type T1 struct {
  }
  func (t T1) m1(){
      fmt.Println("T1.m1")
  }
  type T2 = T1
  type MyStruct struct {
      T1
      T2
  }
  func main() {
      my:=MyStruct{}
      my.m1()
  }

```
***
代码会编译失败，错误的原因是 MyStruct 中包含了两个同名字段 T1 和 T2，它们都有方法 m1()。Go 在遇到方法重名时无法确定应该调用哪个 m1() 方法
```go
  // 编译执行下面代码会出现什么?
  package main

  import (
      "errors"
      "fmt"
  )

  var ErrDidNotWork = errors.New("did not work")

  func DoTheThing(reallyDoIt bool) (err error) {
      if reallyDoIt {
          result, err := tryTheThing()
          if err != nil || result != "it worked" {
              err = ErrDidNotWork
          }
      }
      return err
  }

  func tryTheThing() (string,error)  {
      return "",ErrDidNotWork
  }

  func main() {
      fmt.Println(DoTheThing(true))
      fmt.Println(DoTheThing(false))
  }

```

```go
  // 编译执行下面代码会出现什么?
  package main

  func test() []func()  {
      var funs []func()
      for i:=0;i<2 ;i++  {
          funs = append(funs, func() {
              println(&i,i)
          })
      }
      return funs
  }

  func main(){
      funs:=test()
      for _,f:=range funs{
          f()
      }
  }

```

```go
  // 编译执行下面代码会出现什么?
  package main

  func test(x int) (func(),func())  {
      return func() {
          println(x)
          x+=10
      }, func() {
          println(x)
      }
  }

  func main()  {
      a,b:=test(100)
      a()
      b()
  }

```

```go
    // 编译执行下面代码会出现什么?
    package main
    
    import (
        "fmt"
        "reflect"
    )
    
    func main()  {
        defer func() {
            if err:=recover();err!=nil{
                fmt.Println(err)
            }else {
                fmt.Println("fatal")
            }
        }()
    
        defer func() {
            panic("defer panic")
        }()
        panic("panic")
    }

```

```go
  // 执行下面代码会发生什么?
  package main

  import (
      "fmt"
      "time"
  )

  func main() {
      ch := make(chan int, 1000)
      go func() {
          for i := 0; i < 10; i++ {
              ch <- i
          }
      }()
      go func() {
          for {
              a, ok := <-ch
              if !ok {
                  fmt.Println("close")
                  return
              }
              fmt.Println("a: ", a)
          }
      }()
      close(ch)
      fmt.Println("ok")
      time.Sleep(time.Second * 100)
  }

```

```go
  // 执行下面代码会发生什么?
  import "fmt"

  type ConfigOne struct {
      Daemon string
  }

  func (c *ConfigOne) String() string {
      return fmt.Sprintf("print: %v", c)
  }

  func main() {
      c := &ConfigOne{}
      c.String()
  }

```

```go
  // 输出什么?
  package main

  import (
      "fmt"
  )

  func main() {
      fmt.Println(len("你好bj!"))
  }

```

```go
  // 编译并运行如下代码会发生什么?
  package main

  import "fmt"

  type Test struct {
      Name string
  }

  var list map[string]Test

  func main() {

      list = make(map[string]Test)
      name := Test{"xiaoming"}
      list["name"] = name
      list["name"].Name = "Hello"
      fmt.Println(list["name"])
  }

```

```go
  // ABCD中哪一行存在错误？
  type S struct {
  }

  func f(x interface{}) {
  }

  func g(x *interface{}) {
  }

  func main() {
      s := S{}
      p := &s
      f(s) //A
      g(s) //B
      f(p) //C
      g(p) //D

  }

```

```go
  // 编译并运行如下代码会发生什么？
  package main

  import (
      "sync"
  )

  const N = 10

  var wg = &sync.WaitGroup{}

  func main() {

      for i := 0; i < N; i++ {
          go func(i int) {
              wg.Add(1)
              println(i)
              defer wg.Done()
          }(i)
      }
      wg.Wait()
  }

```

```go
    // 以下会输出什么?
    func main() {
        s := make([]int,3,8)
        a := s[:9]
        fmt.Println(s,a)
    }

```

```go
    // 以下会输出什么?
    var f = func(i int) {
        print("x")
    }
    
    func main() {
        f := func(i int) {
            print(i)
            if i > 0 {
                f(i - 1)
            }
        }
        f(10)
    }

```

```go
    // 以下会输出什么?
    chan_n := make(chan bool)
    chan_c := make(chan bool, 1)
    done := make(chan struct{})

    go func() {
        for i := 1; i < 11; i += 2 {
            <-chan_c 
            fmt.Print(i)
            fmt.Print(i + 1)
            chan_n <- true 
        }
    }()

    go func() {
        char_seq := []string{"A","B","C","D","E","F","G","H","I","J","K"}
        for i := 0; i < 10; i += 2 {
            <-chan_n 
            fmt.Print(char_seq[i])
            fmt.Print(char_seq[i+1])
            chan_c <- true 
        }
        done <- struct{}{}
    }()

    chan_c <- true
    <-done

```

```go
   // 以下会输出什么?
   const N = 10

   func ttest() {
       m := make(map[int]int)
       wg := &sync.WaitGroup{}
       wg.Add(N)
       for i:=0;i<N;i++ {
           go func() {
               defer wg.Done()
               m[rand.Int()] = rand.Int()
           }()
       }
       wg.Wait()
       fmt.Println(len(m))
   }

```

```go
   // 以下会输出什么?
   func a() {
       ch1 := make(chan int,1)
       ch1 <- 1
       close(ch1)
       fmt.Println(<-ch1)
   }

   func b() {
       ch2 := make(chan int,1)
       ch2 <- 1
       close(ch2)
       for v := range ch2 {
           fmt.Println(v)
       }
   }

   func c() {
       ch3 := make(chan int,1)
       ch3 <- 1
       close(ch3)
       ch3 <- 2
   }

```

```go
// 以下会输出什么?
var T int64 = a()

func init() {
    fmt.Println("init in main.go")
}

func a() int64 {
    fmt.Println("calling a()")
    return 2
}

func main() {
    fmt.Println("calling main")
}


```

```go
// 下面对add函数调用正确的是（）?
// A. add(1, 2)
// B. add(1, 3, 7)
// C. add([]int{1, 2})
// D. add([]int{1, 3, 7}...)
func add(args ...int) int {
        sum := 0
        for _, arg := range args {
            sum += arg
        }
        return sum
}

```