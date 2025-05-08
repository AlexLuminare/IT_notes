 ипользуется для разделения одного входного оптока данных на несколько выходных потоков с идентичными данными.


```go
package main

import (
	"fmt"
	"sync"
)

//Реализация  паттерна tee-channel внутри функции Tee()
func Tee(in <-chan int) (_, _ <-chan int) {
	out1 := make(chan int)
	out2 := make(chan int)
	go func() {
		defer close(out1)
		defer close(out2)

		for val := range in {
			var out1, out2 = out1, out2
			for i := 0; i < 2; i++ {
				select {
				case out1 <- val:
					out1 = nil
				case out2 <- val:
					out2 = nil
				}
			}
		}
	}()
	return out1, out2
}

//Генератор данных
func Generator() <-chan int {
	ch := make(chan int)
	go func() {
		for val := range 10 {
			ch <- val
		}
		close(ch)
	}()
	return ch
}

func main() {
	in := Generator()

	out1, out2 := Tee(in)
	wg := &sync.WaitGroup{}

	wg.Add(1)
	go func() {
		defer wg.Done()
		for v := range out1 {
			fmt.Printf("logging...%d\n", v)
		}

	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		for v := range out2 {
			fmt.Printf("writing metrics...%d\n", v)
		}

	}()

	wg.Wait()
}
```