# infobases
HTTP-сервис списка информационных баз

---

Если вы являетесь счастливым обладателем КОРП лицензии, то у вас есть возможность получать список баз по протоколу http.

Описание сервиса доступно на [ИТС](https://its.1c.ru/db/v8317doc#bookmark:adm:TI000000422)

## Установка

Порядок установки рассмотрен для Linux Ubuntu. Впрочем он легко адаптируется под MacOS и Windows.

Получить файлы веб-сервиса удобнее всего с помощью `git clone`

Для получение 1C:Executor можно воспользоваться любым привычным вам способом доставки файлов на сервер. 

### Установка web-сервиса

- `sudo cp infobases.conf /etc/apache2/sites-available`
- `sudo cp -R infobases /var/www`
- `sudo a2enmod cgid`
- `sudo a2ensite infobases`
- `sudo systemctl reload apache2`

### Подготовка 1C:Executor

#### Установка Java 11 Runtime Environment 

Проверить установлена ли `java` и какой версии можно командой `java -version` 

Установить `java`:
```
sudo apt-get install -y openjdk-11-jre
```

#### Установка Исполнителя в `/opt/Executor` из zip-архива

Предположим, что архив с Исполнителем находится в вашей домашней директории по пути `~/Executor_2020_2_0_547.zip`.

Тогда:
```
sudo mkdir -p /opt/Executor
sudo unzip ~/Executor_2020_2_0_547.zip -d /opt/Executor
```
И Исполнитель будет доступен по пути: `/opt/Executor/bin/executor_j11.sh`

Скрипт запуска Исполнителя явно определяет папку для записи логов. Эта папка определена в папке самого исполнителя. Поэтому после такой установки не получится его запускать в непривелигированном режиме.

> Для общего случая я хотел бы подождать комплексного решения от вендора. Для частного, можно, например, разместить Исполнитель в домашнем каталоге пользователя или явно переопределить папку логов. Но это выходит за рамки этого руководства. 

Сразу после установки, папки `logs` нет, создадим ее:
```
sudo mkdir -p /opt/Executor/logs
```

Дадим процессу Apache права записи в папку логов:
```
sudo chgrp -R www-data /opt/Executor/logs
sudo chmod g+w /opt/Executor/logs
```

>Если вы установили Исполнитель через установщик для linux, то сам Исполнитель находится примерно в этой папке
>```
> /opt/1C/1CE/components/1c-executor-ide-2020.2.0+431-x86_64/plugins/com.e1c.g5rt.dt.executor.core_0.1.0.v202006182040/embedded
> ```
> Все вышеперечисленные действия можно повторить и для этой папки. И поправить `GetInfoBases`. Я не пробовал.

## Проверка

Если все настроено верно, то команда
```
curl http://localhost/infobases/WebCommonInfoBases/GetInfoBases
```
должна выдать в консоль json с базами

Аналогичный URL (с правильным именем хоста) можно для проверки указать в браузере на компьютере, где будет настраиваться стартер 1С

## Настройка стартера 1С

Данное действие доступно только счастливым обладателям КОРП лицензии на сервер.

1. Запустить Стартер
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
