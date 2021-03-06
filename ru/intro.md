# Введение

## Actor Model

Actor Model — довольно естественный способ программировать
параллельные программы. При этом подходе используются такие сущности
как Actor'ы. У Actor'а есть адрес. Actor может посылать другому
Actor'у сообщения. Сообщения для данного Actor'а падают в его mailbox,
откуда Actor их может получать.

Actor Model позволяет избегать проблем синхронизации, свойственной
потокам — вся синхронизация происходит на уровне mailboxes.

Больше об Actor Model можно прочитать, например, на [http://en.wikipedia.org/wiki/Actor_model](вики).

## Cocaine Messages and Actors

Сообщения, в терминах Кокаина, это msgpack-кодированные tuples следующего формата:
```
[message_type_id, session_id, [arguments...]]
```
Например, вызов метода `find` сервиса `storage` выглядел бы так:
```
[ 3, // id of find method
  123, // id of session
  [ "collection name", // collection name to look items in
    [ "tag1", "tag2" ] ] ] // tags
```

Actor, в терминах Кокаина, это программа-агент, слушающая входящие
соединеия на определенном адресе/порту, доступная для обнаружения через
Locator, и поддерживающая определенный протокол обмена
сообщениями. Actor'ы в кокаине называются сервисами.

## Cocaine RPC Protocol

В RPC протоколе используется msgpack — json-подобный двоичный формат
данных [msgpack.org](http://msgpack.org).

Протокол, по которому можно общаться с сервисами, такой.
У каждого сервиса есть таблица доступных для вызова методов. Например,
для сервиса Storage, эта таблица выгладит так:
```
+---+-----------+
|id |method name|
+---+-----------+
|0  |read       |
+---+-----------+
|1  |write      |
+---+-----------+
|2  |find       |
+---+-----------+
|3  |remove     |
+---+-----------+
```

чтобы вызвать метод `read`, для коллекции `collectionA`, и ключа `keyB`,
нужно в локаторе получить endpoint сервиса storage, законнектиться на
него, и отправить такой tuple кодированный msgpack'ом:  
```
[0,   // id метода read
 123, // client-generated random id
 ["collectionA", "keyB"]] // arguments to read method
```
Здесь 0 — id метода read из таблицы данного сервиса (в дальнейшем для
простоты записи, в сообщениях id методов будем обозначать
идентификатором (т.е. read), так что `[read, 123, ["coll", "key"]]`
будет обозначать `[0, 123, ["coll", "key"]]`.

Сервис, получив это сообщение, читает значение ключа `keyB` в коллекции
`collactionA`, и отвечает таким образом:
```
[chunk, 123, [<binary-buffer>]]
[choke, 123, []]
```
Здесь ответ состоит из двух сообщений: 
1. chunk, его id берется не из таблицы какого-либо сервиса, а из
специальной таблицы сообщений RPC, 123 — session id, ранее переданный
клиентом в вызове метода сервиса, и в tuple с аргументами передается
прочитанное значение ключа в виде бинарного буфера.
2. choke, который обозначает, что ответ завершился успешно.

Таким образом, вызов метода сервиса происходит сообщением
```
[method_id, session_id, [arguments..]]
```
Ответ на этот вызов выглядит как
```
[chunk, session_id, [<binary-buffer1>]]
[chunk, session_id, [<binary-buffer2>]]
[chunk, session_id, [<binary-buffer3>]]
[choke, session_id, []]
```
т.е. как поток из одного или более сообщений chunk, каждое их которых
несет кусок ответного потока данных, завершающийся сообщением choke. 

или 
```
[chunk, ...]
[chunk, ...]
[error, session_id, [<code>, <message>]]
```
поток чанков, завершающийся сообщением error с указанием кода ошибки и
строкового сообщения об ошибке.

Таблица RPC-сообщений выглядит так:
```
+--+----------+
|0 |handshake |
+--+----------+
|1 |heartbeat |
+--+----------+
|2 |terminate |
+--+----------+
|3 |invoke    |
+--+----------+
|4 |chunk     |
+--+----------+
|5 |error     |
+--+----------+
|6 |choke     |
+--+----------+
```

В одном сокете, открытом к сервису, может происходить обмен
сообщениями для многих сессий одновременно, при этом на клиента
ложится обязанность генерировать уникальные `session_id` в рамках
данного соединения. При этом, `session_id`, на которые полностью получен
ответный поток, разрешается реиспользовать.

Таким образом, процедура вызова метода сервиса и получения ответа
полностью выглядят так.

Клиент создает или использует ранее созданное
соединение к сервису. Отправляет сообщение с id метода из таблицы
методов сервиса, с уникальным в рамках данного открытого соединения
`session_id`, и аргументами данного вызова. На что сервис, после
обработки, отправляет ответный поток из нуля или более chunk'ов,
несущих бинарный payload с ответом, и завершает этот поток сообщением
choke, в случае успешного выполнения, или error, в случае какой-либо
ошибочной ситуации. Id ответных RPC-сообщений берутся из специальной
таблицы RPC-протокола аналогично id сообщений с именами методов
сервиса.

## Locator

Управление сервисами, назначение адресов, обмен информацией о
запущенных сервисах на нодах кластера происходят в cocaine-runtime. 
Чтобы узнать, на каком адресе находится определенный
сервис, используется специальный сервис Locator. Он всегда доступен на
каким-то образом заранее известном адресе.

Процедура обнаружения (resolve) выглядит так.

Клиент, который хочет получить доступ к сервису, например, storage,
вызывает метод resolve сервиса locator с аргументом `<имя-сервиса>`,
обычным Cocaine-RPC способом. Т.е. создает или использует ранее
созданное соединение на адрес:порт локатора, и отправляет такое
сообщение:
```
[resolve, <session-id>, ["storage"]]
```
на что локатор отправляет такой поток с ответом:
```
[chunk, <session-id>,
  [msgpack([
    ["host", 12345], // endpoint
    1,               // protocol version
    {0: "read",      // method table
     1: "write",
     2: "delete",
     3: "find"}])]]
[choke, <session-id>, []]
```

т.е. поток с одним чанком, в payload которого msgpack-упакован tuple
```
[<endpoint>, <version>, <method-table>]
```
и Клиент после этого может обращаться на адрес нужного сервиса, и
вызвать методы, пользуясь таблицей методов этого сервиса, отданной
локатором.


## Apps and Services

Сервисы, в нынешнем мире кокаина, это (trusted) сконфигурированные компоненты
cocaine-core или плагины, выполняющиеся в отдельном потоке или в пуле
потоков внутри процесса cocaine-runtime. Пользовательские приложения
— это сторонний для cocaine-runtime код, выполняющийся в том или ином
уровне изоляции (за это отвечает соответствующий плагин isolate).

Core services регистрируются непосредственно в локаторе при старте
рантайма, и сами непосредственно слушают и обрабатывают свои
сообщения.

Доставка сообщений в инстансы приложений выглядит несколько сложнее.

По комманде app start cocaine-runtime создает invocation сервис,
которому обычным для сервиса образом назначается порт, и присваивается
имя, совпадающее с именем приложения, по обычной процедуре создания
сервиса, и сервис начинает слушать на назначенном адресе.

Начиная с этого момента, клиент может в локаторе по имени приложения
получить адрес invocation сервиса для этого приложения и его таблицу
методов. Для invocation сервиса это всегда методы enqueue и info.

Клмент вызывает у соответствующего его приложению сервиса метод
enqueue, вот таким месседжем:
```
[enqueue, <session-id>, [<event-name>, <binary-buffer>]]
```
и, далее, invocation сервис получает enqueue сообщение, и в очередь
данного приложения отправляет такие три сообщения: 
```
[invoke, <session_id>, [<event-name>]]
[chunk, <session_id>, [<binary-buffer]]
[choke, <session_id>, []]
```
где `event-name` и `binary-buffer` берутся из enqueue-сообщения.

Engine приложения передает эти три сообщения соответствующему инстансу
приложения через pipe.

Приложение получает inovke, chunk и choke для данной сессии,
обрабатывает их в хэндлере соответствующего event'а, и отвечает в пайп
рантайму стандартным RPC ответом, т.е. потоком чанков, завершающимся
сообщением choke или error.

Engine приложения проксирует эти сообщения обратно соответствующему
клиенту.
