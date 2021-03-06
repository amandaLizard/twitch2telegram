Чтобы запустить бота, нужно пройти пять этих пунктов
  1. Linux
  2. Аккаунт телеграм бота
    2.1. Добавить телеграм бота в нужный чат
    2.2. Узнать идентификатор чата телеграмма
    2.3. Прописать идентификатор телеграмм чата в конфигурационном файле
  3. Сделать oauth2 токен на твиче
    3.1 Добавить список каналов для мониторинга
  4. Изменить фильтр для отслеживаемых твич пользователей
  5. Запуск бота


---

1. Linux

Пойдет любой линукс на который можно установить docker и docker-compose

Тестировалось на ubuntu 20.04

Docker version: 19.03.8

Docker compose version: 1.29.2

Клонируем репозиторий командой:

```
git clone https://github.com/amandaLizard/twitch2telegram.git
```

---


2. Создать аккаунт для телеграм бота


Мы должны получить строку вроде этой: `5321242921:FAG5RZ1sXsfxj5LmsGdm2N8uRLGp14gqJgz`

2.1. В телеграмм клиенте, приглашаем бота по имени бота в требуемый телеграмм чат

2.2. У требоемого телеграмм канала нужно узнать ID, он выглядит примерно так: `-1101273091133`


2.3 ID телеграмм чата и аккаунт для телеграмм бота нужно прописать в файл `docker-compose.yml` (строчк 12-13)

```
  TELEGRAM_TOKEN: "5321242921:FAG5RZ1sXsfxj5LmsGdm2N8uRLGp14gqJgz"
  TELEGRAM_CHAT_ID: "-1101273091133"
```
---

3. Сгенерировать oauth токен для твича.

Инструкция: https://dev.twitch.tv/docs/authentication/getting-tokens-oauth#oauth-authorization-code-flow

Нужный oauth токен выглядит примерно так: "oauth:3qllxxxi46ykvitfds7se54nurc138j"

Этот токен нужно прописать в файле `irssi_config`, строчка 6:

```
    ...
    password = "oauth:3qllxxxi46ykvitfds7se54nurc138j";
    ...
```

---

3.1 Добавить список каналов для мониторинга

В конце файла `irssi_config` есть секция с каналами, которые бот будет мониторить

Выглядит это так:

```
channels = (
  { name = "#zapahzamazki"; chatnet = "Twitch"; autojoin = "Yes"; },
);
```

Чтобы добавить новый канал, нужно скопировать уже существующую строчку и поменять имя канала (знак `#` решетки - обязательный)

Допустим, добавим канал silvername, это будет выглядеть так:


```
channels = (
  { name = "#zapahzamazki"; chatnet = "Twitch"; autojoin = "Yes"; },
  { name = "#silvername"; chatnet = "Twitch"; autojoin = "Yes"; },
);
```

4. Изменить фильтр для отслеживаемых твич пользователей

В файле `logstash.conf` есть строчки (32-34)

```
    if ([username] !~ /(?i)Ivan/ and [username] !~ /(?i)Maria/)  {
      drop {}
    }
```

Отслеживаются твич пользователи Ivan и Maria (регистр букв не важен)

Если нужно добавить еще пользователя, например John, то фильтр будет выглядеть так:

```
    if ([username] !~ /(?i)Ivan/ and [username] !~ /(?i)Maria/ and [username] !~ /(?i)John/ )  {
      drop {}
    }
```

---

5. Запуск бота

Если бот запускается первый раз, нужно выполнить команду:

```
docker-compose pull
```

Запускаем бота:

```
docker-compose up
```

Через некоторое время, через минуту, проверяем что все запустилось корректно

```
docker-compose ps
```

Вывод команды должен быть такой:

```
twitch2telegram_irssi_1      irssi --config /root/.irss ...   Up      
twitch2telegram_logstash_1   /usr/local/bin/docker-entr ...   Up      
```

---


---
Теперь все сообщение от пользователей из пункта 4, написавших сообщение в твич чатах из пункта 3.1 будут пересылваться в телеграмм чат из 2.1 

Вот и все, чтобы добавить канал, или изменить другую настройку потребуется перезапуск

Чтобы перезапустить ботов, нужно выполнить команду

```
docker-compose restart
```

---
