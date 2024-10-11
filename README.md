# ЯндексMQ SDK для 1С:Предприятия

Библиотека для работы с [брокером сообщений Яндекс Message Queue](https://yandex.cloud/ru/services/message-queue) из 1С:Предприятия. 
Позволяет разработчикам на платформе 1С:Предприятие создавать программы, взаимодействующие с облачным брокером сообщений ЯндексMQ (YMQ).

## Быстрый старт

```bsl
// для работы с ЯндексMQ обязательно требуются AccessKey и SecretKey.
// Это своеобразные логин и пароль для доступа к сервису.
// как их получить описано на https://yandex.cloud/ru/docs/iam/operations/sa/create-access-key
AccessKey = "...";
SecretKey = "...";
ЯндексMQ  = Обработки.ЯндексMQ.НовыйКлиент( AccessKey, SecretKey );

// создадим новую очередь
АдресОчереди = ЯндексMQ.CreateQueue( "queue-name" );

// положим сообщение в очередь
ЯндексMQ.SendMessage( АдресОчереди, "Это текст сообщения" );

// получим сообщения из очереди
Сообщения = ЯндексMQ.RecieveMessage( АдресОчереди );

// обработаем все полученные сообщения
Для Каждого ДанныеСообщения Из Сообщения Цикл
    // КакТоОбработать( ДанныеСообщения.Body );
    // успешно обработанные - удалим из очереди
    ЯндексMQ.DeleteMessage( АдресОчереди, ДанныеСообщения.ReceiptHandle );
КонецЦикла;
```
## Как установить

Скачайте файл конфигурации (.cf) [в разделе "Releases"](https://github.com/leemuar/yandexmq-sdk-1c/releases) и перенесите из него обработку "ЯндексMQ" в свою конфигурацию.
Рекомендуется переносить обработку напрямую из файла конфигурации, не сохранять ее как внешнюю: обработка конфигурации содержит модуль менеджера с удобным методом-фабрикой, при сохранении обработки как внешней  модуль менеджера теряется

## Как использовать

Библиотека использует объектный подход к своей работе. Для начала работы необходимо получить объект библиотеки. В модуле менеджера обработки есть готовый метод-фабрика для этого:

```bsl
// для работы с ЯндексMQ обязательно требуются AccessKey и SecretKey.
// Это своеобразные логин и пароль для доступа к сервису.
// как их получить описано на https://yandex.cloud/ru/docs/iam/operations/sa/create-access-key
AccessKey = "...";
SecretKey = "...";
ЯндексMQ  = Обработки.ЯндексMQ.НовыйКлиент( AccessKey, SecretKey );
```

После создания объекта клиента можно вызывать его методы, выполняющие запросы к сервису очередей:

```bsl
АдресОчереди = ЯндексMQ.CreateQueue( "queue-name" );
// положим сообщения в очередь
ЯндексMQ.SendMessage( АдресОчереди, "Это текст сообщения" );
```

Библиотека предлагает 2 набора методов: простые и детальные. 

Простые стараются сделать максимум за вас: выполнить запрос и вернуть результат в удобном виде. Они возвращают **результат** операции: успешно или нет, данные сообщения или неопределено и т.п. Например, ```DeleteQueue()``` вернет ```Истина``` в случае успешного удаления, и ```Ложь``` - в случае ошибок. Или ```CreateQueue()``` возвращает строку URL созданной очереди в случае успеха или ```Неопределено``` - при ошибке. Если ошибка при запросе все же произойдет - простые методы не дадут детальной информации почему. Здесь могут пригодиться детальные методы.

Детальные методы заканчиваются на Response: CreateQueueResponse(), PurgeQueueResponse() и пр. Эти методы возвращают детальные **данные ответа** на запрос: код ответа http, тело запроса, заголовки http и т.п. Их можно детально разобрать, залогировать или написать более детальную обработку ответа самостоятельно:

```bsl
Ответ = ЯндексMQ.DeleteQueueResponse( QueueUrl );
Если 200 <> Ответ.КодСостояния Тогда
    ЛогОшибка( Ответ.Заголовки, Ответ.Тело, Ответ.КодСостояния );
КонецЕсли;
```

Тело сообщения будет в виде двоичных данных. Можно использовать вспомогательные функции библиотеки для преобразования двоичных данных во что-то другое, например json в структуру:

```
Ответ = ЯндексMQ.GetQueueAttributesResponse( QueueUrl );
Данные = ЯндексMQ.КакJson( Ответ );
```

## Исключения

Методы библиотеки не перехватывают исключения. Считается, что библиотека не должна решать внутри себя такие ситуации как таймауты, отсутствие связи, некорректный адрес и др. Все это внештатные, исключительные итуации, на которые должно быть брошено исключение, а не возвращено какое-то значение. Всегда оборачивайте вызов методов в Попытку.
