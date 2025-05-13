

Паттерн **Worker Pool** используется для ограничения количества одновременно выполняющихся горутин, что позволяет более эффективно управлять ресурсами (например, CPU, память, сетевые соединения) при обработке большого количества задач.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// Task представляет собой задачу для выполнения
type Task struct {
	ID int
}

// Worker принимает задачи из канала и выполняет их
func workerPool(id int, jobs <-chan Task, wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Printf("Рабочий %d запущен.\n", id)
	for job := range jobs {
		fmt.Printf("Рабочий %d обрабатывает задачу %d\n", id, job.ID)
		time.Sleep(time.Millisecond * 500) // Имитация выполнения задачи
	}
	fmt.Printf("Рабочий %d завершил работу.\n", id)
}

func main() {
	numWorkers := 3
	numJobs := 10
	var wg sync.WaitGroup

	jobs := make(chan Task) // Канал для передачи задач рабочим

	// Отправляем задачи в канал 
	go func() {
		for i := 1; i <= numJobs; i++ {
			jobs <- Task{ID: i}
		}
		close(jobs) // Закрываем канал задач, чтобы сигнализировать рабочим об отсутствии новых задач
	}()

	// Запускаем пул рабочих
	for i := 1; i <= numWorkers; i++ {
		wg.Add(1)
		go workerPool(i, jobs, &wg)
	}

	wg.Wait() // Ожидаем завершения всех рабочих

	fmt.Println("Все задачи выполнены.")
}
```