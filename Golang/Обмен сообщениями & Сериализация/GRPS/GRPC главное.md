
### ИНСТРУМЕНТЫ  ДЛЯ РАБОТЫ С gRPC
- **protoc** - компилятор, который генерирует код на нужном нам ЯП на основе ваших `.proto` файлов и параметров компиляции.
- **Go gRPC плагины для protoc** - генерируют Go-код для gRPC.
	- **protoc-gen-go** - 
	- **protoc-gen-go-grpc** - 
- **.proto** - прото-файл, 



### УСТАНОВКА КОМПОНЕНТОВ
#### Protoc
- [https://github.com/protocolbuffers/protobuf/releases](https://github.com/protocolbuffers/protobuf/releases)- скачать нужный релиз софтины
- распаковать архив в любое удобное место на компе
-  В системной переменной `PATH` установить абсолютный путь к директории /bin протобафа
- проверить установку протобафа: 
```bash
 protoc --version
```

#### Go gRPC плагины для protoc:
```go
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```
Убедиться что путь, эквивалентный `$GOPATH/bin` находится в переменной `$PATH`.


### СОЗДАНИЕ .proto - ФАЙЛА

```proto
syntax = "proto3";  
  
option go_package= "simple_grpc_app/calculatorpb";  
  
package calculator;  
  
message SumRequest {  
    int32 a = 1;  
    int32 b = 2;  
}  
  
message SumResponce {  
    int32 result = 1;  
}  
  
// Сервис  
service CalculatorService{  
    rpc Sum(SumRequest) returns (SumResponce);  
}
```

### protoc - КОМПИЛЯЦИЯ ФАЙЛОВ

Компиляция происходит при помощи команды вида: 
```bash
protoc --proto_path=proto --go_opt=paths=source_relative --go_out=./proto_out/ --go-grpc_out=./proto_out/ --go-grpc_opt=paths=source_relative calculator.proto
```
Где:
- `--proto_path=proto` или `-I proto` - флаг с указанием директории (`proto`) с исходными .proto-файлом 
- `--go_out=./proto_out/` - указывает `protoc-gen-go` генерировать Go-код в  директорию `proto_out`
- `--go_opt=paths=source_relative` - указывает, что пути к генерируемым файлам должны быть относительными к папке, указанной флагом `--go_out`
- `--go-grpc_out=./proto_out/` - указывает `protoc-gen-go-grpc` генерировать Go-код в  директорию `proto_out`
- `--go-grpc_opt=paths=source_relative` - указывает, что пути к генерируемым файлам должны быть относительными к папке, указанной флагом `--go-grpc_out`
- `calculator.proto` -исходный .proto-файл, который нужно компилировать

В итоге получаем 2 файла: 
- **calculator.pb.go** -  Содержит сгенерированные структуры Go для ваших Protocol Buffer сообщений.
- **calculator_grpc.pb.go** - Содержит сгенерированные интерфейсы для нашего gRPC-сервиса, которые нам нужно будет реализовать (для сервера) и использовать (для клиента).

