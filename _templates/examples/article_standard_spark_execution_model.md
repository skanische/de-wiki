# Spark Execution Model

## 1. Кратко

- Spark не выполняет большинство операций сразу.
- Transformations описывают план вычислений, но не запускают выполнение.
- Actions запускают реальное выполнение Spark job.
- Spark application управляется driver-процессом.
- Executors выполняют tasks над partitions.
- Job разбивается на stages.
- Stage разбивается на tasks.
- Shuffle обычно разделяет job на несколько stages.
- Понимание execution model помогает объяснять производительность, количество tasks, shuffle, persist/cache и поведение Spark UI.


## 2. Зачем это нужно и где используется

Spark Execution Model нужна Data Engineer, чтобы понимать, как Spark превращает пользовательский код в распределенное выполнение.

Эта тема используется почти во всех задачах PySpark и Spark SQL:

- чтение больших таблиц;
- фильтрация и агрегация данных;
- join больших датасетов;
- построение витрин;
- инкрементальные загрузки;
- оптимизация долгих Spark jobs;
- диагностика skew, shuffle spill и нехватки памяти;
- анализ Spark UI;
- настройка количества partitions.

Без понимания execution model легко ошибочно считать, что Spark выполняет код построчно. На практике Spark строит план вычислений и запускает его только тогда, когда возникает action.


## 3. Базовая mental model

Главная идея:

Spark — это распределенный движок вычислений, который строит DAG операций и выполняет его лениво.

Базовая модель:

1. Пользователь пишет код.
2. Spark накапливает transformations в план.
3. Action запускает выполнение.
4. Driver анализирует план.
5. Spark делит вычисления на jobs.
6. Jobs делятся на stages.
7. Stages делятся на tasks.
8. Tasks выполняются на executors.
9. Executors обрабатывают partitions.

Основные сущности:

| Сущность | Что означает |
|---|---|
| Application | Запущенное Spark-приложение |
| Driver | Процесс, который управляет Spark-приложением |
| Executor | Процесс, который выполняет tasks |
| Job | Вычисление, запущенное action |
| Stage | Группа tasks без shuffle boundary |
| Task | Минимальная единица выполнения, обычно обработка одной partition |
| Partition | Часть распределенного датасета |
| Transformation | Ленивое описание преобразования данных |
| Action | Операция, которая запускает выполнение |

Инженерная интуиция:

- если операция не требует перемещения данных между executors, скорее всего это narrow transformation;
- если операция требует перераспределения данных по ключам, скорее всего будет shuffle;
- если есть shuffle, почти всегда появится новая stage;
- если есть action, появится job;
- количество tasks в stage обычно связано с количеством partitions;
- один DataFrame не равен одному job;
- один action не всегда означает только одну простую операцию внутри Spark.


## 4. Как это работает внутри

Пример цепочки:

```python
df = spark.read.parquet("/data/events")

result = (
    df
    .filter(F.col("event_date") >= "2026-01-01")
    .groupBy("user_id")
    .agg(F.count("*").alias("events_cnt"))
)

result.count()
```

Что происходит:

1. `spark.read.parquet(...)` описывает источник данных.
2. `filter(...)` добавляет transformation в план.
3. `groupBy(...).agg(...)` добавляет агрегацию.
4. До вызова `count()` Spark может ничего не читать и не считать.
5. `count()` является action.
6. Driver запускает job.
7. Spark анализирует зависимости между операциями.
8. `filter` обычно является narrow transformation.
9. `groupBy` обычно требует shuffle.
10. Shuffle создает boundary между stages.
11. Каждая stage разбивается на tasks.
12. Executors выполняют tasks.
13. Driver получает результат action.

Narrow transformation:

Narrow transformation — операция, при которой каждая выходная partition зависит от ограниченного числа входных partitions.

Примеры:

- `filter`;
- `select`;
- `withColumn`;
- `map`;
- `union`, если не требуется перераспределение данных.

Wide transformation:

Wide transformation — операция, при которой данные нужно перераспределить между partitions.

Примеры:

- `groupBy`;
- `join` без broadcast;
- `distinct`;
- `repartition`;
- `orderBy`.

Shuffle:

Shuffle возникает, когда Spark должен перераспределить данные между executors.

Shuffle дорогой, потому что может включать:

- запись промежуточных данных;
- передачу данных по сети;
- чтение промежуточных данных;
- сортировку;
- spill на диск;
- нагрузку на память и сеть.

Важные правила:

- transformation обычно не запускает job;
- action запускает job;
- shuffle обычно разделяет stages;
- task выполняется над partition;
- больше partitions обычно означает больше tasks;
- слишком мало partitions может плохо использовать кластер;
- слишком много partitions может создать overhead на планирование и запуск tasks.


## 5. Практическое использование

### Пример action и transformation

```python
from pyspark.sql import functions as F

filtered_df = df.filter(F.col("event_date") >= "2026-01-01")

# Здесь Spark еще может ничего не выполнить.
# filter — transformation.

rows_count = filtered_df.count()

# Здесь Spark запускает job.
# count — action.
```

### Пример shuffle

```python
from pyspark.sql import functions as F

agg_df = (
    df
    .groupBy("user_id")
    .agg(F.count("*").alias("events_cnt"))
)

agg_df.count()
```

