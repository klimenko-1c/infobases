# infobases
HTTP-сервис получения списка информационных баз 1С 

---

Порядок установки рассмотрен для Linux Ubuntu. Впрочем он легко адаптируется под MacOS и Windows.

## Подготовка сервера

Предполагается следующее:
 - у вас есть машина с Linux Ubuntu
 - на ней установлен сервер 1С
 - на ней установлен apache (см. ниже)
 - на ней установлена JRE 11 (см. ниже)
 - запущен RAS
 - есть скаченный архив Исполнителя по пути `~/Executor_2020_2_0_547.zip`
 
### Apache

Проверить, установлен ли, можно перейдя в браузере по [http://localhost/](http://localhost/), где вместо `localhost` - имя машины (допускается ip адрес; можно использовать `locahost` (или `127.0.0.1`), если непосредствено с нее же проверять)

Из терминала можно использовать `curl`:
```
curl http://localhost/
```

[Официальное руководство по установке Apache 2](https://help.ubuntu.ru/wiki/руководство_по_ubuntu_server/web_сервера/httpd_apache2_web_server) 

### Java 11 Runtime Environment (JRE 11)

Проверить установлена ли java и какая:
```
java -version
```
Если установлена другая сборка java 11 (не OpenJDK, для которой пример ниже), то ничего делать не нужно.

Чтобы установить java 11 для запуска программ на java (не для разработки), необходимо выполнить:
```
sudo apt-get install -y openjdk-11-jre
```
Если она уже установлена, то ничего плохого не произойдет.

### RAS

Запуск вручную RAS в режиме демона (если запущен, то ничего не произойдет):
```
/opt/1C/v8.3/x86_64/ras cluster --daemon
```
[Правильный запуск RAS](https://its.1c.ru/db/freshex3#content:288:hdoc:_top:ras)

Проверка:
```
/opt/1C/v8.3/x86_64/rac cluster list
```

## Установка на сервер

### Установка http-сервиса

Команды о порядку| Пояснение
---|---
`cd` | Перейти в домашнюю директорию пользователя
`git clone https://github.com/klimenko-1c/infobases.git` | Получить файлы http-сервиса из интернета
`cd infobases` | Перейти в папку с файлами http-сервиса
`sudo cp infobases.conf /etc/apache2/sites-available` | Скопировать конфигурацию сайта http-сервиса в папку с настройками apache
`sudo cp -R infobases /var/www` | Скопировать файлы сайта http-сервиса в папку с сайтами apache
`sudo a2enmod cgid` | Включить модуль выполнения скриптов на apache
`sudo a2ensite infobases` | Включить сайт http-сервиса в конфигурации apache
`sudo systemctl reload apache2` | Перечитать конфигурацию apache

### Установка 1C:Executor
Команды о порядку| Пояснение
---|---
`cd` | Перейти в домашнюю директорию пользователя
`sudo mkdir -p /opt/Executor` | Создать папку, в которую будет установлен Исполнитель
`sudo unzip ~/Executor_2020_2_0_547.zip -d /opt/Executor` | Разархивировать Исполнитель
`sudo mkdir -p /opt/Executor/logs` | Создать папку logs в папке Исполнителя
`sudo chgrp -R www-data /opt/Executor/logs` | Сменить владельца группы папки logs на пользователя, под которой apache запускает скрипты
`sudo chmod -R g+w /opt/Executor/logs` | Дать права на запись владельцу группы

## Проверка

Если все настроено верно, то команда
```
curl http://localhost/infobases/WebCommonInfoBases/GetInfoBases
```
должна выдать json с базами

Аналогичный URL (с правильным именем хоста) можно для проверки указать в браузере на компьютере, где будет настраиваться стартер 1С

## Настройка 1С на рабочем месте (не на сервере)

1. Запустить 1С. Появится окно со списком баз (Стартер 1С)
2. Нажать кнопку "Настройка...". Откроется окно "Настройка диалога запуска"
3. В списке "Адреса интернет-сервисов и списки информационных баз" нажать кнопку "Добавить" (зеленый плюсик). Откроется окно "Добавление ссылки"
4. В поле "Интернет-сервис" ввести `http://HOSTNAME/infobases` , заменив `HOSTNAME` на имя, по которому ваш сервер 1С доступен в сети. Допускается указать IP-адрес (если apache в дефолтной конфигурации)
5. Убедиться, что переключатель (точка) установлен на "Интернет-сервис"
6. Нажать кнопку "ОК" окна "Добавление ссылки"
7. Опционально (по желанию) установить галочку "Отображать в виде дерева". Список баз возвращается в папках по кластерам, поэтому отображение в виде дерева кажется более удобным.
7. Нажать кнопку "ОК" окна "Настройка диалога запуска"
8. Закрыть Стартер
9. Открыть Стартер. Наслаждаться.

## Известные проблемы и их решение
**Иногда список не появляется после переоткрытия**.

Откройте, подождите секунд 5, закройте и откройте Стартер снова. Это происходит из-за долгого запуска Исполнителя (порядка 2 с). На сколько я понял, в Стартере есть жестко зашитое время на ожидание ответов от веб-сервисов. Если за это время ответ не получен, то список баз строится без учета ответа (но используется предыдущий полученный ответ). Однако, если ответ приходит позже этого времени, то он записывается во внутренние файлы, и при следующем запуске эти данные используются.

---

Ставьте лайки 👍 и подписывайтесь 🔔

Больше лайков, больше подписок - больше скриптов 😊

---
> P.S. Описание формата файлов и прочая информация на [ИТС](https://its.1c.ru/db/v8317doc#bookmark:adm:TI000000422)
