
Пакет: context
### Что это такое?
контексты в Go предоставляет мощный механизм для управления **сигналами отмены (cancellation signals)**, **тайм-аутами (timeouts)**, **дедлайнами (deadlines)** и передачи **данных в рамках запроса (request-scoped values)** через границы API и между горутинами. Это особенно важно в распределенных системах и при работе с сетевыми запросами.

Основные задачи `context`:
- **Отмена (Cancellation):** Позволяет одной горутине сигнализировать другой (или нескольким), что выполняемая работа больше не нужна.
- **Тайм-ауты и Дедлайны:** Автоматическая отмена операции по истечении заданного времени или к определенному моменту времени.
- **Передача значений:** Передача специфичных для запроса данных (например, ID трассировки, информация об аутентификации) вниз по стеку вызовов.

#### Методы контекстов
- `Deadline() (deadline time.Time, ok bool)` -  Возвращает время, когда контекст будет отменен, если оно установлено. Если дедлайн не установлен, возвращает  `ok = false`.
- `Done() <-chan struct{}` - Возвращает канал, который закрывается, когда контекст отменен.  Если контекст никогда не отменяется, возвращает `nil`.
- `Err() error` -  Возвращает ошибку, если контекст отменен. Если контекст не отменен, возвращает `nil`. Ошибка может быть `context.Canceled` или `context.DeadlineExceeded`.
- `Value(key interface{}) interface{}` - Возвращает значение, связанное с ключом `key` в контексте.  Если ключ не найден, возвращает `nil`.

####  Виды контекстов

- **`context.Background()`** - Возвращает пустой контекст, который никогда не отменяется. Возвращает пустой контекст, который никогда не отменяется
- **`context.TODO()`** - Аналогичен `context.Background()`, но используется, когда не ясно, какой контекст использовать.
-  **`context.WithCancel(parent Context)(ctx Context, cancel CancelFunc)`** - Создает новый контекст, который может быть отменен с помощью функции `cancel`. Когда вызывается `cancel`, контекст отменяется, и канал `Done()` закрывается.
- **`context.WithTimeout(parent Context, timeout time.Duration)(Context, CancelFunc)`** - Аналогичен `WithDeadline`, но принимает длительность времени `timeout` вместо конкретного времени.  Контекст отменяется через указанное время.
- **`context.WithDeadline(parent Context, deadline time.Time)(Context, CancelFunc)`**  - Создает контекст, который автоматически отменяется в указанное время `d`. Если время `d` уже прошло, контекст отменяется сразу.
- **`context.WithValue(parent Context, key any, value any)(Context)`** - Создает контекст, который содержит значение `val`, связанное с ключом `key`.  Значения в контексте должны использоваться для передачи данных, которые относятся к запросу, а не для передачи необязательных параметров. 

### Примеры
####  Пример использования контекста WithCancel:
	
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
package main

import (
	"context"
	"fmt"
	"time"
)

func operationWithDeadline(ctx context.Context) {
	n := 0
	for {
		select {
		case <-time.After(500 * time.Millisecond):
			n++
			fmt.Printf("Работаем... %d\n", n)
		case <-ctx.Done():
			fmt.Printf("Операция прервана (дедлайн): %v\n", ctx.Err())
			return
		}
	}
}

func main() {
	parentCtx := context.Background()
	deadlineTime := time.Now().Add(2 * time.Second) // Дедлайн через 2 секунды

	ctx, cancel := context.WithDeadline(parentCtx, deadlineTime)
	defer cancel() // Важно!

	fmt.Printf("Запускаем операцию с дедлайном в %s\n", deadlineTime.Format(time.RFC3339))
	go operationWithDeadline(ctx)

	// Ждем, пока не сработает дедлайн
	<-ctx.Done()
	fmt.Printf("Дедлайн достигнут в main. Причина: %v\n", ctx.Err())
	time.Sleep(100 * time.Millisecond) // Дать горутине время вывести сообщение
}
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

// Определяем пользовательский тип для ключа контекста
type keyType string

const requestIDKey keyType = "requestID"
const userKey keyType = "userID"

func processRequest(ctx context.Context) {
	reqID, ok := ctx.Value(requestIDKey).(string)
	if !ok {
		reqID = "unknown"
	}
	fmt.Printf("Обработка запроса с ID: %s\n", reqID)

	userID, ok := ctx.Value(userKey).(int)
	if ok {
		fmt.Printf("  Пользователь ID: %d\n", userID)
	}

	// Представим, что мы вызываем другую функцию, передавая контекст дальше
	logSomething(ctx, "Начало обработки данных")
}

func logSomething(ctx context.Context, message string) {
	reqID, _ := ctx.Value(requestIDKey).(string) // Мы ожидаем, что он там есть
	fmt.Printf("[LOG ReqID: %s] %s\n", reqID, message)
}

func main() {
	parentCtx := context.Background()

	// Добавляем requestID
	ctxWithReqID := context.WithValue(parentCtx, requestIDKey, "abc-123-xyz")

	// Добавляем userID поверх предыдущего контекста
	ctxWithUser := context.WithValue(ctxWithReqID, userKey, 42)

	processRequest(ctxWithUser)

	fmt.Println("\n--- Другой запрос без userID ---")
	ctxAnotherReq := context.WithValue(parentCtx, requestIDKey, "def-456-uvw")
	processRequest(ctxAnotherReq)
}
```

#### Пример: HTTP сервер
```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"time"
)

