`unsafe.Pointer` - может быть получен из обычного указателя

```go
	x := 123
	p := unsafe.Pointer(&x)
```


`unsafe.Pointer` - можно преобразовать в обычный указатель

```go
	newPtr := (*int)(p)
```

`unsafe.Pointer` - можно преобразовать в любой \*ТИП
```go
	x:= uint(23423432)
	y := *(*int)(unsafe.Pointer(&x)) 
```

`unsafe.Sizeof()` - получить размер переменной в байтах 
```go
	v := 111.555
	fmt.Printf("%d", unsafe.Sizeof(v))
```

`unsafe.Alignof()` - указывает по какому числу байт идет выравнивание структуры (выравнивание проводится по самому длинному полю структуры)
```go
type MyStruct struct {
	Field1 byte
	Field2 int64
	Field3 byte
	Field4 byte
}

s := MyStruct{}
fmt.Printf("%d", unsafe.Alignof(s)) //8 байт (по полю Field2)
```


`unsafe.Offsetof()` - показывает смещение в байтах поля структуры относительно начального байта:
```go
type MyStruct struct {
	Field1 byte    // 0 байт смещение
	Field2 int64   //8 + 0 байт смещение
	Field3 byte    //8 + 8 = 16 байт смещение
	Field4 byte    // 16 + 1 = 17 байт смещение 
}

s := MyStruct{}
fmt.Printf("%d", unsafe.Offsetof(s.Field3)) // 16 байт 
fmt.Printf("%d", unsafe.Offsetof(s.Field4)) // 17 байт
```

Зная размер смещений мы можем получать определенные  значения, используя адресную арифметику:
```go
	arr := [4]byte{0x00, 0x01, 0x02, 0x03}
	arr_0 := (*byte)(unsafe.Pointer(uintptr(unsafe.Pointer(&arr)) + 0*unsafe.Sizeof(arr[0])))
	arr_1 := (*byte)(unsafe.Pointer(uintptr(unsafe.Pointer(&arr)) + 1*unsafe.Sizeof(arr[0])))
	arr_2 := (*byte)(unsafe.Pointer(uintptr(unsafe.Pointer(&arr)) + 2*unsafe.Sizeof(arr[0])))
	arr_3 := (*byte)(unsafe.Pointer(uintptr(unsafe.Pointer(&arr)) + 3*unsafe.Sizeof(arr[0])))

	fmt.Printf("arr_0 = %d\n", *arr_0)
	fmt.Printf("arr_1 = %d\n", *arr_1)
	fmt.Printf("arr_2 = %d\n", *arr_2)
	fmt.Printf("arr_3 = %d\n", *arr_3)
```

	


```go
package main

import (
	"fmt"
	"unsafe"
)

type MyStruct1 struct {
	Field1 byte
	Field2 int64
	Field3 byte
	Field4 byte
}

type MyStruct2 struct {
	Field1 int64
	Field2 byte
	Field3 byte
	Field4 byte
}

func main() {
	n := -123
	fmt.Printf("var n: Type = %[1]T,  value = %[1]v\n", n)
	fmt.Printf("var &n: Type = %[1]T,  value = %[1]v\n", &n)

	//получение unsafe.Pointer
	p := unsafe.Pointer(&n)
	fmt.Printf("unsafe ptr: Type = %[1]T,  value = %[1]v\n", p)

	//десятичное значение указателя
	p2 := uintptr(p)
	fmt.Printf("uintptr : Type = %[1]T,  value = %[1]v\n", p2)

	//новое число после преобразования unsafe.Pointer
	n2 := *(*uint)(p)
	fmt.Printf("var n2: Type = %[1]T,  value = %[1]v\n", n2)

	arr1 := [4]byte{}
	fmt.Printf("var arr1: Type = %[1]T,  value = %[1]v\n", arr1)
	fmt.Printf("Size arr1: %d bytes\n", unsafe.Sizeof(arr1))

	arr2 := []byte{0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06}
	fmt.Printf("var arr2: Type = %[1]T,  value = %[1]v\n", arr2)
	fmt.Printf("Size arr2: %d bytes\n", unsafe.Sizeof(arr2))

	s1 := MyStruct1{}
	fmt.Printf("Sizeof s1 = %d  alignof = %d \n", unsafe.Sizeof(s1), unsafe.Alignof(s1))
	fmt.Printf("offset of s1.Field1 = %d\n", unsafe.Offsetof(s1.Field1))
	fmt.Printf("offset of s1.Field2 = %d\n", unsafe.Offsetof(s1.Field2))
	fmt.Printf("offset of s1.Field3 = %d\n", unsafe.Offsetof(s1.Field3))
	fmt.Printf("offset of s1.Field4 = %d\n", unsafe.Offsetof(s1.Field4))

	s2 := MyStruct2{}
	fmt.Printf("Sizeof s2 = %d  alignof = %d\n", unsafe.Sizeof(s2), unsafe.Alignof(s2))
	fmt.Printf("offset of s2.Field1 = %d\n", unsafe.Offsetof(s2.Field1))
	fmt.Printf("offset of s2.Field2 = %d\n", unsafe.Offsetof(s2.Field2))
	fmt.Printf("offset of s2.Field3 = %d\n", unsafe.Offsetof(s2.Field3))
	fmt.Printf("offset of s2.Field4 = %d\n", unsafe.Offsetof(s2.Field4))
}
```


#### 