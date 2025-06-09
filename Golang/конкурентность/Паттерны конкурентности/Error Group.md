
### Что это такое?


### Пример реализации Errgroup с применением канала
```go
package main  
  
import (  
    "errors"  
    "fmt"    "math/rand"    "sync"    "time")  
  
type ErrGroup struct {  
    err      error  
    wg       sync.WaitGroup  
    once     sync.Once  
    doneChan chan struct{}  
}  
  
func NewErrGroup() (*ErrGroup, chan struct{}) {  
    doneChan := make(chan struct{})  
    return &ErrGroup{doneChan: doneChan}, doneChan  
}  
  
func (eg *ErrGroup) Go(f func() error) {  
    eg.wg.Add(1)  
    go func() {  
       defer eg.wg.Done()  
  
       select {  
       case <-eg.doneChan:  
          return  
       default:  
          if err := f(); err != nil {  
             eg.once.Do(func() {  
                eg.err = err  
                close(eg.doneChan)  
             })  
          }  
       }  
    }()  
}  
  
func (eg *ErrGroup) Wait() error {  
    eg.wg.Wait()  
    return eg.err  
}  
  
func main() {  
    group, groupDone := NewErrGroup()  
    for i := 0; i < 5; i++ {  
       group.Go(func() error {  
          timeout := time.Second * time.Duration(rand.Intn(10))  
          timer := time.NewTimer(timeout)  
          defer timer.Stop()  
  
          select {  
          case <-groupDone:  
             fmt.Println("canceled")  
             return errors.New("canceled")  
          case <-timer.C:  
             fmt.Println("timeout")  
             return errors.New("timeout")  
          }  
       })  
    }  
    if err := group.Wait(); err != nil {  
       fmt.Println(err.Error())  
    }  
}
```