В этом примере `groupBy` обычно приводит к shuffle, потому что строки с одинаковым `user_id` должны оказаться вместе для агрегации.

### Пример просмотра плана

```python
result.explain(True)
```

`explain(True)` помогает увидеть logical plan и physical plan. Это один из первых инструментов для анализа того, что Spark собирается выполнять.

### Практические рекомендации

- После сложных преобразований проверять план через `explain`.
- Следить за количеством shuffle-операций.
- Не использовать `cache()` без понимания, сколько памяти это займет.
- После `cache()` вызвать action, если нужно материализовать кэш.
- Проверять Spark UI: jobs, stages, tasks, shuffle read/write, spill, duration.
- При долгих join проверять, не появился ли большой shuffle.
- При перекосе данных искать stages, где отдельные tasks выполняются сильно дольше остальных.


## 6. Подводные камни и диагностика

### Частые ошибки

1. Считать, что transformation выполняется сразу.

Например:

```python
df.filter(F.col("status") == "active")
```

Этот код сам по себе не обязан запускать вычисление. Он только описывает transformation.

2. Путать job, stage и task.

Правильная логика:

- job появляется из action;
- stage появляется как часть job;
- task появляется как работа внутри stage над partition.

3. Думать, что `cache()` сразу кэширует данные.

`cache()` только помечает DataFrame для кэширования. Фактическое заполнение кэша происходит при action.

4. Игнорировать shuffle.

Shuffle часто является причиной долгих jobs, spill на диск и высокой сетевой нагрузки.

5. Считать, что больше partitions всегда лучше.

Слишком большое количество partitions создает overhead на планирование и выполнение большого числа маленьких tasks.

### Как это ломается

Типовые проблемы:

- слишком большой shuffle;
- data skew;
- spill на диск;
- нехватка executor memory;
- слишком мало partitions;
- слишком много partitions;
- повторное вычисление одного и того же lineage без cache/persist;
- неудачный join strategy;
- чтение большого объема данных без partition pruning.

### Как диагностировать

Что смотреть:

- Spark UI;
- вкладки Jobs, Stages, SQL;
- duration stages;
- количество tasks;
- shuffle read/write;
- spill memory/disk;
- skew между tasks;
- physical plan через `explain(True)`.

Минимальные команды и действия:

```python
result.explain(True)
```

```python
df.rdd.getNumPartitions()
```

```python
spark.conf.get("spark.sql.shuffle.partitions")
```


## 7. Сравнения и связанные темы

### Job vs Stage vs Task

| Сущность | Уровень | Что означает |
|---|---|---|
| Job | Верхний | Запускается action |
| Stage | Средний | Часть job между shuffle boundaries |
| Task | Нижний | Выполнение stage над одной partition |

### Transformation vs Action

| Тип операции | Что делает | Запускает выполнение |
|---|---|---|
| Transformation | Описывает преобразование данных | Обычно нет |
| Action | Требует результат | Да |

Примеры transformations:

- `filter`;
- `select`;
- `withColumn`;
- `join`;
- `groupBy`;
- `distinct`.

Примеры actions:

- `count`;
- `collect`;
- `show`;
- `take`;
- `write`;
- `foreach`.

### Narrow vs Wide transformations

| Тип | Что происходит | Примеры |
|---|---|---|
| Narrow | Нет полного перераспределения данных | `filter`, `select`, `map` |
| Wide | Требуется shuffle | `groupBy`, `join`, `distinct`, `orderBy` |

### Связанные темы

- Spark Shuffle;
- Spark Partitioning;
- Spark Joins;
- Spark Cache and Persist;
- Spark SQL Physical Plan;
- Catalyst Optimizer;
- Adaptive Query Execution;
- Data Skew;
- Parquet Partition Pruning.


## 8. Вопросы для самопроверки

1. Почему Spark transformation обычно не запускает вычисление?
2. Что должно произойти, чтобы Spark создал job?
3. Чем job отличается от stage?
4. Чем stage отличается от task?
5. Почему shuffle обычно разделяет job на несколько stages?
6. Почему `groupBy` часто приводит к shuffle?
7. Почему `filter` обычно считается narrow transformation?
8. Что произойдет, если вызвать `cache()`, но не вызвать action?
9. Как количество partitions связано с количеством tasks?
10. Где в Spark UI можно смотреть долгие stages?
11. Какие признаки указывают на data skew?
12. Почему `collect()` может быть опасен на больших данных?
13. Чем client mode отличается от cluster mode с точки зрения driver?
14. Какие метрики стоит смотреть при диагностике shuffle?
15. Почему один DataFrame может участвовать в нескольких jobs?


## 9. Материалы

### Основные

- Apache Spark Documentation — Cluster Mode Overview.
- Apache Spark Documentation — RDD Programming Guide.
- Apache Spark Documentation — SQL, DataFrames and Datasets Guide.
- Apache Spark Documentation — Tuning Guide.
- Learning Spark, 2nd Edition — Jules Damji, Brooke Wenig, Tathagata Das, Denny Lee.

### Дополнительные

- Spark: The Definitive Guide — Bill Chambers, Matei Zaharia.
- High Performance Spark — Holden Karau, Rachel Warren.
- Материалы по Spark UI и диагностике shuffle.
- Материалы по Catalyst Optimizer и Adaptive Query Execution.
