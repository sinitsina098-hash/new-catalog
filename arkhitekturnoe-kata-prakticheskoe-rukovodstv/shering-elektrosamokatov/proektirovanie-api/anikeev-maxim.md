---
order: 0.5
title: Аникеев Максим
---

## 1\. 📖 Описание функциональных границ микросервиса

### 1\.1 Диаграмма компонентов архитектуры

<mermaid path="./anikeev-maxim.mermaid" width="780px" height="140px"/>

### 1\.2 Описание микросервиса

Микросервис ***Notification Service*** является централизованным центром для отправки всех видов уведомлений пользователям и операторам системы. Его ключевая ответственность -- принимать запросы на отправку уведомлений, выбирать оптимальный канал доставки, интегрироваться с соответствующими внешними провайдерами (email, SMS, push) и надежно доставлять сообщения, обеспечивая трассируемость и управление шаблонами.

**Notification service** обеспечивает следующую функциональность:

-  Прием запросов на отправку уведомлений через единый API endpoint.

-  Поддержка множества каналов доставки: SMS, E-mail, Push-уведомления (iOS/Android), In-App уведомления.

-  Выбор и конфигурация канала доставки на основе типа уведомления и предпочтений пользователя.

-  Управление шаблонами сообщений для каждого типа уведомления и канала (локализация, персонализация).

-  Интеграция с внешними провайдерами услуг (например, Twilio для SMS, SendGrid для email, Firebase Cloud Messaging для push).

-  Логирование и отслеживание статуса доставки каждого уведомления.



## 2\. 🧩 Концептуальное проектирование API метода

<table header="row">
<colgroup><col width="191"/><col width="180"/><col width="230"/><col width="236"/><col width="239"/></colgroup>
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

**Сервис поездок (Trip Service)**

</td>
<td>

Информирование пользователя на всех этапах поездки

</td>
<td>

1\. Уведомление об успешной разблокировке\
2\. Информирование о завершении поездки

</td>
<td>

**Path Parameters:**\
\- `userId: UUID`

\- `scooterId: UUID`

**Body JSON:**

`eventKey: string`\
`channels: string[]`

`startTime: string`

`endTime: string`

</td>
<td>

**Response (200):**\
`notificationId: UUID`\
`status: accepted`\
\
**Ошибки:**\
\- 404 Not Found\
\- 429 Too Many Requests

</td>
</tr>
<tr>
<td>

**Платежный сервис (Payment Service)**

</td>
<td>

Информирование о финансовых операциях

</td>
<td>

Подтверждение успешного платежа

</td>
<td>

**Path Parameters:**\
\- `userId: UUID`\
\- `transactionId: UUID`\
**Body JSON:**\
`eventKey: string`\
`channels: string[]`\
`amount: number`\
`currency: string`

</td>
<td>

**Response (200):**\
`notificationId: UUID`\
`status: accepted`\
\
**Ошибки:**\
\- 400 Bad Request\
\- 403 Forbidden

</td>
</tr>
<tr>
<td>

**Сервис пользователей (User Service)**

</td>
<td>

Коммуникация с пользователем по учетным вопросам

</td>
<td>

1\. Восстановление пароля\
2\. Подтверждение email/телефона\
3\. Уведомление об изменении данных

</td>
<td>

**Path Parameters:**

\- `userId: UUID`

**Body JSON:**

`eventKey: string`\
`channels: string[]`\
`userName: string`\
`signupDate: string`

</td>
<td>

**Response (200):**\
`notificationId: UUID`\
`status: accepted`

\
**Ошибки:**\
\- 404 Not Found\
\- 500 Internal Server Error

</td>
</tr>
<tr>
<td>

**Мобильное приложение**

</td>
<td>

Локальные уведомления и in-app messages

</td>
<td>

1\. Локальные напоминания\
2\. In-app сообщения\
3\. Обновления интерфейса в реальном времени\
4\. Уведомления о новых функциях\
5\. Персонализированные предложения

</td>
<td>

**Path Parameters:**\
\- `userId: UUID`\
\
**Body JSON:**

`eventKey: string`\
`channels: string[]`

`title: string`\
`message: string`

</td>
<td>

**Response (200):**\
`notificationId: UUID`\
`status: delivered`\
\
**Ошибки:**\
\- 404 Not Found\
\- 429 Too Many Requests

</td>
</tr>
</table>

## 3\. 🤝 Swagger

<openapi src="./anikeev-maxim.yaml" flag="true"/>