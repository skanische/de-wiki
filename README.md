# DE Wiki

Личная база знаний по Data Engineering.

Цель проекта - систематизировать знания по основным DE-технологиям и подходам, углубиться в литературу и разобрать принципы применения.

## Содержание

### Orchestration

#### Airflow

1. [Airflow Overview and DAG Basics](Orchestration/Airflow/01_airflow_overview_and_dag_basics.md)  
   Введение в Airflow: DAG, task, operator, scheduler, worker, UI, обработка ошибок и rerun.


## Формат статей

В Wiki используются два формата:

- стандартный шаблон — для крупных системных тем;
- укороченный шаблон — для небольших и прикладных тем.

Правила выбора и структура статей описаны в документе:

- [Шаблоны статей DE Wiki](_templates/template_policy.md)

Примеры:

- [Укороченная статья: HTTP Methods](_templates/examples/article_short_http_methods.md)
- [Стандартная статья: Spark Execution Model](_templates/examples/article_standard_spark_execution_model.md)


## Статус проекта

Wiki находится в активной разработке.

На текущем этапе добавляются статьи по Apache Airflow.


## Структура репозитория

```
DEWiki/
  _templates/
    examples/
      article_short_http_methods.md
      article_standard_spark_execution_model.md
    template_policy.md
  Orchestration/
    Airflow/
      01_airflow_overview_and_dag_basics.md
```
