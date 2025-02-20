#### Методы контекстов:
- `Deadline() (deadline time.Time, ok bool)` -  Возвращает время, когда контекст будет отменен, если оно установлено. Если дедлайн не установлен, возвращает  `ok = false`.
- `Done() <-chan struct{}` - Возвращает канал, который закрывается, когда контекст отменен.  Если контекст никогда не отменяется, возвращает `nil`.
- `Err() error` -  Возвращает ошибку, если контекст отменен. Если контекст не отменен, возвращает `nil`. Ошибка может быть `context.Canceled` или `context.DeadlineExceeded`.
- `Value(key interface{}) interface{}` - Возвращает значение, связанное с ключом `key` в контексте.  Если ключ не найден, возвращает `nil`.
- 
### Виды контекстов:

- `context.Background()` - Возвращает пустой контекст, который никогда не отменяется. Возвращает пустой контекст, который никогда не отменяется
- `context.TODO()` - Аналогичен `context.Background()`, но используется, когда не ясно, какой контекст использовать.
-  `context.WithCancel(parent)` - Создает новый контекст, который может быть отменен с помощью функции `cancel`. Когда вызывается `cancel`, контекст отменяется, и канал `Done()` закрывается.
- `context.WithTimeout(parent, timeout)` - Аналогичен `WithDeadline`, но принимает длительность времени `timeout` вместо конкретного времени.  Контекст отменяется через указанное время.
- `context.WithDeadline(parent, deadline)`  - Создает контекст, который автоматически отменяется в указанное время `d`. Если время `d` уже прошло, контекст отменяется сразу.
- `context.WithValue(parent, key, value)` - Создает контекст, который содержит значение `val`, связанное с ключом `key`.  Значения в контексте должны использоваться для передачи данных, которые относятся к запросу, а не для передачи необязательных параметров.

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