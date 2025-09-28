---
order: 3
title: Абалтусова Екатерина
---

## 1\. 📖 Описание функциональных границ микросервиса

### 1\.1 Диаграмма компонентов архитектуры

<mermaid path="./abaltusova-ekaterina.mermaid" width="780px" height="486px"/>

### 1\.2 Описание микросервиса

**Auth Service** (микросервис аутентификации и авторизации) отвечает исключительно за проверку личности пользователей и управление их доступом к системе. Не хранит личные данные пользователей (профиль, настройки).

##### Основные функции:

-  Аутентификация: проверка учетных данных (логин/пароль, OAuth2/OIDC провайдеры)

-  Управление сессиями: выдача JWT access/refresh токенов, валидация токенов, отзыв сессий (logout/revoke)

-  Двухфакторная аутентификация (2FA): запрос и проверка кодов через SMS/TOTP

-  Блокировка доступа: временная блокировка при подозрительной активности

-  Ведение лога аутентификаций: фиксация попыток входа с привязкой к устройству и IP

##### **Интеграции:**

-  User Profile Service: запрос на проверку учетных данных и получение ролей пользователя

-  Внешние Identity Providers (Google, Apple ID) по протоколам OAuth2/OIDC

-  Брокер сообщений: публикация событий (user.logged_in, user.failed_login, user.banned)

-  Все сервисы системы: предоставление introspect API для проверки токенов и прав доступа

##### **Границы:**

-  Не хранит профили пользователей (имя, телефон, аватар) - только идентификаторы и роли

-  Не управляет бизнес-данными (товары, заказы, платежи)

-  Не является источником правды о пользователе, потребляет данные из Profile Service

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

Web/Mobile

</td>
<td>

Регистрация нового пользователя

</td>
<td>

-  Собрать данные профиля

-  Зарегистрировать учетку

-  Сгенерировать событие UserCreated (создать юзера)

</td>
<td>

\{ "email":"[user@ex.com](mailto:user@ex.com)", "password":"P@ssw0rd!", "displayName":"Иван", "type":"buyer", "phone":"+79990000000" }

</td>
<td>

201 Created + \{ "id":"uuid", "email":"[user@ex.com](mailto:user@ex.com)", "status":"pending_verification", "createdAt":"2025-09-20T12:34:56Z" }

Заголовок: Location: /users/\{id}

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

-  Проверить учетные данные

-  Сгенерировать JWT токены

</td>
<td>

\{ "email": "[user@ex.com](mailto:user@ex.com)", "password": "P@ssw0rd!" }

</td>
<td>

200 OK + \{ "access_token": "jwt_token", "expires_in": 3600, "refresh_token": "refresh_token", "token_type": "Bearer" }

</td>
</tr>
<tr>
<td>

Web/Mobile

</td>
<td>

Верификация существующего пользователя 

</td>
<td>

-  Отправить email с верификацией

-  Проверить код подтверждения

-  Верифицировать статус аккаунта или предложить отправить код повторно

</td>
<td>

\{ "id",

"email": "[user@ex.com](mailto:user@ex.com)", "verification_code": "123456" }

</td>
<td>

200 OK + \{ "id":"uuid", "email":"[user@ex.com](mailto:user@ex.com)", "status":"verified", "verifiedAt":"2025-09-20T12:35:56Z" }

</td>
</tr>
<tr>
<td>

Web/Mobile

</td>
<td>

Верификация нового пользователя 

</td>
<td>

-  Отправить email с верификацией

-  Проверить код подтверждения

-  Верифицировать статус аккаунта или предложить отправить код повторно

</td>
<td>

\{ ""email": "[user@ex.com](mailto:user@ex.com)", "verification_code": "123456" }

</td>
<td>

201 OK + \{ "id":"uuid", "email":"[user@ex.com](mailto:user@ex.com)", "status":"verified", "verifiedAt":"2025-09-20T12:35:56Z" }

</td>
</tr>
<tr>
<td>

Web/Mobile

</td>
<td>

Повторная верификация пользователя (если применимо)

</td>
<td>

-  Проверить существование пользователя

-  Отправить новый код подтверждения по указанному email



</td>
<td>

\{ ""email": "[user@ex.com](mailto:user@ex.com)", "verification_code": "123456" }

</td>
<td>

200 OK + \{ "id":"uuid", "email":"[user@ex.com](mailto:user@ex.com)", "status":"verified", "verifiedAt":"2025-09-20T12:35:56Z" }

</td>
</tr>
</table>

## 3\. 🤝 Swagger

<openapi src="./abaltusova_ekaterina.yaml" flag="true"/> 
 
###