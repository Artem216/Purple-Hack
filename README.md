# MISIS Dark Horse

Предсказание продуктового кластера клиента

## Описание задачи

Нам известно, что основной характеристикой клиента, влияющей на его прибыльность, является набор продуктов , которыми клиент активно пользуется, то есть генерирует операционную прибыль. Набор (множество) продуктов клиента называется продуктовым кластером. При изменении кластера клиента, например при открытии нового продукта или закрытии существующего, у него существенным образом меняется профиль доходности. Нужно построить модель, которая будет предсказывать продуктовый кластер клиента - Юридического лица на горизонте в 12 месяцев.

## Работа с данными

### Заполнение пропусков

Заполнение пропусков является важным этапом для качества нашей модели. Поэтому мы уделили им особое внимание.

- Столбцы, в название которых начиналось с **'balance*amt*'**, **'cnt\_'**, **'sum\_'**, заполнили модой, то есть самым популярным значением в столбце.
- Столбцы, в название которых начиналось с **'orgn'**, и столбцы **'channel_code'**, **'segment'** заполнили сначала протягиванием значения с предыдущего месяца к следующему (ffill), а затем протягиванием с будущих месяцев на предыдущие (bfill). Перед этим произвели группировку по id.
- Столбцы, в название которых начиналось с **'city'**, и столбец **'okved'** заполнили протягиванием значения с будущих месяцев на предыдущие (bfill). Перед этим произвели группировку по id.
- Столбцы **'index_city_code'**, **'max_founderpres'**, **'min_founderpres'**, **'ogrn_exist_months'** мы удалили так, как они сильно коррелировали со столбцом **'ft_registration_date'**.
- Столбец **'ft_registration_date'** решили заполнить линейной регрессией, так как заметили линейный помесячный рост в данном столбце. Для этого мы вычислили среднюю помесячную разницу и с помощью нее востанавливали пропуски прибавляя и вычитая разницу с предыдущих и будущх месяцев. Перед этим произвели группировку по id.
- Столбец **'start_cluster'** тестовой выборке заполнили протягиванием значения с предыдущего месяца к следующему (ffill). Перед этим произвели группировку по id.
- Пропуски в категориальных признаках заменили на **None**, чтобы модель смогла их обработать
- Были созданы бинарные столбцы - которые показывали, открыт ли тот или иной продукт.
  После данной обработки мы заполнили большую часть пропусков, но не все. Оставшиеся будут уже обработаны самой моделью.

### Новые фичи

**null_group_feature** - создает бинарные признаки, которые показывают объект содержит то или иное подмножество признаков, с одинаковым количеством пропусков или нет. Подмножество признаков - это признаки исходного датасета, которые содержат одинаковое кол-во пропусков, причем для всех объектов должно выполняться условие, что у всего подмножества либо все значения пропущены, либо нет. Так же создается столбец, который показывает сколько всего пропусков в строке.

**operations_usage_feature** - создаёт признаки, которые показывают наличия операций (входящих/исходящих) по продуктам

**change_class_feature** - создаёт признак, который отслеживает месячные изменения кластера клиента.

## Архитектура модели

### Catboost

Мы тщательно изучили различные алгоритмы машинного обучения и с уверенностью выбрали CatBoost в качестве модели для предсказания продуктового кластера клиентов. Для обучения брались только строки, у которых в исходных данных было меньше 50 пропусков. И так же для обучения исключили столбец **start_cluster**, так как он слишком сильно влиял на **end_cluster**. Мы настроили модель CatBoost с использованием оптимальных гиперпараметров, чтобы добиться максимальной точности предсказаний. Благодаря эффективной обработке категориальных признаков, CatBoost позволяет нам использовать важную информацию о продуктах и клиентах, чтобы сделать более точные прогнозы о продуктовом кластере.

### Что было опробовано

Каскад из моделей бинарной классификации. Отдельная модель предсказывает склонность и отток по каждому продукту. У данного способа есть как плюсы, так и минусы:

- Независимое добавление и изменение модели продукта
- Широкие возможности для улучшение качества модели продукта

* Сложность реализации и доп. алгоритм
* Более долгое обучение

В перспективе данное решение лучше, но из-за нехватки возможностей и времени мы отказались от этого подхода
