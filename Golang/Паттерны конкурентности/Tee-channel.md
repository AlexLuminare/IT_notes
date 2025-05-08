 ипользуется для разделения одного входного оптока данных на несколько выходных потоков с идентичными данными.


### Реализация: Сложно-и-некрасиво 
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
			var out1, out2 = out1, out2 //затемнение переменных, очень осторожно с этой строкой
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

### Реализации красиво-и-попроще
```go
func Tee(in <-chan int) (_, _ <-chan int) {
	out1 := make(chan int)
	out2 := make(chan int)

	wg := &sync.WaitGroup{}

	go func() {
		defer close(out1)
		defer close(out2)

		for val := range in {
			wg.Add(1)
			go func() {
				defer wg.Done()
				out1 <- val
			}()

			wg.Add(1)
			go func() {
				defer wg.Done()
				out2 <- val
			}()
			wg.Wait()
		}
	}()
	return out1, out2
}

//генератор и main() можно взвять из предыдущего примера

```

### Реализация: очень-красиво
```go

package main

import (
	"fmt"
	"sync"
)

func Tee(in <-chan int, num_chans int) []chan int {

	//иниицализация слайса каналов
	chans := make([]chan int, num_chans)

	//инициализыция всех выходных каналов в массиве
	for i := range num_chans {
		chans[i] = make(chan int)
	}

	go func() {
		//отложенно закрываем все созданные каналы
		for i := range num_chans {
			defer close(chans[i])
		}

		//из входного канала считываем значения
		for val := range in {
			wg := &sync.WaitGroup{}

			//в цикле полученное значение пишем во все вызодные каналы
			for i := range num_chans {
				wg.Add(1)
				go func() {
					defer wg.Done()
					chans[i] <- val
				}()
				wg.Wait()
			}
		}

	}()

	return chans
}

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

	chans := Tee(in, 2)
	wg := &sync.WaitGroup{}

	wg.Add(1)
	go func() {
		defer wg.Done()
		for v := range chans[0] {
			fmt.Printf("logging...%d\n", v)
		}

	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		for v := range chans[1] {
			fmt.Printf("writing metrics...%d\n", v)
		}

	}()

	wg.Wait()
}

	
```