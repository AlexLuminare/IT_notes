
### ИНСТРУМЕНТЫ  ДЛЯ РАБОТЫ С gRPC
- **protoc** - компилятор, который генерирует код на основе ваших `.proto` файлов.
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

### КОМПИЛЯЦИЯ ФАЙЛОВ
protoc --proto_path=proto --go_out=./ --go_opt=paths=source_relative 
--go-grpc_out=./ --go-grpc_opt=paths=source_relative proto/calculator.proto