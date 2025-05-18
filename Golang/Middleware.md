Middleware в Go, особенно в контексте веб-разработки с использованием таких фреймворков, как `net/http` или `Gin`, представляет собой функцию, которая располагается между обработчиком запроса (handler) и самим запросом. Она позволяет выполнять определенные действия до или после вызова основного обработчика.

Думай о middleware как о конвейере обработки запроса. Каждый middleware в этом конвейере может выполнять следующие задачи:

- **Логирование:** Записывать информацию о входящих запросах.
- **Аутентификация и авторизация:** Проверять подлинность пользователя и его права доступа.
- **Валидация:** Проверять корректность входящих данных.
- **Изменение запроса или ответа:** Добавлять заголовки, сжимать данные и т.д.
- **Обработка ошибок:** Перехватывать и обрабатывать ошибки, возникшие в последующих обработчиках.

Основная идея middleware заключается в том, чтобы вынести общую логику обработки запросов в отдельные, переиспользуемые компоненты, что делает код более чистым, модульным и легким в поддержке.


#### Классический пример применения middleware
```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

// Middleware для логирования времени выполнения запроса
func loggingMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		startTime := time.Now()
		fmt.Printf("Начало обработки запроса: %s %s\n", r.Method, r.URL.Path)
		next.ServeHTTP(w, r)
		duration := time.Since(startTime)
		fmt.Printf("Завершение обработки запроса: %s %s (длительность: %s)\n", r.Method, r.URL.Path, duration)
	})
}

// Middleware для аутентификации (простой пример)
func authMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Простая проверка наличия заголовка Authorization
		if r.Header.Get("Authorization") == "" {
			http.Error(w, "Требуется авторизация", http.StatusUnauthorized)
			return
		}
		fmt.Println("Пользователь авторизован")
		next.ServeHTTP(w, r)
	})
}

// Обработчик для публичного маршрута
func publicHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Это публичный ресурс")
}

// Обработчик для защищенного маршрута
func protectedHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Это защищенный ресурс")
}

func main() {
	// Создаем новый ServeMux
	mux := http.NewServeMux()

	// Регистрируем обработчик для публичного маршрута без middleware
	mux.HandleFunc("/public", publicHandler)

	// Создаем цепочку middleware для защищенного маршрута
	protectedChain := loggingMiddleware(authMiddleware(http.HandlerFunc(protectedHandler)))

	// Регистрируем обработчик с применением цепочки middleware
	mux.Handle("/protected", protectedChain)

	// Запускаем HTTP-сервер, передавая наш ServeMux
	fmt.Println("Сервер запущен на http://localhost:8080")
	http.ListenAndServe(":8080", mux)
}
```


### Подход с применением цепочек
Более красивый метод

```go
package main  
  
import (  
    "fmt"  
    "net/http"    
    "slices"    
    "time")  
  
type chain []func(http.Handler) http.Handler  
  
func (c chain) thenFunc(h http.HandlerFunc) http.Handler {  
    return c.then(h)  
}  
  
func (c chain) then(h http.Handler) http.Handler {  
    for _, mw := range slices.Backward(c) {  
       h = mw(h)  
    }  
    return h  
}  
  
// Middleware для логирования времени выполнения запроса  
func loggingMiddleware(next http.Handler) http.Handler {  
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {  
       startTime := time.Now()  
       fmt.Printf("Начало обработки запроса: %s %s\n", r.Method, r.URL.Path)  
       next.ServeHTTP(w, r)  
       duration := time.Since(startTime)  
       fmt.Printf("Завершение обработки запроса: %s %s (длительность: %s)\n", r.Method, r.URL.Path, duration)  
    })  
}  
  
// Middleware для аутентификации (простой пример)  
func authMiddleware(next http.Handler) http.Handler {  
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {  
       // Простая проверка наличия заголовка Authorization  
       if r.Header.Get("Authorization") == "" {  
          http.Error(w, "Требуется авторизация", http.StatusUnauthorized)  
          return  
       }  
       fmt.Println("Пользователь авторизован")  
       next.ServeHTTP(w, r)  
    })  
}  
  
// Обработчик для публичного маршрута  
func publicHandler(w http.ResponseWriter, r *http.Request) {  
    fmt.Fprintf(w, "Это публичный ресурс")  
}  
  
// Обработчик для защищенного маршрута  
func protectedHandler(w http.ResponseWriter, r *http.Request) {  
    fmt.Fprintf(w, "Это защищенный ресурс")  
}  
  
func main() {  
    // Создаем новый ServeMux  
    mux := http.NewServeMux()  
  
    // Создаем цепочку middleware для публичного маршрута  
    baseChain := chain{loggingMiddleware}  
    // Создаем цепочку middleware для защищенного маршрута  
    authChain := chain{loggingMiddleware, authMiddleware}  
  
    //// Регистрируем обработчик для публичного маршрута без middleware  
    mux.Handle("/public", baseChain.thenFunc(publicHandler))  
    // Регистрируем обработчик с применением цепочки middleware  
    mux.Handle("/protected", authChain.thenFunc(protectedHandler))  
  
    // Запускаем HTTP-сервер, передавая наш ServeMux  
    fmt.Println("Сервер запущен на http://localhost:8888")  
    http.ListenAndServe(":8888", mux)  
}
```