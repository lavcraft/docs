# REST API для проведение платежей

_ Драфт документа от 14.09.10 _

# Endpoints

Тестовая среда с api:
> * https://testjmb.alfabank.ru/api/v1/fee
  * https://testjmb.alfabank.ru/api/v1/transfers

Тестовые среда для проведения авторизации:     
> * https://testjmb.alfabank.ru/api/oauth/token
  * https://testjmb.alfabank.ru/api/oauth/authorize

```
https://testjmb.alfabank.ru/api/v1/fee
|                          |   |  |   |
`--------------------------`---`--`---`-----`
               |             |   |          |
            base url        api  version     service
                          endpoint          endpoint
                          
https://testjmb.alfabank.ru/api/oauth/authorize
|                          |   |               |
`--------------------------`---`---------------`
               |             |          |
            base url        api        service
                          endpoint     endpoint
```
## Авторизация

Для авторизации необходимо получить логин-пароль через доверенное лицо. 
После чего можно получить токен для исполнения запросов в системе.

Сценарии:

1. Получение токена для пользователя, известного системе
2. Получение токена для пользователя известного лишь партнёру
3. Получение пароля пользователем по коду
4. Получение пользователем токена без участия партнёра

Сценарии 3-4 находятся в тестировании, докуменатция по ним не предоставлена.
 
### Авториация. Сценарий 1

У партнёра должен быть логин и пароль.

С помощью логина и пароля можно произвести Basic аутентификацию на сервере авторизации. После чего появляется возможность получить токен для исполнения операций.

Заголовки для авторизации:

```
Authorization Basic base64(login:password)
```



Получение токена в случае

## POST /fee

Вызывается для определения комиссии при платеже. CalculateFee в SV 
Запрос

```
HEADER: 
TYPE: POST
Content-Type: application/json
{
   "sender_type":"cnm",   
   "sender_value":"xxxxxxxxxxxxxxx",
   "recipient_type":"cnm",
   "recipient_value":"xxxxxxxxxxxxxxx",
   "amount":"1",
   "currency":"RUR"
}
```

Ответ

```
{
    "fee": "0",
    "interest": "-1",
    "constant": "-100",
    "min": "-100",
    "max": "-100"
}
```

Ошибка

```
{
    "error_code": "APE0302",
    "error_text": "APE0302"
}
```

## PUT /transfers

Вызывается для инициации платежа. Фаза AttemptTransfer в SV

Запрос

```
HEADER: 
TYPE: PUT
Content-Type: application/json
{
    "sender_type":"cnm",
    "sender_value":"xxxxxxxxxxxxxxx4285",
    "recipient_type":"cnm",
    "recipient_value":"xxxxxxxxxxxxxxx5864",
    "exp_date":"202501",
    "cvv":"xxx",
    "amount":"1",
    "currency":"RUR",
    "client_ip":"127.0.0.1"
}
```

Ответ

```
{
    "termURL": "http://a103145.moscow.alfaintra.net:7101/uapi10/transfers",
    "acsURL": "https://acs.alfabank.ru/acs/PAReq",
    "md": "3005008BFD2875B880F6E8AFA738036A",
    "pareq": "eJxlUttuozAQ/RXEe/GFS2g0cUVLquaBLsp2+1q5ME1YhUttKOl+/dqEpLvqg61zxuO5nBm4OdYH5wOVrtpm5TKPug42RVtWzW7l/nq6v4rdGwFPe4WY/sRiUCggQ63lDp2qXLm13nnMFZAnW3wXMEcSJpDHgZyp+aKKvWx6AbJ4v908ijCIo4UPZKZQo9qkgnHm+3y6fUrpBAIgp0doZI3iLtmm3F5Ovt6un3+kQCY7FO3Q9OpTRIEJeyYwqIPY932nl4SM4+jJw5vsWtXLg6cGMuLri0b1URVIcp6/9Eo2+g0VAWI/AvmqOx8s0ibRsSpFlibj6axplm7Gx3RzzH4nn9mf9QqI9YBS9ig4ZQGNaOQwvqTxMqBAJjvI2lYomFHghKCzCZKL+V8KRndlxnJu7swAj13boPEwWl8wlKiL7zo53dyBKcE6APlq6e7BDqbojcpXLKRReB3TIPYCvgj9ax7QOPJDGkZ2XJOTTVwZedmCsimzJUBsGDJvApm3xKD/tucvuCzKMA=="
}
```

Ошибка

```
{
    "error_code": "APE0302",
    "error_text": "APE0302"
}
```

## POST /transfers

Вызывается для завершения платежа после подтверждения на АБС банка эмитента карты. Фаза CompleteTransfer в SV
Вызывается либо вручную с Content-Type: application/json для завершение платежа, либо указывается в качестве termUrl при переходе на АБС страницу банка для ввода 3DS пароля
Пример вызова:
Запрос

```
HEADER: 
TYPE: POST
Content-Type: application/json
{
    "MD":"5218593D37AB7922A01E33DB39569A23",
    "PaRes":"..........."
}
```

Ответ:

```
{
    "transactionId":"213123"
    "postbackURL":"http://......."
}
```

Ошибка

```
{
    "error_code":"KSM2010",
    "error_text":"KSM2010 Checksumm \"FDSFSD does not exist on the database "
}
```

АБС банка дёргает с параметрами так:

> Данный функционал релизует редирект лишь для ограниченного числа партнёров. Производится тестирование

```
HEADER: 
TYPE: POST
Content-Type: application/x-www-form-urlencoded
MD="5218593D37AB7922A01E33DB39569A23&PaRes=eJzVWFmzo0iu......................
```

Пример ответа:

Редирект на страничку postbackUrl&transactionId=

Ошибка

Редирект на страничку postbackUrl+?error