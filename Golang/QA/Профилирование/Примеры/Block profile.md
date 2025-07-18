### Шаг 1: Подготовка кода и включение профилирования

В отличие от CPU-профиля, профилирование блокировок **не включено по умолчанию**. Его нужно активировать вручную в коде вашего приложения с помощью функции `runtime.SetBlockProfileRate`.

Эта функция принимает один аргумент `rate`.

- `rate = 1`: Сообщать о каждой блокировке, которая длилась дольше 1 наносекунды. **Идеально для детального анализа.**
- `rate = 10000`: Сообщать о каждой блокировке, длившейся дольше 10,000 наносекунд (10 микросекунд).
- `rate <= 0`: Отключить профилирование.
    

#### **Пример кода**

Создадим веб-сервер с обработчиком, который имитирует медленную операцию (например, ожидание ответа от базы данных или другого сервиса) с помощью `time.Sleep`.

**`main.go`**


```go
package main

import (
	"log"
	"net/http"
	_ "net/http/pprof" // Импортируем для pprof-обработчиков
	"runtime"
	"time"
)

func blockingHandler(w http.ResponseWriter, r *http.Request) {
	log.Println("Handler started. Waiting for a resource...")

	// Имитация долгой операции ввода-вывода или ожидания ресурса
	time.Sleep(2 * time.Second)

	log.Println("Resource received. Handler finished.")
	w.Write([]byte("Blocking operation finished!"))
}

func main() {
	// ВАЖНО: Включаем профилирование блокировок
	// Устанавливаем rate=1, чтобы отслеживать все блокировки.
	runtime.SetBlockProfileRate(1)

	http.HandleFunc("/block", blockingHandler)

	log.Println("Starting server on :8080")
	log.Println("Pprof endpoints available at http://localhost:8080/debug/pprof/")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

### Шаг 2: Сбор профиля

Процесс сбора очень похож на сбор CPU-профиля.
1. **Запустите приложение:**
    
```bash
    go run main.go
```
    
2. **Создайте нагрузку:** Пока сервер работает, в другом терминале отправьте несколько запросов к вашему "медлящему" обработчику. Это создаст события блокировок.
        
    ``` bash
    curl http://localhost:8080/block &
    curl http://localhost:8080/block &
    curl http://localhost:8080/block
    ```
    
    (Символ `&` запускает команду в фоновом режиме, позволяя сразу отправить следующую).
    
3. **Подключитесь к эндпоинту профиля блокировок:** Используйте `go tool pprof` и укажите URL эндпоинта `/debug/pprof/block`.
    
    Bash
    
    ```
    go tool pprof http://localhost:8080/debug/pprof/block
    ```
    
    Вы попадете в интерактивную консоль `pprof`.
    

---

### **Шаг 3: Анализ профиля**

В консоли `pprof` вы можете использовать те же команды, что и для других профилей, но их вывод будет относиться ко времени ожидания.

#### **Команда `top`**

Показывает топ функций, в которых горутины провели больше всего времени в состоянии блокировки.

```bash
(pprof) top
Showing nodes accounting for 6.02s, 100% of 6.02s total
      flat  flat%   sum%        cum   cum%
     6.02s   100%   100%      6.02s   100%  time.Sleep
         0     0%   100%      6.02s   100%  main.blockingHandler
         0     0%   100%      6.02s   100%  net/http.(*ServeMux).ServeHTTP
         0     0%   100%      6.02s   100%  net/http.serverHandler.ServeHTTP
```

- **Анализ:** Мы видим, что почти 100% времени ожидания (`6.02s`) приходится на функцию `time.Sleep`. Это именно то, что мы и ожидали. В реальном приложении здесь могли бы быть функции для работы с каналами, сетью или файлами.
    

#### **Команда `list`**

Позволяет посмотреть на исходный код функции и увидеть, какая именно строка вызывает блокировку.

```bash
(pprof) list blockingHandler
Total: 6.02s
ROUTINE ======================== main.blockingHandler in /path/to/project/main.go
         0      6.02s (flat, cum)   100% of Total
         .          .     12: func blockingHandler(w http.ResponseWriter, r *http.Request) {
         .          .     13: 	log.Println("Handler started. Waiting for a resource...")
         .          .     14:
         .          .     15: 	// Имитация долгой операции ввода-вывода или ожидания ресурса
         .      6.02s     16: 	time.Sleep(2 * time.Second)
         .          .     17:
         .          .     18: 	log.Println("Resource received. Handler finished.")
         .          .     19: 	w.Write([]byte("Blocking operation finished!"))
         .          .     20: }
```

- **Анализ:** `pprof` точно указывает на строку `time.Sleep(2 * time.Second)` как на источник всего времени ожидания в этой функции.
    

#### **Команда `web`**

Генерирует визуальный граф вызовов (требуется установленный Graphviz). Чем "тяжелее" узел, тем больше времени было потрачено на ожидание в этой функции или в функциях, которые она вызывала.

- **Анализ:** Граф наглядно показывает, что путь вызова `http.ListenAndServe` -> `ServeMux` -> `blockingHandler` -> `time.Sleep` является основным источником задержек в приложении.
    

### **Вывод**

Профиль блокировок — незаменимый инструмент, когда ваше приложение "тормозит", но процессор при этом не загружен. Он помогает найти узкие места, связанные с:

- **Медленными операциями ввода-вывода** (диск, сеть).
- **Ожиданием ответов** от баз данных, API или других сервисов.
- **Неэффективной работой с каналами** и другими примитивами синхронизации (`sync.Mutex`, `sync.WaitGroup`).