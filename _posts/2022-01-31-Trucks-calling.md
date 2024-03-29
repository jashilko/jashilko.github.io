---
layout: post
title: Обзвон грузовиков
category: projects
---
# Обзвон грузовиков

Оглавление: 
- [Постановка задачи](#task)
- [Алгоритм работы сервиса](#algoritm)
- [Настройки скрипта](#settings)
- [Логгирование работы](#logging)
- [Скриншоты](#scrins)
- [Трудности, с которыми столкнулся](#challenges)
- [Ссылки, которые помогли](#links)
- [Реклама](#adv)

## <a name="task">Постановка задачи</a>
1. Возможность ввода информации через веб-интерфейс
    * Создать веб-интерфейс с двумя таблицами: список номеров телефонов и расписание звонков по дням. Вход должен осужествляться по логину и паролю. 
    * Создание списка номеров телефонов и поля контекст (todo.json) , начинающихся с 89..; редактирование и удаление элементов в списке
    * Изменение и сохранение расписания запуска (schedule.json) в формате
    ```
        Пн: 09-00
        Вт: 09-00
        Ср: 09-00
        Чт: 09-00
        Пт: 10:36
        Сб: 13:17
        Вс: 00:00 - не запускается
    ```

2. Автоматическое формирование файлов для Астериска, позволяющее обзванивать телефоны. 
* Формат обычных файлов:
```
Channel: local/89161111111@avtoobzvon
CallerID:
MaxRetries: 0
RetryTime: 60
WaitTime: 300
Context: avtoobzvon
Extension: play
Priority: 1
```

* Формат файлов с добавочным номером:
```
Channel: local/89502222222@avtoobzvon-ivr
CallerID: 6122103
MaxRetries: 0
RetryTime: 60
WaitTime: 300
Setvar: dtmf=1
Setvar: CALLEE=89502222222
Context: avtoobzvon-ivr
Extension: play
Priority: 1
```
* Файлы должны копироваться в директорию астериска пачками с задержками между пачек

## <a name="algoritm">Алгоритм работы программы</a>
1. Пользователь в веб интерфейсе добавляет номера телефонов в таблицу. В дополнении к номеру телефона можно указать добавочный код ("Нажмите 1, если хотите...")
2. Так же пользователь в веб интерфейсе задает расписание начала обзвона всех номеров телефона по дням. 
3. После окончания ввода пользователем формируется json-Файл со списком номеров телефонов и json-файл со временем обзвона на каждый день. 
4. Скрипт на python, который вызывается по расписанию проверяет, наступило ли время обзвона. 
5. Если время обзвона наступило, то скрипт формирует call-файлы нужного формата, складывает их в директорию Астериска с заданной задержкой, чтобы не перегрузить АТС


## <a name="settings">Настройки скрипта</a>
```
[Dirs]
; Директория с веб-страницей, куда сохраняются todo.json и schedule.json . Должна быть создана
dir_source = Z:\home\localhost\www\
; Временная директория для хранения call-файлов. Должна быть создана
dir_store = store
; Директория, откуда Астериск берет call-файлы
dir_target = target
; Директория сохранения логов работы программы. Должна быть создана
dir_logs = log
; Имя файла со списком телефонов
file_phones = todo.json
; Имя файла с расписание запуска. 
file_sch = schedule.json
[CreateCallFiles]
; Off - Программа не запущена. Ещё не отработала сегодня или уже отработала. 
status = Off
; Дата последней отработки программы. Если = сегодня, то сегодня запуск уже был и сегодня его уже не будет. 
start = 2022-01-24
; Размер пачки
bundle = 2
; Задержка в секундах между пачками
timeout = 10
```

## <a name="logging">Логгирование работы</a>
В коде есть задание уровня логгирования: logmode = 'INFO'

Варианты, которые можно установить: 

   * **'INFO'** - Логгироваться будут только события с созданием и перемещением call-файлов в заданное время. Это основной режим логгирования
   * **'DEBUG'** - Логгироваться будут все проверки расписания, т.е лог будет писаться каждый раз при вызове скрипта кроном. 
    Это режим отладки

Логи пишутся в отдельную директорию в файлы по дням. Для работы использовал библиотеку logging

## <a name="scrins">Скриншоты</a>
Веб интерфейс: 
![image](https://user-images.githubusercontent.com/5080414/152872167-c9bc1162-d4dd-4b37-b339-188ad3ad6f56.png)

Форма логина: 
![image](https://user-images.githubusercontent.com/5080414/152871619-9177b231-89af-4494-a3cd-ca581073d7de.png)


## <a name="challenges">Трудности, с которыми столкнулся</a>
При разработке возникли некоторые сложности: 

### Старая ОС ###
Заказчик предоставил CentOS 6. Это уже неподдерживаемая ОС. Трудности возникли с установкой интерпретатора Python 3.6. Скачать из репозитория не получалось. Исправление зеркал не помогло. Помогла только сборка Python из исходников

### Трудности с настройкой CRON ###
Задание в CRON не хотело выполняться. Хотя та же самая инструкия в командной строке работала. Решение: нужно было в CRON прописать полный путь к интерпретатору Python

### Кодировка web-страницы ###
Разработка интерфейсной части (web-страница на PHP) велась на Windows. После деплоя на CentOS русские буквы не отображались. Помогло перекодирование из win-1251 → utf-8


## <a name="links">Ссылки, которые помогли</a>
* [Модуль Logging](https://webdevblog.ru/logging-v-python/)
* [Хоткеи mc](https://scabere.livejournal.com/83643.html)
* [Установка Python 3.6 из исходников](https://adminwin.ru/ustanovka-python-3-6-na-centos/)
* [Попытка замены зеркал при ошибке: Cannot find a valid baseurl в CentOS 6](https://xost.su/support/cannot-find-a-valid-baseurl-centos-6)
* [Хоткеи mcedit](https://any-key.net/mcedit-hotkeys/)
* [CRON](https://www.digitalocean.com/community/tutorials/how-to-use-cron-to-automate-tasks-centos-8-ru)
* [Ещё CRON](https://blog.sedicomm.com/2017/07/24/kak-dobavit-zadanie-v-planirovshhik-cron-v-linux-unix/)
* [vi](https://docs.altlinux.org/ru-RU/archive/2.3/html-single/junior/alt-docs-extras-linuxnovice/ch02s10.html)

## <a name="adv">Реклама</a>
Исходный код и инструкция пользователя размещены по ссылке https://github.com/jashilko/BeelineAPItoFTP
Если вам нужны легковестные сервисы дял выполнения бизнес-задач, пишите run@kostya.run
