обеспечивает баланс между производительностью и потреблением ресурсов. Он идеален для сценариев с большим количеством задач, где важно контролировать нагрузку на систему.

```go
// You can edit this code!  
// Click here and start typing.  
package main  
  
import (  
    "fmt"  
    "sync")  
  
func Pool(in chan int, numWorkers int, f func(int) int) chan int {  
    out := make(chan int)  
    wg := &sync.WaitGroup{}  
    wg.Add(1)  
    go func() {  
       defer wg.Done()  
       for range numWorkers {  
          go worker(in, out, f)  
       }  
    }()  
    wg.Wait()  
    return out  
}  
  
func worker(in, out chan int, f func(int) int) {  
    for v := range in {  
       out <- f(v)  
    }  
    close(out)  
}  
  
func sqr(v int) int {  
    return v * v  
}  
  
func main() {  
    in := make(chan int)  
  
    go func() {  
       for i := range 100 {  
          in <- i  
       }  
       close(in)  
    }()  
  
    out := Pool(in, 10, sqr)  
    for v := range out {  
       fmt.Println(v)  
    }  
}
```