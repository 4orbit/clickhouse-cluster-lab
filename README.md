Тестовый кластер ClickHouse на lxc.

TODO

1) развернуть
2) потестить шардинг
3) потестить репликацию

Подготовка
----------

Создал шаблон для клонирования:
* ch-templ -- ubuntu xenial 16.04 с установленным clickhouse-server-common, clickhouse-server-base, clickhouse-client -- тут нет файла для тестовой БД -- его можно брать так
<pre>
cp /var/lib/lxc/ch01/rootfs/opt/ontime.csv.xz /var/lib/lxc/my777/rootfs/opt/
</pre>
где my777 -- нужный сервер

Создал следующие сервера:

ch01-01-1
ch01-01-2
ch01-02-1
ch01-02-2
ch01-03-1
ch01-03-2

for i in 1 2 3 ; do for j in 1 2 ; do lxc-start -n ch01-0$i-$j ; done ; done
for i in 1 2 3 ; do for j in 1 2 ; do lxc-stop  -n ch01-0$i-$j ; done ; done

запускаем и останавливаем все ноды

clickhouse\_remote\_servers -- прописываем в /etc/metrika.xml

https://stackoverflow.com/questions/39095443/where-should-the-remote-server-element-located-when-configurating-clickhouse-clu


после можно 

select * from  system.clusters;

Место на диске до загрузки данных:
18G	/var/lib/lxc/ch01
737M	/var/lib/lxc/ch01-01-1
737M	/var/lib/lxc/ch01-01-2
737M	/var/lib/lxc/ch01-02-1
737M	/var/lib/lxc/ch01-02-2
736M	/var/lib/lxc/ch01-03-1
736M	/var/lib/lxc/ch01-03-2

в серверных конфигах нужно раскомментить
<listen_host>::</listen_host>
чтоб не было проблем с сетью

таблицу, например ontime\_local -- нужно руками создать на всех хостах
таблицу же объединенную ontime\_all -- на том откуда запросы но можно на всех хостах

при этом на репликах основных серверов таблицы заполнились как на праймари
т.е. на ch01-03-1 и ch01-03-2 -- одинаково

записать данные в шардированную таблицу получилось только именно таким спрособом, который описан
в быстром старте: сначала делаем полную таблицу на одном из серверов -- после в распределенную

<pre>
INSERT INTO ontime_all SELECT * FROM ontime;
</pre>

в моем случае с lxc при загрузки из сжатого файла праямо и даже из csv -- загрузка падала из-за ошибок в сетевом взаимодействии при большой нагрузке общей на комп.


