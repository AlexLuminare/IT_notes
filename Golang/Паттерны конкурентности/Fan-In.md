Данные из нескольких входных потоков попадают в один выходной.


```go
package main  
  
import (  
    "context"  
    "fmt"    "sync")  
  
func FanIn(ctx context.Context, chans []chan int) chan int {  
    out := make(chan int)  
    go func() {  
       wg := &sync.WaitGroup{}  
  
       //перебираем массив входных каналов  
       for _, ch := range chans {  
          wg.Add(1)  
  
          go func(ch chan int) {  
             defer wg.Done()  
             for {  
                select {  
                case val, ok := <-ch:  
                   if !ok {  
                      return  
                   }  
                   select {  
                   case out <- val:  
                   case <-ctx.Done():  
                      return  
                   }  
                case <-ctx.Done():  
                   return  
                }  
             }  
          }(ch)  
  
       }  
       wg.Wait()  
       close(out)  
    }()  
    return out  
}  
  
func main() {  
    ch1 := make(chan int)  
    ch2 := make(chan int)  
    chans := make([]chan int, 2)  
  
    chans[0] = ch1  
    chans[1] = ch2  
  
    go func() {  
       ch1 <- 1  
       ch1 <- 1  
       ch1 <- 1  
       ch1 <- 1  
       close(ch1)  
    }()  
  
    go func() {  
       ch2 <- 2  
       ch2 <- 2  
       ch2 <- 2  
       ch2 <- 2  
       close(ch2)  
    }()  
  
    ctx, _ := context.WithCancel(context.Background())  
    out := FanIn(ctx, chans)  
    for val := range out {  
       fmt.Println(val)  
    }  
}
```