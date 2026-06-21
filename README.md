## Практическая работа №2. Синицын Антон, ЭФМО-01-25

### gRPC: создание простого микросервиса, вызовы методов 11.03.2026


### Цель:
Научиться работать с gRPC: описывать контракт в .proto, поднимать gRPC-сервер и вызывать его из другого сервиса (клиента) с дедлайном.


## Структура проекта

```
.
│   go.mod
│   go.sum
│   README.md
├───docs/
│     ├───  pz17_api.md
│     └───  pz17_diagram.md
├───proto/
│    └───auth.proto
├───pkg/
│    ├───auth.pb.go
│    └───auth_grpc.pb.go
├───services/
│   ├───auth/
│   │   ├───go.mod
│   │   ├───go.sum
│   │   ├───cmd/
│   │   │   └───auth/
│   │   │          └─── main.go
│   │   ├───internal/
│   │   |   ├───grps/
│   │   |   │       └─── server.go
│   │   |   ├───handler/
│   │   |   │       └─── auth_handler.go
│   │   |   └───service/
│   │   |           └─── auth_servise.go
│   └───tasks/
│       │   go.mod
│       │   go.sum
│       ├───cmd/
│       │   └───tasks/
│       │           └───main.go
│       ├───internal/
│       |   ├───client/
│       |   │       └───auth_client.go
│       |   ├───handler/
│       |   │       └───task_handler.go
│       |   └───service/
│       |           └───task_service.go
└───shared/
    ├───httpx/
    │       └───client.go
    ├───middleware/
    │       ├───logging.go
    │       └───requestid.go
    └───models/
            └───models.go
```



## Prorto файл
```
syntax = "proto3";

package auth;

option go_package = "tech-ip-sem2/proto/authpb";

service AuthService {
    rpc Verify(VerifyRequest) returns (VerifyResponse);
}

message VerifyRequest {
    string token = 1;
}

message VerifyResponse {
    bool valid = 1;
    string subject = 2;       
    string error = 3;     
}
```


## Команды генерации
```
# Установка плагинов
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest


# Генерация
protoc --proto_path=proto \
    --go_out=pkg \
    --go_opt=paths=source_relative \
    --go-grpc_out=pkg \
    --go-grpc_opt=paths=source_relative \
    proto/auth.proto
```


После генерации создаются файлы:
1. pkg/auth.pb.go
2. pkg/auth_grpc.pb.go


## Таблица ошибок

| HTTP статус | Описание | Сообщение |
|-------------|----------|------------------------|
| **400 Bad Request** | Неверный формат запроса | `{"error": "invalid request format"}` |
| **400 Bad Request** | Неверные учетные данные| `{"error": "invalid credentials"}` |
| **401 Unauthorized** | Отсутствует заголовок Authorization | `{"error": "invalid authorization"}` |
| **401 Unauthorized** | Неверный формат заголовка Authorization | `{"error": "invalid authorization format"}` |
| **401 Unauthorized** | Токен невалиден | `{"error":"unauthorized - invalid token"}` |
| **404 Not Found** | Задача с указанным ID не найдена | `{"error": "task not found"}` |
| **503 Service Unavailable** | Auth сервис недоступен | `{"error": "auth service unavailable"}` |

### Требования
- Go 1.21 или выше
- Порт 8081 для сервиса авторизации и 8082 для сервися заданы должны быть свободны

### Инструкция по запуску
1. Скопировать репозиторий
   ```
   https://github.com/Yaroslavaaaa/tech-ip-sem2-pz2.git
   ```
2. Войти в папку где лежит сервис авторизации и запустить его
   ```
   cd services/auth
   go run ./cmd/auth
   ```
3. Веонуться в корневую папку и войти в папкус где лежит сервис задач
   ```
   cd ../..
   cd services/tasks
   go run ./cmd/tasks
   ```
4. Приложение готово к использованию
   

### Создание задачи
<img width="1374" height="506" alt="2026-03-11_12-38-18" src="https://github.com/user-attachments/assets/7bbaad6e-61f1-4576-bc38-a76a830bfb85" />

### Просмотр списка задач
<img width="1373" height="640" alt="2026-03-11_12-38-35" src="https://github.com/user-attachments/assets/ca21f7bf-38ce-496f-8755-52ab2b9783df" />

### Логи сервисов при создании задачи и просмотре списка задач
<img width="620" height="113" alt="2026-03-11_12-36-34" src="https://github.com/user-attachments/assets/4c94b243-79ae-417f-8d8e-15c91a86e2bd" />
<img width="799" height="399" alt="2026-03-11_12-37-48" src="https://github.com/user-attachments/assets/25548e7b-b137-44bf-b60b-d39bae7828da" />

### Попытка создать задачу с невалидным токеном
<img width="1397" height="553" alt="2026-03-11_12-39-52" src="https://github.com/user-attachments/assets/137e6952-c741-4bc5-8cde-3ac67c348b24" />

### Логи при попытке создать задачу с невалидным токеном
<img width="436" height="40" alt="2026-03-11_12-40-14" src="https://github.com/user-attachments/assets/58f77c71-7469-4ed6-b031-735b315a8239" />
<img width="793" height="70" alt="image" src="https://github.com/user-attachments/assets/906f6d16-d801-42e9-ab7e-162c0b9cc450" />

### Попытка создать задачу когда сервис авторизации недоступен
<img width="1384" height="512" alt="2026-03-11_12-41-21" src="https://github.com/user-attachments/assets/dbf3d8f8-6370-4ee6-9d1a-1c910d64bcee" />

### Логи при попытке создать задачу когда сервис авторизации недоступен
<img width="804" height="70" alt="image" src="https://github.com/user-attachments/assets/a1c9e057-89ec-43f1-9bef-48d01c97abe7" />






## Ответы на контрольные вопросы:
1. Что такое .proto и почему он считается контрактом?
.proto - это язык описания интерфейсов для Protocol Buffers от Google. Он считается контрактом, потому что строго определяет типы данных и структуру сообщений, задает сигнатуры методов сервиса, обеспечивает языковую независимость и гарантирует, что клиент и сервер будут придерживаться единого формата обмена данными. Любое изменение в .proto требует согласования между обеими сторонами.
2. Что такое deadline в gRPC и чем он полезен?
Deadline - это максимальное время, которое клиент готов ждать ответ от сервера. Он полезен тем, что предотвращает зависание приложений, освобождает ресурсы, позволяет быстро обнаруживать проблемы с доступностью сервисов, улучшает пользовательский опыт и дает возможность реализовать graceful degradation, когда при недоступности одного сервиса система возвращает осмысленную ошибку, а не зависает.
3. Почему “exactly-once” не даётся просто так даже в RPC?
Exactly-once не даётся просто из-за проблем распределенных систем: сетевые сбои, сбои сервера, сбои клиента, таймауты. На практике используются более простые гарантии: at-most-once или at-least-once, а exactly-once требует сложных механизмов вроде идемпотентности операций и дедупликации запросов.
4. Как обеспечивать совместимость при расширении .proto?
Совместимость при расширении .proto обеспечивается следующими правилами: никогда не изменять номера тегов существующих полей, не изменять типы полей, для новых полей использовать только новые номера тегов, при удалении полей помечать их как reserved, чтобы их номера нельзя было использовать повторно, использовать optional для необязательных полей, использовать repeated для списков, а также добавлять новые поля только в конец сообщения. Эти правила позволяют старым клиентам работать с новыми версиями сервера и наоборот.






