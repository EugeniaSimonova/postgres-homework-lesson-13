# Настройка PostgreSQL 

Описание/Пошаговая инструкция выполнения домашнего задания:
• развернуть виртуальную машину любым удобным способом
• поставить на неё PostgreSQL 14 любым способом
• настроить кластер PostgreSQL 14 на максимальную производительность не
обращая внимание на возможные проблемы с надежностью в случае
аварийной перезагрузки виртуальной машины
• нагрузить кластер через утилиту
https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench) или через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
• написать какого значения tps удалось достичь, показать какие параметры в
какие значения устанавливали и почему

Использовала утилиту pgbench:

```
pgbench -c 50 -j 2 -P 10 -T 30 postgres

pgbench -c 50 -C -j 2 -P 10 -T 30 -M extended postgres
```

Я использовала следующие настройки

```
shared_buffers = 2GB
effective_cache_size = 6GB
maintenance_work_mem = 2048MB
checkpoint_completion_target = 0.9
checkpoint_timeout=60min
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 5242kB
min_wal_size = 1GB
max_wal_size = 4GB
max_worker_processes = 4
max_parallel_workers_per_gather = 2
max_parallel_workers = 4
max_parallel_maintenance_workers = 2
synchronous_commit = off
wal_writer_delay = 200ms
log_autovacuum_min_duration = 0
autovacuum_max_workers = 2
autovacuum_naptime = 15s
autovacuum_vacuum_threshold = 25
autovacuum_vacuum_scale_factor = 0.05
autovacuum_vacuum_cost_delay = 10
autovacuum_vacuum_cost_limit = 1000

```
