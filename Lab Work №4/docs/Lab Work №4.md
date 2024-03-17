# Лабораторная работа 4

## Документация по api

Работа с иногородними пациентами

#### Примечание:
* Так как все api работают с веб-интерфейсом, все ответы возвращают статус OK (200) для облегчения обработки ответов, внутри себя содержат DTO c со следующей структурой:

        {
            code: HttpStatusCode (int),
            message: string,
            content: object
        }
    Все коды статусов ответов, описанных ниже будут взяты из DTO.

#### GET /api/Ingorod

Описание: получить иногородних пользователей. Возвращает пользователей согласно введенным параметрам поиска.

Заголовки:
* Authorization: Bearer

**Входные параметры:**
* Query Params согласно спецификации OData

Пример запроса:

Код статуса ответа:
* 200 OK - пользователь найден
* 404 Not Found - что то не сработало
* 401 No authorizated - токен не прошел аутентификацию

#### POST /api/Ingorod

Описание: создание нового пациента

Заголовки:
* Authorization: Bearer

**Входные параметры:**

Body:
* id - идентификатор пациента
* name - Имя
* surname - Фамилия
* thirdName - Отчество
* birthDay - Дата рождения
* smo - СМО
* polis - Полис
* polisType - Тип полиса
* document - Серия и номер документа
* documentType - Тип документа
* smp - номер станции СМП

Пример запроса: 

    {
    "id": "string",
    "name": "string",
    "surname": "string",
    "thirdName": "string",
    "birthDay": "2024-03-08",
    "smo": "string",
    "polis": "string",
    "polisType": 0,
    "document": "string",
    "documentType": "string",
    "smp": "string"
    }

Код статуса ответа:
* 200 OK - пациент добавлен
* 404 Not Found - Что то пошло не так
* 401 No Authorized - Токен не прошел аутентификацию

Примеры ответа:


#### PATCH /api/Ingorod

Описание: Отредактировать данные пациента

Заголовки:
* Authorization: Bearer

**Входные параметры:**

Query Params:
* id - идентификатор пациента

Body: 
* id - идентификатор пациента
* name - Имя
* surname - Фамилия
* thirdName - Отчество
* birthDay - Дата рождения
* smo - СМО
* polis - Полис
* polisType - Тип полиса
* document - Серия и номер документа
* documentType - Тип документа
* smp - номер станции СМП


#### DELETE /api/Ingorod

Описание: Удалить пациента

Заголовки:
* Authorization: Bearer

**Входные параметры:**

Query Params:
* id - идентификатор пациента
