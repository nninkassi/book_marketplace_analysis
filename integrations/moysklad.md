МойСклад – облачная ERP-система для малого и среднего бизнеса. Интеграция с ней покрывает следующие сценарии: синхронизация карточки товара при добавлении товара продавцом, резерв товаров под заказ, подтверждение резерва после успешной оплаты и отмена резерва при ошибке оплаты, создание отгрузки при передаче курьеру. 

M-1. Синхронизировать карточку товара
Инициатор: Catalog API; при повторных попытках — Sync Worker
Приемник: МойСклад
Частота взаимодействия: по событию POST /api/products; повторные попытки по отложенному сообщению брокера при products.status = 'pending'
Характер взаимодействия: синхронно
Условия вызова: успешная валидация данных товара в Catalog API
Метод и URL: POST https://api.moysklad.ru/api/remap/1.2/entity/product
Тело запроса:
```json
{
  "syncId": "products.idempotance_key",
  "name": "products.title",
  "article": "products.products_sku",
  "description": "products.description",
  "salePrices": [
    {
      "value": "products.price * 100",
      "currency": {
        "meta": {
          "href": "STORED_CURRENCY_HREF",
          "type": "currency",
          "mediaType": "application/json"
        }
      }
    }
  ]
}
```

Обработка ответов:

|     Ответ интеграции                       |     Действия   системы                                                                                    |     Ответ   вызывающей стороне    |   |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------|-----------------------------------|---|
|     200 OK {meta: {href: "URL товара"}}    |     products.moysklad_product_href =   href, products.status = 'active'                                   |     201 Created                   |   |
|     409 Conflict                           |     Удаление   записи о товаре в products                                                                 |     422 Unprocessable   Entity    |   |
|     503 Service   Unavailable              |     products.status   = 'pending'; публикация отложенного сообщения (delay = 1 мин, retry_count =   0)    |     202 Accepted                  |   |
|                                           

Механизм повторных вызовов (retry):
|     Результат   вызова                               |     Действия   системы                                                                                       |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
|     200 OK {meta: {href: "URL товара"}}              |     products.status = 'active',   moysklad_product_href = href                                               |
|     409 Conflict                                     |                                                                                                              |
|     503 Service   Unavailable, retry_count < 5       |     retry_count   += 1; публикация отложенного сообщения с задержкой min(1 мин × 2^retry_count,   30 мин)    |
|     503 Service   Unavailable, retry_count >=   5    |     products.status = 'failed'; публикация события для   уведомления продавца                                |
|

Соответствие полей: 
|     Поле запроса к МойСклад             |     Тип данных Мойсклад    |     Поле в БД          |     Тип данных в БД    |     Описание поля                              |
|-----------------------------------------|----------------------------|------------------------|------------------------|------------------------------------------------|
|     syncId                              |     UUID                   |     idempotance_key    |     UUID               |     Обеспечивает идемпотентность   запросов    |
|     name                                |     String(255)            |     title              |     varchar (200)      |                                                |
|     article                             |     String(255)            |     products_sku       |     varchar (200)      |                                                |
|     description                         |     String(4096)           |     description        |     varchar(4096)      |                                                |
|     salePrices[0].value                 |     Float                  |     price * 100        |                        |                                                |
|     salePrices[0].currency.meta.href    |     Meta                   |     -                  |     -                  |     UUID валюты из конфигурации                |
|     salePrices[0].priceType             |     Meta                   |     -                  |     -                  |     UUID типа цены                             |





