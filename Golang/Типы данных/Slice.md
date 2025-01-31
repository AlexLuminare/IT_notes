
**Все варианты создания**
```go
package main

import "fmt"

func main() {
    // Создание слайса с использованием литерала
    sliceLiteral := []int{1, 2, 3, 4, 5}
    fmt.Println("Slice literal:", sliceLiteral)

    // Создание слайса с использованием make
    sliceMake := make([]int, 3) // Длина 3, емкость 3
    fmt.Println("Slice make:", sliceMake)

    sliceMakeWithCapacity := make([]int, 3, 5) // Длина 3, емкость 5
    fmt.Println("Slice make with capacity:", sliceMakeWithCapacity)

    // Создание слайса на основе массива
    arr := [5]int{1, 2, 3, 4, 5}
    sliceFromArray := arr[1:4] // Элементы с индексами 1, 2, 3
    fmt.Println("Slice from array:", sliceFromArray)

    // Создание слайса на основе другого слайса
    sliceFromSlice := sliceLiteral[2:5] // Элементы с индексами 2, 3, 4
    fmt.Println("Slice from slice:", sliceFromSlice)

    // Создание пустого слайса
    var emptySlice []int
    fmt.Println("Empty slice:", emptySlice, len(emptySlice), cap(emptySlice))

    // Добавление элемента в пустой слайс
    emptySlice = append(emptySlice, 10)
    fmt.Println("After append:", emptySlice)
}
```