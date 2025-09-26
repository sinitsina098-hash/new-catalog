---
order: 2
title: Костин Андрей Михайлович
---

## 1\. 📖 Описание функциональных границ микросервиса

### 1\.1 Диаграмма компонентов архитектуры

<mermaid path="./arkhitekturnoe-kata-3.mermaid" width="780px" height="603px"/>

### 1\.2 Описание микросервиса

**Review Service** -- микросервис, который отвечает за управление отзывами внутри мини-маркетплеса.

функциональность сервиса:

-  возможность оставлять отзывы с оценкой (от 1 до 5) и текстовым комментарием

-  отзывы могут оставлять только покупатели

-  отзыв можно оставить на:

   -  товар (описать качество товара, соответствие, опыт использования)

   -  продавца (описать взаимодействие, скорость доставки/ответов в чате)

-  отзыв можно редактировать/удалять в течение 1 месяца после создания

-  модераторы могут удалять отзывы, нарушающие правила площадки

-  возможность просмотра отзыва по товару или продавцу

-  автоматический расчёт средней оценки товара или продавца

## 2\. 🧩 Концептуальное проектирование API метода

<table header="row">
<colgroup><col width="156"/><col width="156"/><col width="310"/><col width="192"/><col width="239"/></colgroup>
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

покупатель



</td>
<td>

оставить отзыв о товаре

</td>
<td>

-  создать отзыв

-  указание оценки и текста отзыва

-  привязка отзыва к товару или продаву

</td>
<td>

```json
{ 
	"user_id": "u_uuid",  
	"product_id": "p_uuid",
	"order_id": "o_uuid", 
	"rating": 4, 
	"text": "text" 
}
```



</td>
<td>

statuses

\[201, 400, 401, 404\]

**user_response**

```json
{
	"status_code": 201,
	"message": "отзыв успешно опубликован",
	"rating": "4",
	"text": "text"
}
```

**system_response** (ответ, который хранится внутри системы и служит для добавление информации во внутренние бд)

```json
{
	"user_id": "a_uuid",
	"product_id": "t_uuid",
	"order_id": "o_uuid",
	"review_id": "r_uuid",
	"status": "active",
	"created_at": "date",
	"modified_at": "date"
}
```

</td>
</tr>
<tr>
<td>

покупатель

</td>
<td>

публикация обновленного  отзыва

</td>
<td>

-  изменить текст и/или оценку (сам факт изменения, без проверок на возможность внесения изменений)

</td>
<td>

```json
{
	"user_id": "u_uuid",
	"review_id": "r_uuid",
	"rating": 4,
	"text": "new text"
}
```

</td>
<td>

statuses

\[200, 400\]

**user_response**

```json
{
	"status_code": 200,
	"message": "отзыв успешно обновлён",
	"rating": 4,
	"text": "new text"
}
```

**system_response**

```json
{
	"user_id": "u_uuid",
	"review_id": "r_uuid",
	"status": "active",
	"modified_at": "date"
}
```

</td>
</tr>
<tr>
<td>

покупатель

</td>
<td>

удаление отзыва

</td>
<td>

-  удаление отзыва (сам процесс удаления отзыва, без проверки на возможность удаления)

</td>
<td>

```json
{
	"user_id": "u_uuid",
	"review_id": "r_uuid"
}
```

</td>
<td>

statuses

\[200, 400\]

**user_response**

```json
{
	"status_code": 200,
	"message": "отзыв успешно удалён"
}
```

**system_response**

```json
{
	"user_id": "u_uuid",
	"review_id": "r_uuid"
}
```

</td>
</tr>
<tr>
<td>

покупатели / продавцы / модераторы

</td>
<td>

посмотреть существующие отзывы

</td>
<td>

-  получение списка отзывов по товару или продавцу

</td>
<td>

```json
{
	"type": "product/user",
	"product_id": "p_uuid",
	"page": 1,
	"limit": 20,
	"sorting_field": "field"
}
```

</td>
<td>

statuses

\[200, 400\]

**user_response**

```json
{
	"status_code": 200,
	"reviews": [
					{
						"name": "r_name",
						"rating": 4,
						"text": "text",
						"modified_at": "date"
					},
				],
}
```



**system_response**

```json
{
	"product_id": "p_uuid",
	"page": 1,
	"limit": 20,
	"sorting_field": "field"
}
```



</td>
</tr>
<tr>
<td>

модератор

</td>
<td>

удалить отзыв, нарушающий правила площадки

</td>
<td>

-  удаление отзыва (сам процесс удаления отзыва после проверки на возможность удаления)

</td>
<td>

```json
{
	"user_id": "u_uuid",
	"review_id": "r_uuid"
}
```

</td>
<td>

statuses

\[200, 400\]

**user_response**

```json
{
	"status_code": 200,
	"message": "отзыв успешно удалён"
}
```

**system_response**

```json
{
	"user_id": "u_uuid",
	"review_id": "r_uuid"
}
```



</td>
</tr>
</table>

## 3\. 🤝 Swagger

<openapi src="./kostin-andrey-mikhaylovich.yaml" flag="true"/>