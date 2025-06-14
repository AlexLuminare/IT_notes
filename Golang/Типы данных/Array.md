
Массивы представляют собой фиксированную последовательность элементов одного типа. Являются основополагающей структурой данных, на базе которой строятся более сложные структуры, такие как слайсы. Рассмотрим, как устроены массивы, их особенности, а также сравнение с другими структурами данных.

#### Основные характеристики 

- Фиксированный размер
Размер массива задается при его объявлении и не может изменяться во время выполнения программы.
- Тип элементов
Все элементы массива имеют один и тот же тип.
- Непосредственное хранение данных
- Размер массива является частью сигнатура типа массива
- массив аллоцируется в стеке, если его размер <= 10 МБ. иначе - в пуле.
- Если массив используется вне блока, в котором он объявлялся - массив аллоцируется в пуле.
- массивы могу аллоцироваться на протяжении работы программы. Следовательно, могут менять свои адреса
#### Объявление массива
- происходит с указанием типа элементов и фиксированного размера. Это объявление создает массив из пяти целых чисел, инициализированных нулями.
```go
var arr [5]int
```
- нельзя передать переменную в качестве размера массива - ошибка компиляции
 ```go
var v_size int = 10
arr [v_size]int    // ОШИБКА!!!!

const c_size = 10
arr [c_size]int    // ОК!!!
```


#### Инициализация массива
```go
arr1 := [5]int{1, 2, 3, 4, 5} //[1 2 3 4 5]
arr2 := [5]int{1, 2}          //[1 2 0 0 0]
arr3 := [...]int{1,2,3,4,5}   //[1 2 3 4 5]
arr4 := [5]int{2:5,6, 1:7}    //[0 7 5 6 0]
```


#### API массивов

```go
arr[0]        //чтение из массива
arr[5]        //запись в массив
len(arr)      // получение длинны массива
cap(arr)      //получение емкости массива
p := &arr     //получение указателя на массив
s := arr[1:3] //получение среза из массива
```

#### Итерирование по массиву
```go
// Способ 1
arr := [...]int{1,2,3,4,5,6}
for i := 0; i < len(arr); i++ {
	fmt.Printf("arr[%d] = %d", i, arr[i])
}

// Способ 2
for ind, val := range arr {
	fmt.Printf("arr[%d] = %d", ind, val)
}
```

####  Копирование массива
При присваивании одного массива другому копируются все элементы:
```go
arr1 := [5]int{1, 2, 3, 4, 5}
arr2 := arr1
arr2[0] = 10
fmt.Println(arr1) // [1 2 3 4 5]
fmt.Println(arr2) // [10 2 3 4 5]
```

#### Передача массива в функции
При этом массив передается по значению - в функцию попадает его копия а не сам массив-аргумент
```go
func modifyArray(a [5]int) {
    a[0] = 10
}

arr := [5]int{1, 2, 3, 4, 5}
modifyArray(arr)
fmt.Println(arr) // [1 2 3 4 5]
```

#### Сравнение с другими массивами (`==`, `!=`)
- Сравнивать массивы можно только с одинаковыми типами (учитываются размер массива + тип хранимых в нем значений)
-  Сравнение происходит поэлементно. Если все соответствующие элементы равны, то массивы считаются равными.
- Нельзя сравнивать массивы при помощи **<, <=, >,>=** - будет ошибка компиляции 
```go
package main

import "fmt"

func main() {
    a := [3]int{1, 2, 3}
    b := [3]int{1, 2, 3}
    c := [3]int{1, 2, 4}

    fmt.Println(a == b)  // Выведет true
    fmt.Println(a == c)  // Выведет false
}
```



#### Предметное сравнение массивов со слайсами

 - **Размер**: Массивы имеют фиксированный размер, тогда как слайсы динамичны.

- **Производительность**: Массивы могут быть более производительными для небольших коллекций данных из-за отсутствия накладных расходов на управление динамическими данными.

- **Гибкость**: Слайсы более гибки благодаря динамическому изменению размера и доступным методам.

```go
package main

import (
    "fmt"
)

func main() {
    // Объявление и инициализация массива
    arr := [5]int{1, 2, 3, 4, 5}

    // Доступ к элементам
    fmt.Println("First element:", arr[0]) // First element: 1

    // Изменение элементов
    arr[1] = 10
    fmt.Println("Modified array:", arr) // Modified array: [1 10 3 4 5]

    // Длина массива
    fmt.Println("Length of array:", len(arr)) // Length of array: 5

    // Копирование массива
    arr2 := arr
    arr2[0] = 20
    fmt.Println("Original array:", arr) // Original array: [1 10 3 4 5]
    fmt.Println("Copied array:", arr2)  // Copied array: [20 10 3 4 5]

    // Передача массива в функцию
    modifyArray(arr)
    fmt.Println("Array after modifyArray call:", arr) // Array after modifyArray call: [1 10 3 4 5]
}

func modifyArray(a [5]int) {
    a[0] = 10
}

```