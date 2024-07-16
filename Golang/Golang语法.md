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



