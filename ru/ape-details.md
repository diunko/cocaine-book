
# APE.sh Details

## Терминология.

Контейнер приложения -- докер-контейнер, полностью готовый к запуску
на нодах кластера.

Базовый контейнер -- заранее созданный контейнер с предустановленными
buildpacks.

Buildstep -- Репозоторий, который содержит код сборки базового
контейнера, а также generic код запуска сборки приложения,
использующий функциональность, предоставляемую `buildpack`'ами.

Buildpack -- пакет с кастомным кодом, который строит (компилирует)
собирает и подготавливает к запуску определенный тип приложения
(напр., buildpack-nodejs, buildpack-python)

## Обзор

Полный процесс билда сейчас выглядит так. Заранее создается базовый
контейнер (сейчас это ubuntu precise), куда устанавливаются системные
deb пакеты и клонируются все buildpack'и.

При сборке коткретного приложения, результат git archive
приложения распаковывается в контейнер, выполняются тесты из каждого
билдпака на тип приложения, и выполняется compile-скрипт определенного
билдпака. Compile-скрипт делает все необходимое, чтобы собрать и
установить все системные и project-specific зависимости, и обеспечить
готовность приложения к запуску в облачной среде.


## buildstep (basic container build script)

`Dockerfile` основан на `ubuntu:precise`, добавляет sources.list и ключи с
подписями, монтирует `build` в `/build` и вызывает `/build/prepare`.

`/build/prepare` устанавливает deb-пакеты, перечисленные в
`build/packages.txt`, и клонирует репозитории с билдпэками, перечисленные в
`build/buildpacks.txt` в `/build/buildpacks/`.

Скрипт `/build/builder` выполнится при сборке приложения, и обеспечит
вызов `bin/compile` из подходящего `buildpack`'а.

## heroku-buildpack-nodejs

`bin/detect` возвращает 0 exit code, когда находит package.json в
приложении. Такой сейчас способ определить, что приложение -- nodejs.

`bin/compile` 

1. получает из package.json версию ноды и main скрипт.
1. устанавливает deb-пакеты, перечисленные в `packages.txt` 
1. скачивает и устанавливает нужную версию nodejs (из s3.herokurepo --
как то изменить это)
1. выполняет `npm install --production --userconfig .npmrc`
1. TODO описать логику кеширования npm-модулей.
1. создает `/build/app/.profile.d/node.sh`, в котором в `PATH`
добавлен путь к нужной версии ноды.
1. копирует `/build/app` в `/app`
1. создает `/start` скрипт, которым будет запускаться воркер
приложения. Этот скрипт сорсит `/app/.profile.d/*`, где лежит ранее
созданный `node.sh` и, возможно, пользовательские скрипты.


## ape
Это простая обертка над cocaine-tool, обеспечивающая аутентификацию и
управление приложениями в облаке. Должна быть заменена на админку с
REST-api, cli-клиентом и веб-интерфейсом. Позволяет сравнительно
просто выполнять базовые операции с приложениями в облаке.

### Обзор

#### Авторизация

Для авторизации всех пользовательских действий используется следующая
возможность sshd (сокращенно описание из `man authorized_keys`):

```
AuthorizedKeysFile specifies the files containing public keys for
public key authentication; ... the default is ~/.ssh/authorized_keys
... Each line of the file contains one key ... [and the] following
space-separated fields: options, keytype, base64-encoded key,
comment ...
```

и из описания полей опций:
```
command="command"
  Specifies that the command is executed whenever this key is used for
  authentication. The command supplied by the user (if any) is ignored.
  ...  This option might be useful to restrict certain public keys to
  perform just a specific operation.  An example might be a key that
  permits remote backups but nothing else.  Note that the client may
  specify TCP and/or X11 forwarding unless they are explicitly
  prohibited.  The command originally supplied by the client is
  available in the SSH_ORIGINAL_COMMAND environment variable ...
```

Все пользователи используют аккаунт ape, и его
`.ssh/authorized_keys`. Строчки в этом файле выглядят так:

