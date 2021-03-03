## Тестовое задание по SQL на собеседовании Поток. 


Оглавление: 
- [Описание задач](#task)
- [Окружение](#envr)
- [Задача 1](#task1)
- [Задача 2](#task2)
- [Задача 3](#task3)



## <a name="task">Описание задач</a>

Ниже приведена схема базы данных и описания запросов, которые необходимо составить.

|     dim_stores    	|                      	|
|-------------------	|----------------------	|
|     store_id      	|     int, PK          	|
|     store_nm      	|     nvarchar(255)    	|



|     dim_customers    	|                      	|
|----------------------	|----------------------	|
|     customer_id      	|     int, PK          	|
|     customer_nm      	|     nvarchar(255)    	|



|     fct_sales      	|                      	|
|--------------------	|----------------------	|
|     sale_id        	|     int, PK          	|
|     store_id       	|     int              	|
|     customer_id    	|     int              	|
|     sale_dt        	|     date             	|
|     sale_amt       	|     decimal(10,2)    	|


## <a name="envr">Окружение</a>

База данных использовалась SQLite, соответственно и ее ограничения были учтены (нет типа данных datetime). В задании, вижу nvarchar, значит скорее всего требовалось сделать все на MS SQL Server, но на домашнем ПК он не установлен, и явного требования к синтакису SQL не было. 

Прикрепляю файл дампа базы: https://disk.yandex.ru/d/V8yFXCmHoeyl7Q


### Задачи: 
1. Для каждого покупателя найдите магазин, в котором он совершил максимальное количество покупок. 
2. Найдите всех покупателей, которые в текущем году за каждый полный месяц совершали не менее 3 покупок. 
3. Напишите запрос, формирующий ниже представленный отчет: 

|     отчётный месяц    	|     продажи компании    	|     магазин    	|     Сумма продаж   магазина    	|     доля продаж магазина    	|
|-----------------------	|-------------------------	|----------------	|--------------------------------	|-----------------------------	|
|     2016.01           	|     100                 	|     Store1     	|     20                         	|     0,20                    	|
|     2016.01           	|     100                 	|     Store2     	|     80                         	|     0,80                    	|
|     2016.02           	|     500                 	|     Store1     	|     150                        	|     0,30                    	|
|     2016.02           	|     500                 	|     Store2     	|     350                        	|     0,70                    	|
|     …                 	|     …                   	|     …          	|     …                          	|     …                       	|





## <a name="task1">Задача 1</a>
```
select c.customer_nm, d.store_nm, max(count) as max_count, c.customer_id, d.store_id from 
(
	select customer_id, count(sale_amt) as count, store_id
	from fct_sales
	group by customer_id, store_id
) a
left join dim_stores d on d.store_id=a.store_id
left join dim_customers c on c.customer_id=a.customer_id
group by a.customer_id
```


## <a name="task2">Задача 2</a>
> Эта задача скорее всего решена не верно. Требуется пояснение фразы "за каждый полный месяц совершали не менее 3 покупок". 
Я реализовал: найдены клиенты, у которых в этом году в любом из месяцов было более трех продаж. 
Имеется ли в виду, что я должен вычислить, сколько полных месяцев прошло в этом году (в 2021, например, пока два), и посчитать, у кого за эти два месяца больше 3 продаж. 
Если дадите пояснения и сутки времени, попытаюсь исправить решение.

```
select customer_id, month, count(*)
FROM
(
	select *, strftime('%m', sale_dt) month from fct_sales
	where strftime('%Y', sale_dt) = strftime('%Y', 'now')
)
group by customer_id, month
having count(*)>3
```


## <a name="task3">Задача 3</a>

```	
select strftime('%m-%Y', sale_dt) month, t.t as 'total', s.store_id, ds.store_nm, sum(sale_amt) as 'sum_store', ROUND(sum(sale_amt)/t.t, 2) as '%' 
	from fct_sales s
	left join 
	(
		select strftime('%m-%Y', sale_dt) m, store_id as 's', sum(sale_amt) as 't' 
		from fct_sales
		group by m
	) as t on month=t.m
	left join dim_stores ds on ds.store_id = s.store_id
	group by month, s.store_id
	order by month, s.store_id
	
	
```

