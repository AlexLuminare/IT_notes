### УСТАНОВКА КОМПОНЕНТОВ
#### Protoc
- [https://github.com/protocolbuffers/protobuf/releases](https://github.com/protocolbuffers/protobuf/releases)- скачать нужный релиз софтины
- распаковать архив в любое удобное место на компе
-  В системной переменной `PATH` установить абсолютный путь к директории /bin протобfфа
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


### ИНСТРУМЕНТЫ  ДЛЯ РАБОТЫ С gRPC
- **protoc** - компилятор, который генерирует код на основе ваших `.proto` файлов.
- **Go gRPC плагины для protoc** - генерируют Go-код для gRPC.
- **protoc-gen-go** - 
- **protoc-gen-go-grpc** - 