# 1 类型

```go
type 类型名字 底层类型
```

1. 不同的类型，则不能直接进行比较

   ```go
   var c Celsius
   var f Fahrenheit
   fmt.Println(c == 0)          // "true"
   fmt.Println(f >= 0)          // "true"
   fmt.Println(c == f)          // compile error: type mismatch
   fmt.Println(c == Celsius(f)) // "true"!
   
   ```

   

2. 命名类型还可以为该类型的值定义新的行为

   ```go
   func (c Celsius) String() string { return fmt.Sprintf("%g°C", c) }
   
   ```

   

# 2 复合数据类型

## 2.1 数组

1. 简化声明

   ```go
   q := [...]int{1, 2, 3}
   fmt.Printf("%T\n", q) // "[3]int"
   
   ```

2. 自定义索引

```Go
type Currency int

const (
    USD Currency = iota // 美元
    EUR                 // 欧元
    GBP                 // 英镑
    RMB                 // 人民币
)

symbol := [...]string{USD: "$", EUR: "€", GBP: "￡", RMB: "￥"}

fmt.Println(RMB, symbol[RMB]) // "3 ￥"
```

3. 在这种形式的数组字面值形式中，初始化索引的顺序是无关紧要的，而且没用到的索引可以省略，和前面提到的规则一样，未指定初始值的元素将用零值初始化。例如，

```Go
r := [...]int{99: -1}   //定义了一个含有100个元素的数组r，最后一个元素被初始化为-1，其它元素都是用0初始化
```



## 2.2 Slice

1. slice并没有指明序列的长度
   - 这会隐式地创建一个合适大小的数组，然后slice的指针指向底层的数组
2. 和数组不同的是，slice之间不能比较
   - 不过标准库提供了高度优化的bytes.Equal函数来判断两个字节型slice是否相等（[]byte），但是对于其他类型的slice，我们必须自己展开每个元素进行比较



## 2.3 Map

1. 判断key是否存在

   ```go
   if age, ok := ages["bob"]; !ok { /* ... */ }
   ```



## 2.4 struct

1. 匿名成员
   - Go语言有一个特性让我们只声明一个成员对应的数据类型而不指名成员的名字；这类成员就叫匿名成员。匿名成员的数据类型必须是命名的类型或指向一个命名的类型的指针。下面的代码中，Circle和Wheel各自都有一个匿名成员。我们可以说Point类型被嵌入到了Circle结构体，同时Circle类型被嵌入到了Wheel结构体。





# 3 Deffered函数

defer语句中的函数会在return语句更新返回值变量后再执行

1. 使用defer记录函数执行时间

   ```go
   func bigSlowOperation() {
       defer trace("bigSlowOperation")() // don't forget the extra parentheses
       // ...lots of work…
       time.Sleep(10 * time.Second) // simulate slow operation by sleeping
   }
   func trace(msg string) func() {
       start := time.Now()
       log.Printf("enter %s", msg)
       return func() { 
           log.Printf("exit %s (%s)", msg,time.Since(start)) 
       }
   }
   
   ```

2. 我们知道，defer语句中的函数会在return语句更新返回值变量后再执行，又因为在函数中定义的匿名函数可以访问该函数包括返回值变量在内的所有变量，所以，对匿名函数采用defer机制，可以使其观察函数的返回值。

以double函数为例：

```Go
func double(x int) int {
    return x + x
}
```

我们只需要首先命名double的返回值，再增加defer语句，我们就可以在double每次被调用时，输出参数以及返回值。

```Go
func double(x int) (result int) {
    defer func() { fmt.Printf("double(%d) = %d\n", x,result) }()
    return x + x
}
_ = double(4)
// Output:
// "double(4) = 8"
```





# 4 接口

1. 可能的bug

   ```go
   const debug = true
   
   func main() {
       var buf *bytes.Buffer
       if debug {
           buf = new(bytes.Buffer) // enable collection of output
       }
       f(buf) // NOTE: subtly incorrect!
       if debug {
           // ...use buf...
       }
   }
   
   // If out is non-nil, output will be written to it.
   func f(out io.Writer) {
       // ...do something...
       if out != nil {
           out.Write([]byte("done!\n"))
       }
   }
   
   ```

   
