### Структура
- **Стартовая строка (Status Line):** Содержит версию протокола, код состояния и текстовое описание состояния.
    
    - **Версия HTTP:** (HTTP/1.1, HTTP/2.0, HTTP/3.0)
        
    - **Код состояния (Status Code):** Трехзначное число, указывающее на результат запроса (например, 200 OK, 404 Not Found, 500 Internal Server Error).
        
    - **Причина (Reason Phrase):** Краткое текстовое описание кода состояния.
        
    
    Пример: `HTTP/1.1 200 OK`
    
- **Заголовки (Headers):** Предоставляют дополнительную информацию об ответе, сервере или передаваемых данных.
    
    Примеры:
    
    - `Content-Type: text/html; charset=UTF-8` (тип и кодировка возвращаемого контента)
    - `Content-Length: 1234` (размер тела ответа в байтах)
    - `Server: Apache/2.4.41 (Ubuntu)` (информация о сервере)
    - `Date: Wed, 02 Jul 2025 06:30:00 GMT` (дата и время отправки ответа)
    - `Last-Modified: Tue, 01 Jul 2025 10:00:00 GMT` (дата последнего изменения ресурса)
    - `ETag: "abcdef123456"` (идентификатор версии ресурса для кэширования)
    - `Cache-Control: public, max-age=3600` (инструкции для кэширования)
        
- **Пустая строка:** Разделяет заголовки и тело сообщения.
    
- **Тело сообщения (Message Body) - Опционально:** Содержит запрошенные данные (HTML-страница, изображение, JSON-данные и т.д.).
    
    Пример полного HTTP-ответа:
    ```http
    HTTP/1.1 200 OK 
    Content-Type: text/html; charset=UTF-8 
    Content-Length: 156 
    
    <!DOCTYPE html> 
    <html> 
	    <head> 
		    <title>Пример страницы</title> 
	    </head> 
	    <body> 
		    <h1>Привет, мир!</h1> 
	    </body> 
    </html>
```