|     Ответ интеграции                       |     Действия   системы                                                                                    |     Ответ   вызывающей стороне    |   |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------|-----------------------------------|---|
|     200 OK {meta: {href: "URL товара"}}    |     products.moysklad_product_href =   href, products.status = 'active'                                   |     201 Created                   |   |
|     409 Conflict                           |     Удаление   записи о товаре в products                                                                 |     422 Unprocessable   Entity    |   |
|     503 Service   Unavailable              |     products.status   = 'pending'; публикация отложенного сообщения (delay = 1 мин, retry_count =   0)    |     202 Accepted                  |   |
|                                           


|     Результат   вызова                               |     Действия   системы                                                                                       |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
|     200 OK {meta: {href: "URL товара"}}              |     products.status = 'active',   moysklad_product_href = href                                               |
|     409 Conflict                                     |                                                                                                              |
|     503 Service   Unavailable, retry_count < 5       |     retry_count   += 1; публикация отложенного сообщения с задержкой min(1 мин × 2^retry_count,   30 мин)    |
|     503 Service   Unavailable, retry_count >=   5    |     products.status = 'failed'; публикация события для   уведомления продавца                                |
|                                                      |                                                                                                              |
|                                                      |                                                                                                              |
|                                                      |                                                                                                              |
|                                                      |                                                                                                              |
|                                                      |                                                                                                              |
|                                                      |                                                                                                              |
|                                                      |                                                                                                              |
