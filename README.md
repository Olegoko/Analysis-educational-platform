# Analysis-educational-platform
Анализ образовательной платформы

## Стек: ##
SQL, Clickhouse, Python и библиотеки(pandas, seaborn, numpy, scipy.stats, requests, json, pingouin, pandahouse).

## Цель: ## 
1. Проанализировать результаты AB-тестирования новой механики оплаты услуг на сайте.
2. Выявление когорты активных учеников + построение метрик, характеризующих показатели дохода:
   - ARPU
   - ARPAU
   - CR в покупку
   - СR активного пользователя в покупку
   - CR пользователя из активности по математике (subject = ’math’) в покупку курса по математике
3. Реализация функции которая будет автоматически подгружать информацию из дополнительного файла и на основании дополнительных параметров пересчитывать метрики и результаты AB-тестирования.

## Ход работы: ## 
1. У нас 4 датасета.  
- group - файл с информацией о принадлежности пользователя к контрольной или экспериментальной группе (А – контроль, B – целевая группа).  
- groups_add - дополнительный файл с пользователями, который прислали спустя 2 дня после передачи данных.  
- active_studs - файл с информацией о пользователях, которые зашли на платформу в дни проведения эксперимента.  
- checks - файл с информацией об оплатах пользователей в дни проведения эксперимента.  
*План действий:*
- Производим предобработку данных.
- Объединяем 2 датасета с информацией о принадлежности методом pd.concat()(добавление строк в датафрейм).  
- Объединяем получившийся датасет с датасетом active_studs методом merge()(добавление стролбцов в датафрейм по ключевому полю). Используя аргумент how = 'left', отсечем id, которые не появлялись в день эксперимента.  
- Объединяем получившийся датасет с датасетом checks методом merge(). Получим финальный датафрейм с информацией об оплате.
- Проверяем на ошибки полученный датафрейм.
- Рассчитываем метрики: Conversion Rate и Average check. Методом хи-квадрат Пирсона и т-критерий Стьюдента проводим их статистический анализ. Делаем выводы.
2. Используя библиотеку Pandahouse из Jupyternotebook подключаемся к Clickhouse и составляем сложный запрос на SQL, используя операторы JOIN и WITH (временный результирующий набор данных, который можно использовать в рамках одного запроса).
3. Реализована функция, принимающая недостающий датафрейм, на основании которых, заново рассчитывает необходимые метрики и проводит их статистический анализ.
  

## Вывод: ## 
1. Основываясь на полученных данных, можно сделать вывод, что целевой способ оплаты *увеличивает средний чек*. А при схожей конверсии в покупку он увеличит и общую выручку.
Этот вывод можно сделать только теоретически.   
К сожалению, практически мне мешает отсутствие некоторых второстепенных условий:  
Мне не понятен принцип распределения пользователей на группы. Изначально они распределены так, что выборки получаться разными по размеру.
Для значимости эксперимента *лучше делить пользователей в момент, когда они посещают сайт, а не до него.* Так выборки оказались бы примерно равными по размеру.
К сожалению, в этом тесте я не имею информацию о том, насколько сильно и как поменялась механика оплаты. Ведь изменения в первую очередь должны повлиять на конверсию, а не на средний чек (конечно, если изменения повлияли на более "дорогие" или более "объёмные" покупки, то тут понятны расхождения в данных). Такое расхождение средних чеков могло бы быть, если пользователи распределены по группам не случайно, а по какому-нибудь признаку (например, город-область).
**В итоге:** Новую механику оплаты однозначно **выпускать** на всех пользователей.
2. С помощью оптимизированного SQL-запроса выявлена когорта *усердных* учеников. Выгружены метрики, характеризующие показатели дохода, для оптимизации воронки продаж.
3. Реализована функция для автоматизации расчета необходимых метрик и построения графиков.
