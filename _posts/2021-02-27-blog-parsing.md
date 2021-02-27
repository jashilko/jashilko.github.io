---
layout: post
title: Парсинг блогов
category: projects
---

# Проект о том, как скопировать блог. 


Оглавление: 
- [Постановка задачи](#task)
- [Схема работы программы](#schema)
- [Парсинг](#parsing)
- [Экспорт в Google Docs](#export)
- [Результаты](#result)
- [The middle](#the-middle)



## [Постановка задачи](#task)

**Проблема**: пользователь ведет блог на платформе, которая вот-вот закроется. Терять написанное годами он не хочет. Хочет сохранить для истории все, что писал, с картинками.

**Задача**: скопировать содержимое блога с блог-платформы в контролируемое пользователем место. 

**Используемые технологии**: Python, BeautifulSoup, Google Docs API


## [Схема работы программы](#schema)
КАРТИНКА

## [Парсинг](#parsing)

1. На вход программе подается адрес блога, с помощью request получаем первую страницу блога.
2. Программа парсит страницу блога. Она обычно состоит из нескольких постов. Пост состоит из элементов: заголовок, контент, дата, время, кол-во комментариев, url-поста. Значение каждого элемента записываются в словарь "Пост", этот словарь затем добавляется в качестве элемента словаря "Блог"
3. После парсинга всей страницы, ищется ссылка на следующую страницу, и следующая страница загружается.  
4. Предыдущие два пункта повторяеем, пока страницы не закончатся.
5. Словарь "Блог" записывается в JSON

## [Экспорт в Google Docs](#export) 

У меня было несколько вариантов, где сохранять содержимое блога: MS Word, Google Docs, Блокнот или оставить сырой JSON. Варинт с вордом хороший, но недоступный из онлайна, а в 2021 - это важно. Можно, конечно, выгрузить его в Google Docs, но зачем лишние движения, если можно сразу. В блокнот не вставить картинки, а JSON не поймут блондинки. 

6. Создаем по [гайду гугла](https://developers.google.com/docs/api/quickstart/python#step_3_set_up_the_sample) заготовку для работы с Google Docs (аналог ворда) - пустой документ.
7. Пробегаемся по JSON, и в каждом элементе "Пост", записываем в переменные значение элементов эже самого "Поста". 
8. Анализируем содержимое элемента "Контент". В нем может быть форматирование текста, картинки, ссылки.
9. С помощью Google Docs API вставляем в документ каждый элемент и одновременно задаем ему форматирование его абзазу. 
10. Повторяем предыдущие два пункта для всех элементов "Пост"
11. У Google API если лимиты на использование. После нескольких десяток вставок нужно запустить таймер и ждать заданный таймаут. 


## [Результаты](#result)
Программа работает не быстро из-за таймаутов гугла, но поставленная задача решена.

Сформированный JSON случайного блога:


Результат экспорта в Google Docs:

## [The middle](#the-middle)

Proin quis velit et eros auctor laoreet. Aenean eget nibh odio. Suspendisse mollis enim pretium, fermentum urna vitae, egestas purus. Donec convallis tincidunt purus, scelerisque fermentum eros sagittis vel. Aliquam ac aliquet risus, tempus iaculis est. Fusce molestie mauris non interdum hendrerit. Curabitur ullamcorper, eros vitae interdum volutpat, lacus magna lacinia turpis, at accumsan dui tortor vel lectus. Aenean risus massa, semper non lectus rutrum, facilisis imperdiet mi. Praesent sed quam quis purus auctor ornare et sed augue. Vestibulum non quam quis ligula luctus placerat sed sit amet erat. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia curae; Fusce auctor, sem eu volutpat dignissim, turpis nibh malesuada arcu, in consequat elit mauris quis sem. Nam tristique sit amet enim vel accumsan. Sed id nibh commodo, dictum sem id, semper quam.


