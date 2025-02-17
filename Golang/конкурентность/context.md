1. **Виды контекстов:**
     - `context.Background()` — создаёт корневой контекст (используется, если нет других контекстов).
     - `context.TODO()` — используется как временный заглушка, если контекст ещё не определён.
     - `context.WithCancel(parent)` — создаёт новый контекст, который может быть отменён.
    - `context.WithTimeout(parent, timeout)` — задаёт таймаут для выполнения операции.
     - `context.WithDeadline(parent, deadline)` — задаёт фиксированное время завершения.
     - `context.WithValue(parent, key, value)` — добавляет данные в контекст.

2. **Использование каналов:**
    - ctx.Done() — возвращает канал, который закрывается при завершении (дедлайн, таймаут или ctx.Err()) или отмене контекста вручную 
                с использованием соответствующей функции отмены, передающейся в качестве значения при создании контекста.
     - ctx.Err() — возвращает причину завершения (например, context.Canceled или context.DeadlineExceeded).


#### Пример использования контекста WithCancel:
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	go monitor(ctx)

	time.Sleep(2 * time.Second) // Симулируем некоторую работу
	cancel()                    // Отменяем контекст
	time.Sleep(1 * time.Second)
}

func monitor(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("Monitoring stopped:", ctx.Err())
			return
		default:
			fmt.Println("Monitoring in progress...")
			time.Sleep(500 * time.Millisecond)
		}
	}
}
```


#### Пример использования контекста WithDeadline:
```go

```

#### Пример использования контекста WithTimeout:
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	// Создаём контекст с таймаутом 2 секунды
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	// Передаём контекст в функцию
	result := make(chan string)
	go performTask(ctx, result)

	select {
	case res := <-result:
		fmt.Println("Result:", res)
	case <-ctx.Done():
		fmt.Println("Operation timed out:", ctx.Err())
	}
}

func performTask(ctx context.Context, result chan<- string) {
	// Симулируем длительную задачу
	time.Sleep(3 * time.Second)
	select {
	case <-ctx.Done():
		// Если контекст завершён, выходим
		fmt.Println("Task canceled:", ctx.Err())
	default:
		// Отправляем результат
		result <- "Task completed successfully!"
	}
}
```

#### Пример использования контекста WithValue:
```go
package main

import (
	"context"
	"fmt"
)

func main() {
	// Создаём контекст с данными
	ctx := context.WithValue(context.Background(), "userID", 42)

	processRequest(ctx)
}

func processRequest(ctx context.Context) {
	// Извлекаем данные из контекста
	userID := ctx.Value("userID")
	if userID != nil {
		fmt.Printf("Processing request for user ID: %v\n", userID)
	} else {
		fmt.Println("No user ID found in context.")
	}
}
```