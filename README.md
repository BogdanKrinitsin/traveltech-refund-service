# ТЗ на разработку микросервиса возвратов (Refund-Service) в рамках контура TravelTech

Документ описывает логику работы бэкенд-сервиса автоматического оформления возвратов при отмене бронирования пользователем. Сервис должен валидировать билеты, блокировать места в инвенторной системе и проводить финансовый реверс транзакции через шлюз банка-эквайера.

## 1. Сквозной процесс обработки запроса (Sequence Diagram)

Логика взаимодействия Фронтенда, сервиса возвратов, БД и API банка приведена на схеме ниже.

```mermaid
sequenceDiagram
    actor User as Юзер
    participant Front as Frontend (App/Web)
    participant Service as Refund-Service
    participant DB as БД (PostgreSQL)
    participant Bank as API Банка

    User->>Front: Клик на "Вернуть билет"
    Front->>Service: POST /v1/refunds {ticket_id, reason}
    Note over Service: Открытие транзакции
    Service->>DB: Проверка статуса (status == 'active')
    DB-->>Service: Билет валиден
    
    Service->>DB: Смена статуса билета на 'lock_for_refund'
    Service->>Bank: POST /v1/payments/reverse {transaction_id, amount}
    Note over Bank: Проведение чарджбэка
    Bank-->>Service: HTTP 200 OK {status: success, gateway_transaction_id}
    
    Service->>DB: INSERT в таблицу refunds + статус билета 'refunded'
    Note over Service: Коммит транзакции
    
    Service-->>Front: HTTP 200 OK {status: completed, refund_id}
    Front-->>User: Показ экрана с успешным статусом 







