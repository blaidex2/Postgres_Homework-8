## Домашнее задание №8

• развернуть виртуальную машину любым удобным способом
Развернута ВМ Ubuntu: 4 ГБ ОЗУ, 50 ГБ ПЗУ, 4 ЦПУ

![image](https://github.com/blaidex2/Postgres_Homework-8/assets/130083589/ae0f1e27-6a8f-435c-9bbb-4a6b2b4dfb92)

 
• поставить на неё PostgreSQL 15 любым способом

>sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-15

• настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае
аварийной перезагрузки виртуальной машины
• нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
• написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему

Тестирование производительности с помощью утилиты pgbench будем выполнять со следующими параметрами: в течение 1 минуты 1 рабочий процесс pgbench будет имитировать транзакции одновременно от 50 клиентов, выводя результат каждые 15 секунд.

>sudo -u postgres pgbench -i postgres
>
>sudo -u postgres pgbench -c50 -P 15 -T 60 -U postgres postgres

Тест с настройками по умолчанию: tps = 694

![image](https://github.com/blaidex2/Postgres_Homework-8/assets/130083589/85933f81-57e1-4c13-a055-2d5c5142d929)

Выключим синхронный режим:

>alter system set synchronous_commit = 'off';
>
>sudo systemctl restart postgresql
>
>show synchronous_commit;


Тест с включенным асинхронным режимом: tps = 1543

![image](https://github.com/blaidex2/Postgres_Homework-8/assets/130083589/dcde3c16-fb81-4fb3-987d-801119d1bad0)


Выполним настройку постгре по следующим параметрам:

>sudo nano /etc/postgresql/15/main/postgresql.conf

| Параметр | Значение | Комментарий |
|----------|---------------------------------|----------------------------------|
|shared_buffers|512 MB| 25% RAM|
|max_connections|55| Кол-во потоков в тесте + 10%|
|effective_cache_size| 1536 MB| 75% RAM|
|work_mem| 10 MB|25% RAM / max_connections|
|maintenance_work_mem| 64 MB| |
|wal_buffers|16 MB||
|min_wal_size|4 GB||
|max_wal_size|16 GB||
|checkpoint_timeout|30 min||
|effective_io_concurrency|200| |


Тест с измененными настройками: tps =1543

![image](https://github.com/blaidex2/Postgres_Homework-8/assets/130083589/d6a822b9-0377-43d4-a969-0c9687c95e8a)

 

Выполним настройку постгре по следующим параметрам:

| Параметр | Значение | Комментарий |
|----------|---------------------------------|----------------------------------|
|shared_buffers|820 MB| 40% RAM|
|max_connections|55| Кол-во потоков в тесте + 10% (не изменено)|
|effective_cache_size| 1536 MB| 75% RAM (не изменено)|
|work_mem| 10 MB|25% RAM / max_connections (не изменено)|
|maintenance_work_mem| 512 MB| |
|wal_buffers|16 MB| не изменено|
|min_wal_size|4 GB| не изменено|
|max_wal_size|16 GB| не изменено|
|checkpoint_timeout|60 min||
|effective_io_concurrency|1000| |




Тест с измененными настройками: tps = 1542

![image](https://github.com/blaidex2/Postgres_Homework-8/assets/130083589/07d7c258-1dce-4238-9d79-98f655fa5ab3)


> При включенном асинхронном режиме настройки постгре дают приблизительно одинаковые результаты производительности, а при выключенном асинхронном режиме изменение конфигурации дает более значимые результаты производительности.

Задание со *: аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench)

Выполним установку утилиты: 

>curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench
git clone https://github.com/Percona-Lab/sysbench-tpcc
>

Установим пароль для пользователя postgres:

>alter user postgres password 'psswd';


Подготовим данные для тестирования:

>./tpcc.lua --pgsql-user=postgres --pgsql-db=postgres --pgsql-password=psswd --time=60 --threads=3 --report-interval=30 --tables=10 --scale=5 --use_fk=0  --trx_level=RC --db-driver=pgsql prepare

Тестирование производительности с помощью утилиты pgbench будем выполнять со следующими параметрами:

>./tpcc.lua --pgsql-user=postgres --pgsql-db=postgres --pgsql-password=psswd --time=60 --threads=3 --report-interval=30 --tables=10 --scale=5 --use_fk=0  --trx_level=RC --db-driver=pgsql run

Тест с настройками по умолчанию: tps = 124

![image](https://github.com/blaidex2/Postgres_Homework-8/assets/130083589/b8a95375-6e3b-4de2-b899-ffc4b99f735a)

Тест с включенным асинхронным режимом: tps =179

![image](https://github.com/blaidex2/Postgres_Homework-8/assets/130083589/ccca8e61-64ba-406c-8b95-f8b260e0b212)


Выполним настройку постгре по следующим параметрам:

| Параметр | Значение | Комментарий |
|----------|---------------------------------|----------------------------------|
|shared_buffers|512 MB| 25% RAM|
|max_connections|55| Кол-во потоков в тесте + 10%|
|effective_cache_size| 1536 MB| 75% RAM|
|work_mem| 10 MB|25% RAM / max_connections|
|maintenance_work_mem| 64 MB| |
|wal_buffers|16 MB||
|min_wal_size|4 GB||
|max_wal_size|16 GB||
|checkpoint_timeout|30 min||
|effective_io_concurrency|200| |


Тест с измененными настройками: tps = 186

![image](https://github.com/blaidex2/Postgres_Homework-8/assets/130083589/81a53b5d-936a-45d2-90f1-1fd1f5862457)

 
Выполним настройку постгре по следующим параметрам:

| Параметр | Значение | Комментарий |
|----------|---------------------------------|----------------------------------|
|shared_buffers|820 MB| 40% RAM|
|max_connections|55| Кол-во потоков в тесте + 10% (не изменено)|
|effective_cache_size| 1536 MB| 75% RAM (не изменено)|
|work_mem| 10 MB|25% RAM / max_connections (не изменено)|
|maintenance_work_mem| 512 MB| |
|wal_buffers|16 MB| не изменено|
|min_wal_size|4 GB| не изменено|
|max_wal_size|16 GB| не изменено|
|checkpoint_timeout|60 min||
|effective_io_concurrency|1000| |


Тест с измененными настройками: tps = 229

![image](https://github.com/blaidex2/Postgres_Homework-8/assets/130083589/04418a34-be40-4523-ad36-b3c4e789d648)

 

> Вывод: 
Утилита sysbench в отличии от pgbench видит разницу в производительности в зависимости от конфигурации при включенном асинхронном режиме
