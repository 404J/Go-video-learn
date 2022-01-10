### 方法，数据的行为表现，是函数写法的一种语法糖
1. 数据上面的方法应该采取的语义
    - 内置类型应该采用值语义
    - 引用类型也应该采用值语义，但是解码和拆分的时候列外
    - 结构体类型需要根据数据类型进行判断
2. 方法解耦的代价
    ```
    func main() {
        u := user{
            name: "404",
        }
        // f1 是一个指针类型的数据，指向的数据也为指针类型，最终指向 displayName 的代码
        f1 := u.displayName // 操作的是 user 数据副本
        u.name = "hello"
        f1()

        // f1 是一个指针类型的数据，指向的数据也为指针类型，最终指向 displayName 的代码和 user 数据
        f2 := u.setName // 操作时共享的 user
        f2()            // 编译器无法对二次中转（多重指针）的 user 判断是否可以存在于栈中，最终分配到堆中
        fmt.Println(u.name)
    }

    type user struct {
        name string
    }

    // 值语义
    func (u user) displayName() {
        fmt.Println(u.name)
    }

    // 指针语义
    func (u *user) setName() {
        u.name = "world"
    }
    ```
### 接口
1. 以下接口设计会引发堆分配
    ```
    type reader interface {
        read(i int) ([]byte, err) // 编译器无法判断 []byte 切片的大小，[]byte 的支撑数组会分配到堆上
    }
    ```
2. 接口实现多态可以实现不同具体数据的解耦，但是代价就是通过指针中转触发内存分配（因为编译器静态代码分析的弱点）
3. 共有的行为通过接口定义，接口的名称不应该是一个具体的事物，应该为行为的描述比如：Reader/Writer
4. 可以通过接口进行解耦，但是，比如只有一种方式实现的接口，这个接口的存在就没有意义，避免接口污染
> 多态就是一段代码可以根据具体的数据表现出不同的行为