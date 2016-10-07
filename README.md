[![Build Status](https://travis-ci.org/centosadmin/redmine_2fa.svg?branch=master)](https://travis-ci.org/centosadmin/redmine_2fa)
[![Code Climate](https://codeclimate.com/github/centosadmin/redmine_2fa/badges/gpa.svg)](https://codeclimate.com/github/centosadmin/redmine_2fa)
# Redmine 2FA

Two-factor authorization plugin for Redmine.

Supports:
* Telegram
* SMS
* Google Auth

Developed by [Centos-admin.ru](https://centos-admin.ru/)

## Requirements

This plugin works only on HTTPS host, because of Telegram Bot Webhook needs to POST on HTTPS hosts.

Ruby 2.3+

### Важно!!!

Бот для этого плагина должен быть уникальным. Иначе могут быть конфликты, если тот же бот используется в другом плагине в режиме опроса обновлений.

Бот может работать либо через web-hook либо через периодический опрос.

В этом плагине используется механизм web-hook, поэтому использование протокола HTTPS обязательно.

Если в разных плагинах один и тот же бот использует разные механизмы, приоретет отдаётся web-hook.

Инструкция по созданию бота: https://core.telegram.org/bots#3-how-do-i-create-a-bot

## Установка плагина

После добавления плагина в пупку `plugins` выполните следующие команды
```
bundle
bin/rake redmine:plugins:migrate
```

# Авторизация через Telegram

## Первый запуск

При первом запуске нужно: 
* ввести токен бота в настройках плагина
* сохранить настройки
* инициализировать бота по ссылке в настройках

Во время иницализации будут сохранены id и username бота и установлен web-hook, который будет обрабатывать команды направляемые боту.

## Настройки для пользователя

При первом входе пользователю будет предложено выбрать способ для второго шага аутентификцаии.
 
При выборе Telegram пользвателю нужно будет добавить себе бота, чтобы получать от него одноразовые пароли.

При добавлении бота, он предложит ввести команду `/connect e@mail.com` для получения ссылки на связывание аккаунтов Telegram и Redmine.

После выполнения команды пользователь получит письмо со ссылкой.
Переход по ссылке свяжет аккаунты пользователя и он сможет получать одноразовые пароли от бота

# Авторизация через SMS

## Общая информация

При первом входе пользователю будет предложено выбрать способ для второго шага аутентификцаии.
 
При выборе SMS пользвателю нужно ввести номер телефона, на который он будет получать СМС и подтвердить его.

## Конфигурация

В связи с тем, что различные СМС-шлюзы используют различные API, в плагине испольуется отправка СМС через системную 
команду. Например

```
curl http://my-sms-gateway.net?phone=%{phone}&message=%{password}%20ExpiredAt:%{expired_at}
```
`%{phone}`, `%{password}` и `%{expired_at}` системные переменные, которые будут замещены соответствующими значениями. 

* phone - номер телефона в формате 7894561230 (только цифры, начинается с кода страны)
* password - одноразовай пароль, который необходимо ввести на втором шаге аутентификации
* expired_at - время после которого пароль перестаёт быть действительным (2 минуты от момента отправки сообщения)

Команда по-умолчанию: `echo %{phone} %{password}`.

Команда для отправки СМС указывается в файле `config/configuration.yml` в секции `production`:
```yaml
# specific configuration options for production environment
# that overrides the default ones
production:
  redmine_2fa:
    sms_command: 'echo %{phone} %{password}'
```

## Миграция с плагина redmine_sms_auth

* обновите плагин redmine_sms_auth до последней версии
* обновите плагин redmine_2fa до последней версии
* выполните команду `bundle install`
* выполните команду `bundle exec rake redmine:plugins:migrate`
* выполните команду `bundle exec rake redmine:plugins:migrate VERSION=0 NAME=redmine_sms_auth`
* удалите каталог с плагином redmine_sms_auth
* обновите настройки в файле `configuration.yml`
  * было
    ```yaml
    production:
      sms_auth:
        command: 'echo %{phone} %{password}'
        password_length: 5
    ```
  * стало
    ```yaml
    production:
      redmine_2fa:
        sms_command: 'echo %{phone} %{password}'
    ```

* перезапустите Redmine

Параметр password_length больше не используется, так как в Google Auth используется фиксированная длинна кода - 6 цифр.

В плагине redmine_sms_auth к польвателям было добавлено поле "Мобильный телефон", значение которого используется для 
отправки СМС.

При миграции в соотвествии с этой инструкцией поле будет сохранено и данные с номерами телефонов будут доступны в 
плагине redmine_2fa.

# Google Authenticator

При первом входе пользователю будет предложено выбрать способ для второго шага аутентификцаии.
 
При выборе Google Auth пользвателю будет показан QR-код, который нужно сосканировать в приложении [Google 
Authenticator](https://support.google.com/accounts/answer/1066447).

# Сброс способа аутентификации

Пользователь может сбросить способ двухфакторной аутентификации на странице "Моя учётная запись".

# Игнорирование второго шага аутентификации

В настройка пользователя администратор может указать "Игнорировать 2FA".

Если в настройка плагина снята галка "Обязательно требовать выбрать один из способов аутентификации 2FA", то 
пользователь может выбрать "Не использовать" при первом входе в систему.

# Автор плагина

Плагин разработан [Centos-admin.ru](http://centos-admin.ru/).