```
command="FINGERPRINT=7d:27:ec:e3:14:a0:eb:54:80:56:cf:be:fb:ae:74:fa \
  LOGIN=diunko `cat /home/ape/.sshcommand` $SSH_ORIGINAL_COMMAND",\
  no-agent-forwarding,no-user-rc,no-X11-forwarding,no-port-forwarding \
  ssh-rsa <base64-encoded-key> some@email
```

Таким образом, ssh-сессия на пользователя ape с ключом
<base64-encoded-key>, приведет к выполнении команды 
```
"FINGERPRINT=7d:27:ec:e3:14:a0:eb:54:80:56:cf:be:fb:ae:74:fa LOGIN=diunko `cat /home/ape/.sshcommand` $SSH_ORIGINAL_COMMAND"
```
где в `SSH_ORIGINAL_COMMAND` будут аргументы, переданные пользователем
команде ssh.

В `/home/ape/.sshcommand` написано `/home/ape/bin/ape`,
т.е. effectively команда выполнится так:
```
"FINGERPRINT=7d:27:ec:e3:14:a0:eb:54:80:56:cf:be:fb:ae:74:fa LOGIN=diunko /home/ape/bin/ape $SSH_ORIGINAL_COMMAND"
```

#### Доступные действия
Результат всех действий namespaced по имени пользователя, их
выполняющего, что позволяет избежать различных коллизий.

`git push` На git push в ape@dockerhost:appname создается репозиторий appname
(namespaced by $LOGIN, в описании других действиях это напоминание
повторять не будем). В репозитории создается prereceive-хук.
Хук выполнит вызов сборки приложения, и вернет non-zero exit code при
неуспешной сборке, что приведет к неуспеху апдейта рефов в репозитории, и
позволит пользователю увидеть ошибки сборки, починить их и сделать
повторный push.

`status` Команда status показывает суммарный статус приложений пользователя:
все загруженные приложения, запущенные, и настройки роутинг-груп.
`start` Запустить определенную версию приложения.
`stop` Остановить определенную версию приложения.
`delete` удалить версию приложения.
`adduser` добавить ключ пользователя в `authorized_keys`.


Рассмотрим работу скрипта подробнее.

### git-*
git push effectively выполняет команду наподобие 
```
git-archive | ssh ape@remote.host git-receive-pack
```
Аргумент `git-receive-pack` попадает в ветку `case git-*)`, которая
создает репозиторий и в нем prereceive-хук, вызывающий `ape git-hook`

### git-hook
`git-hook`, если получаемая ветка -- master, вызывает 
`git archive | ape build app-name <hash>`.

### build
Запускает базовый контейнер, распаковывает в нем в `/app` приложение,
и вызывает в контейнере `/build/builder /app /cache`. Билдер собирает
приложение. Контейнер комитится (`docker commit`) с именем
`<dockerregistry>/user_did_app_at_<hash>`, и пушится в репозиторий
(`docker push`).
Создается роутинг-группа с именем `user_did_app`:
```
cocaine-tool group create user_did_app
```
Создается кокаин-приложение (значимы здесь манифест и имя, архив нужен
потому, что это такой обязательный аргумент в cocaine-tool)
```
cocaine-tool app upload --name user_did_app_at_<hash> --manifest='{"slave":"/start"}' --package /home/ape/app.tgz
```

### start
На каждой ноде выполняется 
```
cocaine-tool app start --name user_did_app_at_<hash>
```
Версия приложения добавляется в ранлист и роутинг группу `user_did_app`.

### stop
Действия, обратные `stop`

### delete
Приложение и манифест удаляются из стореджа
`cocaine-tool app remove --name user_did_app_at_<hash>`

и из ранлиста
```
cocaine-tool runlist remove-app \
  --name user_did_app \
  --app user_did_app_at_<hash>
```
### status

Показывет вывод от `cocaine-tool info`, `cocaine-tool runlist view`,
`cocaine-tool app list`, отфильтрованный по логину пользователя.

