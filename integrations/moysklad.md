|     Ответ интеграции                       |     Действия   системы                                                                                    |     Ответ   вызывающей стороне    |   |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------|-----------------------------------|---|
|     200 OK {meta: {href: "URL товара"}}    |     products.moysklad_product_href =   href, products.status = 'active'                                   |     201 Created                   |   |
|     409 Conflict                           |     Удаление   записи о товаре в products                                                                 |     422 Unprocessable   Entity    |   |
|     503 Service   Unavailable              |     products.status   = 'pending'; публикация отложенного сообщения (delay = 1 мин, retry_count =   0)    |     202 Accepted                  |   |
|                                           
