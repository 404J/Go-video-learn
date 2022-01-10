### 数据结构（面向数据的设计）
代码分为三种：分配内存/读取内存/写入内存

#### 数组
1. 为什么 go 只提供了数组、切片、map 这三种数据结构？
    如果应用程序使用的数据结构是地址连续的、操作系统（缓存系统）可预测的（数据内存连续，存在同一缓存行中），那么可以大大提高程序的运行效率。go 应用程序是直接运行在真机上的，并不像 Java 有虚拟机
2. 构造一个很大的二维数组，按行遍历和按列遍历那个效率高？
    按行遍历效率高，因为前者的内存是连续的，操作系统可预测的
3. 理解数组的内存情况
    ```
    // 1. 指针语义
    func main() {
        people := [5]string{"404", "Leason", "小乾", "Distant", "纸片"}

        for i := range people {
            people[1] = "RNM"
            if i == 1 {
                fmt.Println(people[i])
            }
        }
    }
    // 2. 值语义, for range 中操作的是数组的一份副本
    func main() {
        people := [5]string{"404", "Leason", "小乾", "Distant", "纸片"}

        for i, v := range people {
            people[1] = "RNM"
            if i == 1 {
                fmt.Println(v)
            }
        }
    }
    ```
4. 数组的内存地址是连续的，字符串数组，在 64 位机器上面，每个元素的地址间隔是 16（两个字节）
5. 数组是指类型，分配在栈上的

#### 切片
1. 切片是引用类型，是三个字长的数据，三字长存储的内容分别是：指向支撑数组的指针，长度，容量（类似于字符串，但是字符串不是引用类型，因为字符串的零值为空字符串不是 nil）
2. 切片本来就应该使用值语义操作，切片是一个分配在栈上的三字长数据，如果支撑数组被多个切片引用，该数组就会分配到堆上
3. 理解切片的内存情况
    ```
    // 1. 共用同一个支撑数组，支撑数组的堆上
    func main() {
        arr1 := []string{"1", "2", "3", "4", "5"}
        arr2 := arr1[1:3] // 此方式新的切片和原来切片共用同一个支撑数组
        arr1[1] = "change"
        // 地址相同，这样复制切片只需复制出一个切片值即可，虽高效，但是有副作用
        fmt.Println(&arr1[1])
        fmt.Println(&arr2[0])
        fmt.Println("len: ", len(arr1), ", cap: ", cap(arr1), ", arr1: ", arr1)
        fmt.Println("len: ", len(arr2), ", cap: ", cap(arr2), ", arr1: ", arr2)
    }
    // 2. 不共用
    func main() {
        arr1 := []string{"1", "2", "3", "4", "5"}
        arr2 := make([]string, len(arr1))
        copy(arr2, arr1) // 此方式新的切片和原来切片不共用一个支撑数组
        arr1[1] = "change"
        fmt.Println(&arr1[0])
        fmt.Println(&arr2[0])
        fmt.Println("len: ", len(arr1), ", cap: ", cap(arr1), ", arr1: ", arr1)
        fmt.Println("len: ", len(arr2), ", cap: ", cap(arr2), ", arr1: ", arr2)
    }
    // 3. 慎用 append
    func main() {
        arr1 := make([]user, 5)
        share := &arr1[0]
        share.like++
        arr1 = append(arr1, user{}) // len == cap, 复制数组
        fmt.Println("len: ", len(arr1), ", cap: ", cap(arr1), ", arr1: ", arr1)
        share.like++ // 无效，且引用的原数组无法被垃圾回收
        fmt.Println("len: ", len(arr1), ", cap: ", cap(arr1), ", arr1: ", arr1)
    }

    type user struct {
        like int
    }
    ```
4. 字符串在 go 中是分三层存储的，字节 -> 码点 -> 字符，for range 迭代的时候是一个码点一个码点进行迭代的
    ```
    func main() {
        s := "你好 hello"
        for i, v := range s { // i 的值为0、3、6、7、8...
            fmt.Println(i)
            fmt.Println(string(v))
        }
    }
    ```
5. 切片的值语义和指针语义的 for range 迭代
    ```
    // 1. 值语义
    func main() {
        people := []string{"404", "Leason", "小乾", "Distant", "纸片"}

        for _, v := range people { // 复制出一份切片，支撑数组不变
            people = people[:2] // 改变的是原切片的 len
            fmt.Println(v)      // v 为新切片的迭代
        }
        fmt.Println(people)
    }
    // 2. 指针语义
    func main() {
        people := []string{"404", "Leason", "小乾", "Distant", "纸片"}

        for i := range people {
            people = people[:2]
            fmt.Println(people[i]) // Err
        }
        fmt.Println(people)
    }
    ```