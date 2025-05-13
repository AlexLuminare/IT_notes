
Ограничитель скорости контролирует количество событий, которые могут произойти за определенный период времени. Это полезно для предотвращения перегрузки системы, защиты от злоупотреблений (например, DoS-атак) или соблюдения ограничений внешних API.

### Пример реализации (с использованием `time.Ticker`):

```go
package main

import (
	"fmt"
	"time"
)

func processRequest(id int) {
	fmt.Printf("Обработка запроса %d в %s\n", id, time.Now().Format(time.RFC3339))
	time.Sleep(time.Millisecond * 200) // Имитация обработки
}

func main() {
	rate := time.Second / 2 // Разрешаем 2 запроса в секунду
	limiter := time.Tick(rate)

	for i := 1; i <= 5; i++ {
		<-limiter // Блокируемся до тех пор, пока не пройдет разрешенное время
		processRequest(i)
	}
}
```
