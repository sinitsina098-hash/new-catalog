---
order: 4
title: Андрей Непомнящий
---

## 1\. 📖 Описание функциональных границ микросервиса

### 1\.1 Диаграмма компонентов архитектуры

<mermaid path="./arkhitekturnoe-kata.mermaid" width="780px" height="395px"/>

### 1\.2 Описание микросервиса

**Fraud detection Service** -- микросервис, который отвечает за анализ объявлений на предмет фрода, их фильтрацию и приостановку.

Функциональность сервиса:

-  Возможность провести скоринг объявления на предмет фрода

-  Возможность автоматически заблокировать постинг/обновление объявления

-  Возможность приостановки публикации/обновления объявления для ручного анализа

-  При блокировке автоматически производится и приостановка, но при приостановке блокировка необязательна

-  Отправить запрос может только система или модератор

-  Скоринг проводится автоматически при каждой попытке публикации/обновлении объявления

-  Модератор может просматривать, вручную публиковать/блокировать подозрительные объявления

-  Подозрительные объявления отмечаются в БД

## 2\. 🧩 Концептуальное проектирование API метода

<table header="row">
<colgroup><col width="131"/><col width="134"/><col width="199"/><col width="192"/><col width="239"/></colgroup>
<tr>
<td>

Потребители

</td>
<td>

Цель

</td>
<td>

Задачи

</td>
<td>

Входные данные

</td>
<td>

Выходные данные

</td>
</tr>
<tr>
<td>

Продавцы

</td>
<td>

Публикация объявления

</td>
<td>

-  Автоматический скоринг при публикации

-  Приостановка публикации

-  Блокировка публикации

</td>
<td>

```json
{ 
	"user_id": "u_uuid",  
	"product_id": "p_uuid", 
	"text": "text",
	"photos": ["photos"],
	"created_at": "datetime",
	"modified_at": "datetime",
	"place": "place"
}
```

</td>
<td>

statuses

\[200, 400, 401, 403, 404\]

**user_response**

```json
{
	"status_code": 200,
 	"message": "Публикация заблокирована" 
}
```

**system_response** (ответ, который хранится внутри системы и служит для добавление информации во внутренние бд)

```json
{
	"user_id": "u_uuid",
	"product_id": "p_uuid",
	"old_product_id": "p_uuid",
	"status": "blocked",
	"created_at": "datetime",
	"modified_at": "datetime", 
	"text": "text",
	"photos": ["photos"],
	"place": "place"
	"score": 0.8,
	"block": true,
	"stop_for_manual": true
}
```

</td>
</tr>
<tr>
<td>

Продавцы

</td>
<td>

Обновление объявления

</td>
<td>

-  Автоматический скоринг при обновлении

-  Приостановка обновления

-  Блокировка обновления

</td>
<td>

```json
# каждую модификацию объявления будем
# записывать как новый продукт с сохранением
# старого uid для удобного трекинга версий
{ 
	"user_id": "u_uuid",  
	"product_id": "p_uuid",
	"old_product_id": "old_p_uuid",
	"text": "text",
	"photos": ["photos"],
	"created_at": "created_datetime",
	"modified_at": "datetime",
	"place": "place"
}
```

</td>
<td>

statuses

\[200, 400, 401, 403, 404\]

**user_response**

```json
{
	"status_code": 200,
 	"message": "Обновление заблокировано"  
}
```

**system_response** (ответ, который хранится внутри системы и служит для добавление информации во внутренние бд)

```json
{
	"user_id": "u_uuid",
	"product_id": "p_uuid",
	"old_product_id": "old_p_uuid",
	"status": "blocked",
	"created_at": "created_datetime",
	"modified_at": "datetime", 
	"text": "text",
	"photos": ["photos"],
	"place": "place"
	"score": 0.8,
	"block": true,
	"stop_for_manual": true
}
```

</td>
</tr>
<tr>
<td>

Модераторы

</td>
<td>

Получение материалов по объявлению для модерации

</td>
<td>

-  Выдать модератору материалы по объявлению по uid юзера и объявления

</td>
<td>

```json
{ 
	"user_id": "u_uuid",  
	"product_id": "p_uuid",
}
```

</td>
<td>

statuses

\[200, 400, 401, 403, 404\]

**user_response** 

```json
{
	"user_id": "u_uuid",
	"product_id": "p_uuid",
	"old_product_id": "old_p_uuid",
	"status": "blocked",
	"created_at": "created_datetime",
	"modified_at": "datetime", 
	"text": "text",
	"photos": ["photos"],
	"place": "place"
	"score": 0.8,
	"block": true,
	"stop_for_manual": true
}
```

</td>
</tr>
<tr>
<td>

Модераторы

</td>
<td>

Модерация объявления

</td>
<td>

-  Отправить в сервис и БД результат

</td>
<td>

```json
{ 
	"user_id": "u_uuid",  
	"product_id": "p_uuid",
	"status": "blocked",
	"solution_dt": "datetime"
}
```

</td>
<td>

statuses

\[200, 400, 401, 403, 404\]

**user_response**

```json
{
	"status_code": 200,
 	"message": "Блокировка произведена успешно"  
}
```

**system_response** (ответ, который хранится внутри системы и служит для добавление информации во внутренние бд)

```json
{
	"user_id": "u_uuid",
	"product_id": "p_uuid",
	"status": "blocked",
	"block": true,
	"stop_for_manual": true
}
```

</td>
</tr>
</table>

## 3\. 🤝 Swagger

<openapi src="./_index-2.yaml" flag="true"/>

### 