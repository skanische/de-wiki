# Roadmap DEWiki

## Airflow

- [x] [Роль Airflow для data-пайплайнов](10%20Orchestration/10.1%20Airflow/10.1.1_airflow_role_in_data_pipelines.md)
- [ ] Task, Operator и Task Instance
- [ ] Scheduler, Executor и Worker
- [ ] Расписания и интервалы данных
- [ ] Повторные попытки и запуски. Связать с темой идемпотентности

## Spark

- [x] [Что такое Spark](9%20Spark/9.1_what_is_spark.md)
- [ ] Модель выполнения Spark
- [ ] DataFrame и Spark SQL
- [ ] Партиции и shuffle

## Общие темы

- [ ] Идемпотентность в data-пайплайнах
- [ ] SQL и моделирование DWH по мере чтения

## Структура wiki

- [ ] По мере наполнения:
  - [ ] 11 Data Quality and Observability.
    Папка общая, но `monitoring_alerting_and_lineage.md` объединяет материал для разных вопросов:
    - Мониторинг и алерты можно раскрыть на примере опоздавшей витрины.
    - Lineage заслуживает отдельной заметки, когда появится кейс про происхождение данных
    или влияние изменения источника
  - [ ] хранилища в 7, 12 и 13.
  Границу удобно провести по вопросу читателя:
  - *Почему Parquet и как разложить файлы?* -- 7 Storage and File Formats
  - *Как выбрать между lake, warehouse и lakehouse?* -- 12 Data Architecture.
  - *Как настроить доступ к объектному хранилищу?* -- 13 Infrastructure.

## Будущая структура wiki

```text
DEWiki/
├── README.md
├── ROADMAP.md
│
├── 1 Common/
│   ├── 1.1_git.md
│   ├── 1.2_linux_and_shell.md
│   └── 1.3_docker.md
├── 2 Computer Science/
│   ├── 2.1_data_structures.md
│   └── 2.2_networking_and_http.md
├── 3 Python/
│   ├── 3.1_python_for_data_engineering.md
│   └── 3.2_testing_with_pytest.md
├── 4 SQL and Databases/
│   ├── 4.1_sql_queries.md
│   ├── 4.2_indexes_and_query_plans.md
│   └── 4.3_transactions_and_isolation.md
├── 5 Data Modeling and Warehousing/
│   ├── 5.1_grain_facts_and_dimensions.md
│   ├── 5.2_star_schema_and_data_marts.md
│   └── 5.3_slowly_changing_dimensions.md
├── 6 Data Pipelines/
│   ├── 6.1_etl_and_elt.md
│   ├── 6.2_incremental_loading.md
│   ├── 6.3_idempotency.md
│   ├── 6.4_backfill_and_replay.md
│   └── 6.5_change_data_capture.md
├── 7 Storage and File Formats/
│   ├── 7.1_row_and_columnar_formats.md
│   ├── 7.2_partitioning_and_file_layout.md
│   └── 7.3_schema_evolution.md
├── 8 Hadoop and Hive/
│   ├── 8.1_hdfs_and_yarn.md
│   └── 8.2_hive_tables_and_metastore.md
├── 9 Spark/
│   ├── 9.1_what_is_spark.md
│   ├── 9.2_spark_execution_model.md
│   ├── 9.3_dataframe_and_spark_sql.md
│   └── 9.4_partitions_and_shuffle.md
├── 10 Orchestration/
│   └── 10.1 Airflow/
│       ├── 10.1.1_airflow_role_in_data_pipelines.md
│       ├── 10.1.2_task_operator_and_task_instance.md
│       ├── 10.1.3_scheduler_executor_and_worker.md
│       ├── 10.1.4_schedules_and_data_intervals.md
│       └── 10.1.5_retries_and_reruns.md
├── 11 Data Quality and Observability/
│   ├── 11.1_data_quality_checks.md
│   ├── 11.2_reconciliation.md
│   └── 11.3_monitoring_alerting_and_lineage.md
├── 12 Data Architecture/
│   ├── 12.1_batch_and_streaming.md
│   └── 12.2_data_lake_warehouse_and_lakehouse.md
└── 13 Infrastructure/
    ├── 13.1_object_storage.md
    └── 13.2_environments_and_deployment.md
```
