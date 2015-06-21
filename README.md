# Yet Another Unpacker for 1C:Enterprise to Sync Depot with Git for Code Review

Консольный синхронизатор хранилища 1С и git

* требует установленного 1Script версии => 1.0.9.100 [Официальный репозиторий](https://bitbucket.org/EvilBeaver/1script/wiki/Home) 

## Порядок использования

* установите 1Script, чтобы oscript.exe оказался доступен В %PATH%
* необходимо скопировать v8unpack.exe в директорию с установленным 1Script
* заполнить профиль интеграции (или использовать переменные окружения в случае серверов сборок)
* запустить в первый раз
  * настроить файл авторов с использованием корректных имени и фамилии, а также валидного e-mail адреса
  * указать начальную версию для интеграции (учитывайте что самая большая конфигурация ERP 2.0 выгружается около 15 минут)
* выполнить фейковую инициализацию для корректного расчета статистики и последующего **code climate**

## Решаемые бизнес-задачи

* контроль кода внутри команды

![Code Review](https://habrastorage.org/files/a79/8d7/348/a798d734830d48738770f0956d34e2e7.png)

* связь задач ALM и кода для прохождения аудита по CMMI, Due Diligence, ГОСТ и т.д.

![Issue Relates](https://habrastorage.org/files/753/dc1/58c/753dc158c7c44914853a249204cc1634.png)

* Agile при разработке на 1С, контроль ответственности за объекты 1С и прочее, прочее, прочее

## Настройка профиля

для начала необходимо настроить профиль публикации

```
cp profile.yml.example my-repo.yml
```

в профиле укажите

```
storage_dir: \\depot-server\depot-erp-20\main\ # Папка с хранилищем 1С
git_local_repo: d:\\git-for-erp\ # локальный репозиторий, в которой будет склонирован удаленный репозиторий
v8_executable: C:\Program Files (x86)\1cv8\8.3.6.2041\bin\1cv8.exe # прямой адрес к исполняемому файлу платформы 1С версии старше 8.3.6.*
git_executable: C:\Program Files (x86)\Git\bin\git.exe # прямой адрес установки исполняемого файла git (MSGit для Windows)
git_remote_repo: file:///d:/repos/erp-20/ # действующий адрес bare репозитория (об авторизации на удаленном репозитории вам придется позаботиться отдельно) 
git_email_domain: example.com # домен команды имеющей доступ в хранилище, будет использован для попытки автоматической генерации e-mail адресов) 
unpack_dir: d:\git-for-erp\src\ # каталог для распаковки 
branch: master # ветка в которую будет помещаться исходники (в случае наличия нескольких хранилищ одной конфигурации используются соответвенно имена develop, hotfix и т.д.)
```

## Порядок первого запуска

```
oscript.exe vanessa-unpack.os ./my-repo.yml
```

В момент первого запуска выполняется попытка склонировать удаленный репозиторий, так как он чаще всего пустой, то почти всегда вы увидите ошибку об отсутствии головной ревизии.
поэтому в этот момент необходимо выполнить установочную команду 

> пример bat файла для инициализации

```
@echo Перейдем локальный репозиторий из профиля
cd d:\\git-for-erp\ 
set GIT_AUTHOR_DATE="Mon, 1 Jan 2001 04:00:00 +0300"
set GIT_COMMITTER_DATE="Mon, 1 Jan 2001 04:00:00 +0300"
git add -A
git commit -a -m "Start integration with dirty date" --author="My Name <me@example.com>"
git push
@echo Создадим дополнительный подкаталог
mkdir d:\git-for-erp\src\ 
```

в данном файле присутствуют 2 "магические установки" которые позволяют инициализировать репозиторий раньше, чем любая возможная версия хранилища 1С


## Порядок второго запуска

для окончательной настройки необходимо выполнить повторно команду

```
oscript.exe vanessa-unpack.os ./my-repo.yml
```

во время второго запуска произойдет

* выгрузка истории хранилища в виде XML файла
* выгрузка всех авторов с их именами пользователей

файлы будут выгружены по указанному вами адресу **unpack_dir** 

для полноценного использования необходимо

* указать начальную версию в файле VERSION в каталоге **unpack_dir**

```
<?xml version="1.0" encoding="UTF-8"?>
<VERSION>Текущая версия - 100 версий назад</VERSION>

```


* указать соответствия авторов в файле AUTHORS в каталоге **unpack_dir**

```
Ваня=Ivan Kojedub <ikojedub@examle.com>
Маша=Maria Nesterenko <mnesterenko@example.com>
```

## Автоматизация последующих запусков

* для владельцев Windows Server - taskshd.msc: простой запуск через планировщик (рекомендуемое время перезапуска от 15 до 30 минут)
* для владельцев CI серверов - советуем использовать Jenkins с плагином автоматической подписки на изменения файла 1CD (файла базы хранилища)
* для владельцев linux - cron: единый способ автоматизации рутинных операции с настройкой повторяемости

> Работа в linux полностью поддерживается через wine (wine oscript.exe ...)

## Разбор проблем

```
oscript.exe vanessa-unpack.os ./my-repo.yml -debug
```
в этом режиме будут выводится отладочные сообщения, для полноценного разбора нештатных ситуаций

## История "синхронизатора" и контрибьюция

* в 2012 году проект развивался в виде обработки v83unpack https://github.com/pumbaEO/v83unpack
* в 2013 году проект представлял собой конфигурацию 1С для интеграции множества хранилищ 1С https://github.com/xDrivenDevelopment/v83unpack 
* в 2014 году проект был полностью портирован на 1Script силами https://bitbucket.org/EvilBeaver

## Особенности

* текстовые модули имеют расширение **bsl**
* текущая версия адаптирована под системы управления проектами OpenProject/Redmine

 


