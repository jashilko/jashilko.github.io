---
layout: post
title: Обработка разговоров из облачной АТС Билайн Бизнес
category: projects
---
# Сервис для сохранения записи разговоров из облачной АТС Билайн-Бизнес. 


Оглавление: 
- [Постановка задачи](#task)
- [Алгоритм работы сервиса](#algoritm)
- [Настройки](#settings)
- [Логгирование](#logging)
- [Трудности](#challenges)
- [Развертывание](#deploy)
- [Реклама](#adv)



## <a name="task">Постановка задачи</a>

**Проблема**: клиент пользуется облачной АТС Билайна и хочет. АТС умеет сохранять записи разговоров сотрудников и информацию о звонках. Нужно получать аудио-файл разговора, краткую информацию о нем и сохранять его на FTP клиента для последующей обработки.

**Задача**: Подключиться к облаку Билайна через API, получить последние разговоры, информацию о каждом разговоре сохранить отдельный в json-файл, скачать mp3 диалога. Оба файла загрузить на FTP клиента

**Используемые технологии**: Python, Beeline API, json, ftp, logging


## <a name="algoritm">Алгоритм работы программы</a>

1. Сервис подключается к API Билайна и получает список последних записей разговоров, начиная с того, на котором закончили в прошлый раз
2. Для каждого разговора по ИД API вызывается второй раз, и сервис получает детальную информацию. 
3. Детальная информация записывается в файл json в формате, который определил заказчик и сохраняется на локальный диск
4. По ИД разговора делается третий запрос к API, и сервис получает mp3-файл разговора, сохраняя его на диск. 
5. Сервис подключается к FTP-серверу клиента.
5. json и mp3-файлы заливаются на FTP
6. Оба файла удаляются с локального диска
7. Действия 2-6 повторяются в цикле, пока не закончится список записей, полученных в п.1
8. ИД последней обработанной записи сохраняется, чтобы в следующей итерации начать с него. 


## <a name="settings">Настройки</a>

В процессе обсуждения с заказчиком принято решение основые параметры работы сервиса сохранять в файле настроек. Это пригодиться для дальнейшего масштабирования сервиса на других клиентов. Кроме того, нужно было сохранять последний обработанный ИД. Избыточно создавать для небольшой группы настроек БД, пусть даже легковесную и принято решение сохранять все в ini-файле. 

Очевидные настройки: параметры FTP, токен к API и последнее обработанное ИД. Позднее добавились лимит на количество одновременно обработанных файлов, что связано с пропускной способностью FTP, и уровень логгирования. 

Для работы с настройками использовалась python-библиотека configparser


## <a name="logging">Логгирование</a>
Сервис использует большое относительно своей задачи число интеграций с другими системами: API, FTP, локальный диск. Для мониторинга корректной работы логгирую основные подключения к системам. Кроме того в логгирование добавил записи о том, какие файлы обработаны или какое количество файлов орбработано в зависимости от уровня логгирования. 

Логи пишутся в отдельную директорию в файлы по дням. Для работы использовал библиотеку logging


## <a name="challenges">Трудности</a>
При разработке возникли некоторые сложности: 

### Подключение к FTP клиента ###
Стандартная библиотека python ftplib не подключалась к FTP клиента. Хотя прекрасно читала и загружала файлы на локальном тестовом сервере, на некоторых тестовых серверах в сети. Не помогли подключение в пассивном режиме и запус сервиса с другого IP. Ошибка "TimeoutError: [WinError 10060] Попытка установить соединение была безуспешной, т.к. от другого компьютера за требуемое время не получен нужный отклик, или было разорвано уже установленное соединение из-за неверного отклика уже подключенного компьютера" не уходила. 
Заказжик предложил попробовать подключение через SFTP. После смены библиотеки на pysftp и замены реализации подключени к серверу заработало успешно. 

### Часть файлом терялось при записи на FTP ###
Сервис полуал от Билайна список из 100 файлов, обрабатывал их и загружал на сервер. Но на FTP оказывалось меньше файлов. Причины потерь установить не удалось, но при добавлении в таймера задержки проблема исчезала. Величину таймера решено было добавить в настройки. 

### При завершении цикла работы файл программы "исчезал" с диска ###
После отработки цикла файл сервиса исчезал с диска. Это было связано с тем, что по своей задаче каждый из файлов сервис сохраняет на локальном диске, а через какое-то время удаляет его. Разработка велась на Windows, и какая-то часть ОС считала программу вредоносной. Отключение антивируса и других защитников не помогло. Найти быстрое решение, чтобы сервис на лету получал файл из АТС и записывал его на FTP не удалось, диск как транспортная точка был оставлен. Промышленное развертывание сервиса планировалось на unix-системе, там этой проблемы не возникало. 


## <a name="deploy">Развертывание</a>
Заказчик выделил unix-машину для развертывания сервиса. Изначально предполагалось разворачивать сервис через Docker, но из-за ограниченных ресурсов по времени и неограниченных по железу принято разворачивать 1 сервис на 1 машине. 
На выделенном сервере установлен git, python, pip. Настроен cron. Каждые 15 минут планировщик выполняет питоновский скрипт. 
При необходимости обновлений скрипта, они выполняются подтягиваем изменений с git-репозитория. 

## <a name="adv">Реклама</a>
Исходный код и инструкция пользователя размещены по ссылке https://github.com/jashilko/BeelineAPItoFTP
Если вам нужны легковестные сервисы дял выполнения бизнес-задач, пишите run@kostya.run