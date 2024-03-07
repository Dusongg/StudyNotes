## union的构造函数默认删除

![image-20240307144823789](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240307144823789.png)

C++11标准引入了对联合体（union）的新规定，要求联合体中如果含有非平凡（non-trivial）的成员（如自定义的构造函数、拷贝构造函数、析构函数等），则该联合体的默认构造函数会被删除。这是因为联合体的特性决定了只能同时使用一个成员，而非平凡的构造函数可能会导致程序员误解联合体的使用方式，可能会造成不符合预期的行为

```cpp
class test {
public:
    union testunion {
        testunion() {}    //不能用default
        ~testunion() {}
        function<int(int)> f1;
        function<int(int)> f2;
    };

    testunion func;
    test() {}         // 默认构造函数
    ~test() {}        // 析构函数
};

int main() {
    test t1; // 创建一个 test 对象
    return 0;
}
```

