---
order: 3
title: Абалтусова Екатерина
---

## 1\. 📖 Описание функциональных границ микросервиса

### 1\.1 Диаграмма компонентов архитектуры

<mermaid path="./abaltusova-ekaterina.mermaid" width="492px" height="363px"/>

### 1\.2 Описание микросервиса

Auth Service отвечает за всю логику, связанную с управлением пользователями и их доступом к системе.

##### Основные функции:

-  Регистрация новых пользователей (email/phone, buyer/seller).

-  Подтверждение личности через email/SMS (OTP/verification link).

-  Поддержка входа с использованием внешних провайдеров (Google, Apple ID).

-  Аутентификация пользователей через username/password, refresh-токены или external IdP.

-  Управление сессиями: хранение access/refresh токенов, logout, revoke.

-  Управление ролями и правами: покупатель, продавец, модератор, администратор.

-  Двухфакторная аутентификация (2FA): SMS/OTP/TOTP.

-  Восстановление пароля (reset via email/SMS).

-  Блокировка и разблокировка аккаунтов при подозрительных действиях.

-  Ведение истории логинов, устройств и IP-адресов.

##### Интеграции:

-  Внешние сервисы доставки уведомлений (Email/SMS) для верификации.

-  Интеграция с внешними IdP по протоколам OAuth2/OIDC.

-  Публикация событий (`user.created`, `user.verified`, `user.banned`, `user.suspicious_activity`) в брокер сообщений для Fraud Detection и Notification Service.

-  Другие сервисы используют introspect API для проверки токенов и получения claims пользователя.

##### Границы:

-  Auth Service не управляет контентом (объявления, заказы, платежи).

-  Не хранит медиафайлы.

-  Является единственным источником правды о пользователях.

## 2\. 🧩 Концептуальное проектирование API метода

<table header="row">
<colgroup><col width="137"/><col width="171"/><col width="250"/><col width="192"/><col width="239"/></colgroup>
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

Web/Mobile (покупатель/продавец)

</td>
<td>

Регистрация нового пользователя

</td>
<td>

\- Собрать данные профиля\
\- Зарегистрировать учетку\
\- Отправить верификацию email/SMS\
\- Сгенерировать событие UserCreated

</td>
<td>

json \{ "email":"user@ex.com", "password":"P@ssw0rd!", "displayName":"Иван", "type":"buyer|seller", "phone":"+7999..." } 

</td>
<td>

201 Created и тело: json 

\{ "id":"uuid", "email":"user@ex.com", "status":"pending_verification", "createdAt":"2025-09-20T12:34:56Z" } 

HTTP заголовки: Location: /users/\{id}

</td>
</tr>
<tr>
<td>

Web/Mobile

</td>
<td>

Аутентификация (логин)

</td>
<td>

\- Валидация пароля\
\- Проверка блокировок/фрод-статуса\
\- Выдать JWT access + refresh token\
\- Логировать событие

</td>
<td>

json \{ "grant_type":"password", "username":"user@ex.com", "password":"P@ssw0rd!", "deviceId":"string" } 

</td>
<td>

200 OK: json 

\{ "access_token":"jwt.v1...", "expires_in":3600, "refresh_token":"opaque-token", "refresh_expires_in":604800, "token_type":"Bearer" } 

Set-Cookie (HttpOnly) опционально для web.

</td>
</tr>
<tr>
<td>

Services (Catalog, Order)

</td>
<td>

Валидация токена / получение профиля

</td>
<td>

\- Проверить подпись JWT\
\- Возвратить минимальный профиль и роли

</td>
<td>

HTTP GET /introspect или /users/\{id}?fields=minimal, Authorization: Bearer

</td>
<td>

200 OK: json 

\{ "id":"uuid", "email":"user@ex.com", "roles":\["buyer","seller"\], "flags":\{"risk":"low"} }

</td>
</tr>
</table>

## 3\. 🤝 Swagger

<openapi src="./_index.yaml" flag="true"/>

###