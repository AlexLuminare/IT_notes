

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

// Определяем структуру с тегами JSON
type Person struct {
	Name    string `json:"name"`
	Age     int    `json:"age"`
	Address string `json:"address"`
}

func main() {
	// Создаем объект структуры
	p := Person{
		Name:    "Alice",
		Age:     30,
		Address: "123 Main St",
	}

	// Сериализация структуры в JSON
	jsonData, err := json.Marshal(p)
	if err != nil {
		log.Fatalf("Ошибка сериализации: %v", err)
	}
	fmt.Println("Сериализованные данные в формате JSON:")
	fmt.Println(string(jsonData))

	// Десериализация JSON в структуру
	var newPerson Person
	err = json.Unmarshal(jsonData, &newPerson)
	if err != nil {
		log.Fatalf("Ошибка десериализации: %v", err)
	}
	fmt.Println("\nДесериализованные данные:")
	fmt.Printf("Имя: %s, Возраст: %d, Адрес: %s\n", newPerson.Name, newPerson.Age, newPerson.Address)
}

```