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