func slowAPICall(ctx context.Context) (string, error) {
	// Имитируем вызов к внешнему API, который может занять время
	// или должен быть отменен, если родительский контекст завершился.
	select {
	case <-time.After(5 * time.Second): // Эта операция занимает 5 секунд
		return "Данные от медленного API", nil
	case <-ctx.Done(): // Если контекст запроса отменен (например, клиент ушел или таймаут сервера)
		log.Printf("slowAPICall: контекст отменен: %v", ctx.Err())
		return "", ctx.Err()
	}
}

func handler(w http.ResponseWriter, r *http.Request) {
	ctx := r.Context() // Получаем контекст запроса

	log.Printf("Обработка запроса: %s %s", r.Method, r.URL.Path)

	// Можно создать дочерний контекст с таймаутом для конкретной операции
	// Например, если мы хотим, чтобы наш slowAPICall не длился дольше 3 секунд,
	// даже если общий таймаут сервера больше.
	opCtx, opCancel := context.WithTimeout(ctx, 3*time.Second)
	defer opCancel()

	data, err := slowAPICall(opCtx) // Передаем opCtx
	if err != nil {
		log.Printf("Ошибка при вызове slowAPICall: %v", err)
		if err == context.DeadlineExceeded {
			http.Error(w, "Операция заняла слишком много времени (внутренний таймаут)", http.StatusGatewayTimeout)
		} else if err == context.Canceled { // Может быть из-за r.Context()
			http.Error(w, "Запрос был отменен клиентом", 499) // 499 Client Closed Request
		} else {
			http.Error(w, "Внутренняя ошибка сервера", http.StatusInternalServerError)
		}
		return
	}

	fmt.Fprintln(w, "Успешно получены данные:", data)
	log.Printf("Запрос успешно обработан для: %s %s", r.Method, r.URL.Path)
}

func main() {
	http.HandleFunc("/data", handler)

	server := &http.Server{
		Addr: ":8080",
		// ReadTimeout:  10 * time.Second, // Общий таймаут чтения запроса
		// WriteTimeout: 10 * time.Second, // Общий таймаут записи ответа
		// IdleTimeout:  15 * time.Second, // Таймаут простоя для keep-alive
	}

	log.Println("Сервер запущен на http://localhost:8080/data")
	log.Println("Попробуйте открыть в браузере. slowAPICall занимает 5с, таймаут на него 3с.")
	log.Println("Также попробуйте обновить страницу в браузере до завершения 3с (это отменит предыдущий r.Context())")

	if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
		log.Fatalf("Ошибка запуска сервера: %v", err)
	}
}
```

### Распространение контекста (Propagation)
Ключевой аспект работы с `context` — это его передача (проброс) через вызовы функций.

- Функции, которые могут быть длительными или требуют отмены/таймаута, должны принимать `context.Context` **в качестве первого параметра**.
- Имя параметра обычно `ctx`.
```go
func MyFunction(ctx context.Context, arg1 string, arg2 int) error {
    // ... какая-то подготовка ...

    // Если MyFunction вызывает другую функцию, которая также поддерживает контекст:
    err := AnotherFunction(ctx, "данные") // Передаем тот же или дочерний контекст
    if err != nil {
        return err
    }

    // ... основная работа MyFunction, периодически проверяя ctx.Done() ...
    select {
    case <-ctx.Done():
        return ctx.Err() // Контекст отменен
    default:
        // Продолжаем работу
    }

    // ...
    return nil
}
```

### Лучшие практики и распространенные ошибки
- **Передавайте `Context` как первый аргумент:** `func DoSomething(ctx context.Context, ...)`
- **Не храните `Context` в структурах:** Вместо этого передавайте его явно в методы, которым он нужен.
- **Всегда вызывайте `cancel()` функцию:** Используйте `defer cancel()` сразу после получения `cancel` функции от `WithCancel`, `WithTimeout` или `WithDeadline`, чтобы гарантировать освобождение ресурсов.
- **`context.Background()` только в `main` или на верхнем уровне:** Для инициализации корневого контекста.
- **`context.TODO()` как временная мера:** Когда вы еще не решили, какой контекст использовать, или функция находится в процессе рефакторинга.
- **`context.WithValue()` с осторожностью:** Только для данных, специфичных для запроса, и используйте кастомные типы для ключей. Не для опциональных параметров!
- **Никогда не передавайте `nil` `Context`:** Если не уверены, передайте `context.TODO()`. Однако, функции должны быть готовы к тому, что `Context` может быть `nil`, хотя это считается плохой практикой со стороны вызывающего. Лучше всегда ожидать не-`nil` `Context`.
- **Канал `Done()` только для чтения:** `<-ctx.Done()`.
- **Проверяйте `ctx.Err()` после `ctx.Done()`:** Чтобы понять причину отмены.
- **Контекст распространяется вниз по стеку вызовов.** Отмена родительского контекста отменяет все дочерние.
