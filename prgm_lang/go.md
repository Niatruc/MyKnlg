* 特性
    * 并发编译: 利用go自身特性实现, 编译速度快. 最小元素是包(1.9开始是函数). 

* `go`命令
    * `run main.go`: 运行`main.go`文件. 
    * `build main.go`: 编译并生成`main`程序. 
    * ``: 

# 语法
    ```go
        package main

        import "fmt"

        func main() {
            var a int // 声明变量
            // 或者
            var a = 1 // 声明变量并赋值
            // 或者
            a := 1 // 声明变量并赋值

            const C = "abc"; // 声明常量
            // 声明多个常量
            const (
                a = "abc"
                b = len(a)
                c = 1
            )
            // 特殊常量iota, 它在const关键字出现时被重置为0. 
            const (
                a = iota
            )

            g, h := 123, "hello" // 多声明赋值

            &a // 取地址

            fmt.Println("hello")

            // 选择
            if a {
            
            }
            
            // 循环
            for a := 0; a < 10; a++ {
                ...
            }
            
            // 自增
            i++;
        }
    ```



# 字符串
```go
"a" + "b"
```