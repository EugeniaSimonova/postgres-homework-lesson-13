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
progress: 10.0 s, 4432.2 tps, lat 11.197 ms stddev 10.829
progress: 20.0 s, 4463.9 tps, lat 11.197 ms stddev 10.938
progress: 30.0 s, 4450.8 tps, lat 11.237 ms stddev 10.787
progress: 40.0 s, 4229.8 tps, lat 11.819 ms stddev 11.424
progress: 50.0 s, 4370.5 tps, lat 11.439 ms stddev 10.757
progress: 60.0 s, 4347.3 tps, lat 11.497 ms stddev 11.171
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
duration: 60 s
number of transactions actually processed: 262994
latency average = 11.400 ms
latency stddev = 10.997 ms
initial connection time = 61.989 ms
tps = 4383.173941 (without initial connection time)
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
```
