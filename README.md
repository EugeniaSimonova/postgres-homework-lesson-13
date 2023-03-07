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
postgres@ees-09:~$ pgbench -c 50 -j 2 -P 10 -T 60 postgres
pgbench (14.7 (Ubuntu 14.7-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 4406.4 tps, lat 11.261 ms stddev 10.410
progress: 20.0 s, 4417.3 tps, lat 11.322 ms stddev 11.170
progress: 30.0 s, 4635.1 tps, lat 10.778 ms stddev 10.527
progress: 40.0 s, 4635.9 tps, lat 10.787 ms stddev 10.556
progress: 50.0 s, 4500.6 tps, lat 11.113 ms stddev 10.810
progress: 60.0 s, 4480.3 tps, lat 11.156 ms stddev 11.084
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
duration: 60 s
number of transactions actually processed: 270804
latency average = 11.071 ms
latency stddev = 10.775 ms
initial connection time = 64.200 ms
tps = 4513.819785 (without initial connection time)
```

```
postgres@ees-09:~$ pgbench -c 50 -C -j 2 -P 10 -T 30 -M extended postgres
pgbench (14.7 (Ubuntu 14.7-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 380.1 tps, lat 125.404 ms stddev 164.820
progress: 20.0 s, 380.7 tps, lat 128.637 ms stddev 158.021
progress: 30.0 s, 374.6 tps, lat 130.040 ms stddev 187.616
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: extended
number of clients: 50
number of threads: 2
duration: 30 s
number of transactions actually processed: 11404
latency average = 128.272 ms
latency stddev = 170.795 ms
average connection time = 3.242 ms
tps = 379.477015 (including reconnection times)
```

Создала конфигурационный файл в каталоге conf.d cо следующими настройками, большинство из которых рассчитано pgtune. Те, которые меняла, откомментировала, добавила настройки autovacuum, использовала опыт предыдущего ДЗ. Изменение остальных на тестовых данных дают просадку производительности.

```
shared_buffers = 2GB
effective_cache_size = 6GB
maintenance_work_mem = 2048MB // увеличила, параметр определяет максимальное количество ОП для операций типа VACUUM, CREATE INDEX, CREATE FOREIGN KEY.
checkpoint_completion_target = 0.9
checkpoint_timeout=60min // увеличила время между контрольными точками
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
synchronous_commit = off // увеличивает производительность, но снижает надёжность
wal_writer_delay = 200ms
log_autovacuum_min_duration = 0
autovacuum_max_workers = 3
autovacuum_naptime = 15s
autovacuum_vacuum_threshold = 25
autovacuum_vacuum_scale_factor = 0.05
autovacuum_vacuum_cost_delay = 10
autovacuum_vacuum_cost_limit = 1000
fsync=false

```
