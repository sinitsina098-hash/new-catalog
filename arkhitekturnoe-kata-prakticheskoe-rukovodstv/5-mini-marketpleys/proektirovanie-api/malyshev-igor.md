---
order: 2
title: Малышев Игорь
---

## 1\. 📖 Описание функциональных границ микросервиса

### 1\.1 Диаграмма компонентов архитектуры

<mermaid path="./malyshev-igor.mermaid" width="767px" height="405px"/>

### 1\.2 Описание микросервиса

Auth Service: регистрация, аутентификация, управление профилями и ролями, сессиями и токенами (access/refresh), 2FA/OTP, восстановление пароля, блокировки/разблокировки пользователей. Границы: не хранит данные каталога/заказов/платежей; публикует события (user.created, user.banned, user.flagged) в шину сообщений; делегирует отправку e-mail/SMS внешним провайдерам; предоставляет introspect API для других сервисов.

## 2\. 🧩 Концептуальное проектирование API метода

<table header="row">
<colgroup><col width="156"/><col width="156"/><col width="156"/><col width="192"/><col width="239"/></colgroup>
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

Продавец (Mobile/Web)

</td>
<td>

Создать объявление (создать listing)

</td>
<td>

\- Получить presigned URL для изображений\
\- Принять метаданные объявления\
\- Создать черновик, затем отправить на автоматическую проверку\
\- Индексировать для поиска (после модерации)

</td>
<td>

\{ "title":"iPhone 18, 128GB", "description":"Состояние: почти новое", "price":24990, "currency":"RUB", "categoryId":"phones", "attributes":\{"brand":"Apple","model":"13","color":"black"}, "images":\["img1.jpg", "..."\]

</td>
<td>

201 Created: json \{ "id":"listing-uuid", "status":"pending_auto_check", "createdAt":"2025-09-20T12:00:00Z", "moderationScore":0.86 }

</td>
</tr>
<tr>
<td>

Покупатель / Поиск

</td>
<td>

Найти товар по фильтрам и сортировке

</td>
<td>

\- Быстрый полнотекстовый поиск\
\- Фильтры по цене/бренду/категории/локации\
\- Пагинация и релевантность

</td>
<td>

GET /v1/catalog/search?q=iphone&minPrice=10000&maxPrice=30000

</td>
<td>

200 OK: json \{ "total":1234, "page":1, "size":20, "items":\[\{"id":"...","title":"...", "price":...}\] }

</td>
</tr>
<tr>
<td>

Moderator UI / Fraud

</td>
<td>

Пометить объявление как отклоненное или заблокировать

</td>
<td>

\- Менять статус (approved/rejected/flagged)\
\- Логировать причину и ревизии

</td>
<td>

PATCH /v1/catalog/\{id}/moderation \{status:"rejected", reason:"fake photos"}

</td>
<td>

200 OK + event `listing.moderated`

</td>
</tr>
</table>





## 3\. 🤝 Swagger

<openapi src="./_index.yaml" flag="true"/>

###