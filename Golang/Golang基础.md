# Golang

## 实现并发的工作池

```go
package main

import (
    "fmt"
    "time"
)

func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Println("worker", id, "started  job", j)
        time.Sleep(time.Second)
        fmt.Println("worker", id, "finished job", j)
        results <- j * 2
    }
}

func main() {

    const numJobs = 5
    jobs := make(chan int, numJobs)
    results := make(chan int, numJobs)

    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }

    for j := 1; j <= numJobs; j++ {
        jobs <- j
    }
    close(jobs)
	//阻塞接收以避免主程序结束
    for a := 1; a <= numJobs; a++ {
        <-results
    }
}
```

## 使用WaitGroup来等待多协程结束

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int) {
    fmt.Printf("Worker %d starting\n", id)

    time.Sleep(time.Second)
    fmt.Printf("Worker %d done\n", id)
}

func main() {

    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)
//避免在每个协程闭包中重复利用相同的 i 值
        i := i
//将 worker 调用包装在一个闭包中，可以确保通知 WaitGroup 此工作线程已完成。 这样，worker 线程本身就不必知道其执行中涉及的并发原语。
        go func() {
            defer wg.Done()
            worker(i)
        }()
    }
//阻塞，直到 WaitGroup 计数器恢复为 0； 即所有协程的工作都已经完成。
    wg.Wait()

}
```

## 排序

### 使用sort包

```go
package main

import (
    "fmt"
    "sort"
)

func main() {

    strs := []string{"c", "a", "b"}
    sort.Strings(strs)
    fmt.Println("Strings:", strs)

    ints := []int{7, 2, 4}
    sort.Ints(ints)
    fmt.Println("Ints:   ", ints)

    s := sort.IntsAreSorted(ints)
    fmt.Println("Sorted: ", s)
}
```

### 使用函数自定义排序

创建一个自定义类型， 为它实现 `sort.Interface` 接口的 `Len`、`Less` 和 `Swap` 方法， 然后对这个自定义类型的切片调用 `sort.Sort` 方法， 我们就可以通过任意函数对 Go 切片进行排序了。

```go
package main

import (
    "fmt"
    "sort"
)
//string[]类型的别名
type byLength []string

func (s byLength) Len() int {
    return len(s)
}
func (s byLength) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}
func (s byLength) Less(i, j int) bool {
    return len(s[i]) < len(s[j])
}

func main() {
    fruits := []string{"peach", "banana", "kiwi"}
    sort.Sort(byLength(fruits))
    fmt.Println(fruits)
}
```



