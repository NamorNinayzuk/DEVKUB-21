
# Домашнее задание по лекции "11.2 Микросервисы: принципы"

> Вы работаете в крупной компанию, которая строит систему на основе микросервисной архитектуры.
> Вам как DevOps специалисту необходимо выдвинуть предложение по организации инфраструктуры, для разработки и эксплуатации.

## Обязательная задача 1: API Gateway

> Предложите решение для обеспечения реализации API Gateway. Составьте сравнительную таблицу возможностей различных программных решений. На основе таблицы сделайте выбор решения.
> 
> Решение должно соответствовать следующим требованиям:
> - Маршрутизация запросов к нужному сервису на основе конфигурации
> - Возможность проверки аутентификационной информации в запросах
> - Обеспечение терминации HTTPS

Честно эта задача из разряда: Пойди туда, низная куда, найди то, низная чего! (с)
Это я к тому что невозможно сделать точное и качественное оценночное суждение без перечисления конкретных требований! Так и выходит что де факто подходят любые продукты у которых есть определённые задокументированные возможности. Под определенные задачи нужны тесты, поэтому мой критерий отбора проходит "Как есть!"


Характеристика | [SberCloud API Gateway](https://sbercloud.ru/ru/products/api-gateway) | [Yandex API Gateway](https://cloud.yandex.ru/docs/api-gateway/) | [nginx](https://nginx.org/ru/) | [HAProxy](https://www.haproxy.org/) | [Kong Gateway](https://docs.konghq.com/gateway/latest/) | [KrakenD](https://www.krakend.io/open-source/) | [Tyk](https://github.com/TykTechnologies/tyk)
--- | --- | --- | --- | --- | --- | --- | ---
Маршрутизация на основе конфигурации | Да | OpenAPI 3.0 | Файлы настроек | Файлы настроек | Файлы настроек | GUI + схема | OpenAPI 2/3
Проверка аутентификационной информации в запросах | Да | Да | Да | Да | Да | Да | Да
Обеспечение терминации HTTPS | Да | Да | [Да](https://www.8host.com/blog/nastrojka-balansirovki-nagruzki-nginx-i-ssl-terminacii/) | [Да](https://www.haproxy.com/blog/haproxy-ssl-termination/) | Да | Да | Да
Стоимость | платно | платно | бесплатно | бесплатно | бесплатно | бесплатно | бесплатно


Судя по форумам и туториалам наиболее популярным решением является **Kong**. Исходя из этого, скорее всего, у него наиболее большое сообщество и наибольшее количество различных инструкций и мануалов в интернете. Но! `KrakenD` создан с целью на производительность (до 7 раз быстрее **Tyk** и до 2 раз быстрее **Kong**), с его низким потребление памяти (100 MB на высоконагруженных нодах), **Stateless** ) и другими интересными фундервафлями, понятную (siс!) документацию и возможность установки через образ **Docker**.
 

---

## Обязательная задача 2: Брокер сообщений

> Составьте таблицу возможностей различных брокеров сообщений. На основе таблицы сделайте обоснованный выбор решения.
> 
> Решение должно соответствовать следующим требованиям:
> - Поддержка кластеризации для обеспечения надежности
> - Хранение сообщений на диске в процессе доставки
> - Высокая скорость работы
> - Поддержка различных форматов сообщений
> - Разделение прав доступа к различным потокам сообщений
> - Простота эксплуатации

Так как и в этой задачи - нет количественных критериев оценки решения, то подходящими будут являться удовлетворяющие всем требованиям:

Характеристика | [Yandex Message Queue](https://cloud.yandex.ru/docs/message-queue/) | [Apache Kafka](https://kafka.apache.org/) | [RabbitMQ](https://www.rabbitmq.com/) | [Redis](https://redis.io/)
--- | --- | --- | --- | ---
Поддержка кластеризации | Да | Да | [Да](https://www.rabbitmq.com/clustering.html) | [Да](https://redis.io/docs/manual/scaling/)
Хранение сообщений на диске в процессе доставки | [Да](https://cloud.yandex.ru/blog/posts/2019/06/message-queue) | [Да](https://habr.com/ru/company/southbridge/blog/535374/) | [Да](https://www.rabbitmq.com/persistence-conf.html) | [Частично](https://redis.io/docs/manual/persistence/)
Высокая скорость работы | [Существенные ограничения](https://cloud.yandex.ru/docs/message-queue/concepts/limits) | Да | Да | Да
Поддержка различных форматов сообщений | Да | Да | Да | Да
Разделение прав доступа к различным потокам сообщений | [Да](https://cloud.yandex.ru/docs/message-queue/security/) | [Да](https://habr.com/ru/company/nsd/blog/661007/) | [Да](https://www.rabbitmq.com/access-control.html) | [Да](https://redis.io/docs/manual/security/acl/)
Простота эксплуатации | Относительно | [Сложно](https://habr.com/ru/company/parimatch_tech/blog/591827/) | [Относительно](https://habr.com/ru/post/434016/) | Простая


### Решение

Выбираю **Redis**, просто и надежно.

## Задача 3: API Gateway * (необязательная)

### Есть три сервиса:

**minio**
- хранит загруженные файлы в бакете images,
- S3 протокол,

**uploader**
- принимает файл, если картинка сжимает и загружает его в minio,
- POST /v1/upload,

**security**
- регистрация пользователя POST /v1/user,
- получение информации о пользователе GET /v1/user,
- логин пользователя POST /v1/token,
- проверка токена GET /v1/token/validation.

### Необходимо воспользоваться любым балансировщиком и сделать API Gateway:

**POST /v1/register**
1. Анонимный доступ.
2. Запрос направляется в сервис security POST /v1/user.

**POST /v1/token**
1. Анонимный доступ.
2. Запрос направляется в сервис security POST /v1/token.

**GET /v1/user**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис security GET /v1/user.

**POST /v1/upload**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис uploader POST /v1/upload.

**GET /v1/user/{image}**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис minio GET /images/{image}.

### Ожидаемый результат

Результатом выполнения задачи должен быть docker compose файл, запустив который можно локально выполнить следующие команды с успешным результатом.
Предполагается, что для реализации API Gateway будет написан конфиг для NGinx или другого балансировщика нагрузки, который будет запущен как сервис через docker-compose и будет обеспечивать балансировку и проверку аутентификации входящих запросов.
Авторизация
curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/token

**Загрузка файла**

curl -X POST -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' -H 'Content-Type: octet/stream' --data-binary @yourfilename.jpg http://localhost/upload

**Получение файла**
curl -X GET http://localhost/images/4e6df220-295e-4231-82bc-45e4b1484430.jpg

---




# Как запускать
После написания nginx.conf для запуска выполните команду
```
docker-compose up --build
```

# Как тестировать

## Login
Получить токен
```
curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/token
```
Пример
```
$ curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/token
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I
```

## Test
Использовать полученный токен для загрузки картинки
```
curl -X POST -H 'Authorization: Bearer <TODO: INSERT TOKEN>' -H 'Content-Type: octet/stream' --data-binary @1.jpg http://localhost/upload
```
Пример
```
$ curl -X POST -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' -H 'Content-Type: octet/stream' --data-binary @1.jpg http://localhost/upload
{"filename":"c31e9789-3fab-4689-aa67-e7ac2684fb0e.jpg"}
```

 ## Проверить
Загрузить картинку и проверить что она открывается
```
curl localhost/image/<filnename> > <filnename>
```
Example
```
$ curl localhost/images/c31e9789-3fab-4689-aa67-e7ac2684fb0e.jpg > c31e9789-3fab-4689-aa67-e7ac2684fb0e.jpg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 13027  100 13027    0     0   706k      0 --:--:-- --:--:-- --:--:--  748k

$ ls
c31e9789-3fab-4689-aa67-e7ac2684fb0e.jpg
```
