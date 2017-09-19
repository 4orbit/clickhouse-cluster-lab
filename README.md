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

h2. Без Zookeeper -- тест как работает

Данные при загрузке в распределенную таблицу -- залились на шарды равномерно примерно.
Данные на репликах залились тоже и соответствуют данным на мастерах.

При схеме с тремя шардами и репликой каждого шарда, запросы к распределенной таблице работают при наличии мастера ИЛИ реплики из каждого шарда. Это без использования ZooKeeper -- только то, что есть в metrika.xml. Если запрос делается с мастера ???

При запросе к распределенной таблице с реплики -- требуется чтоб все мастера были доступны для этой таблицы. Но после выключения мастера и реплики -- вообще не работает, а затем достаточно включить что-то одно, хватает теперь уже и реплики только. Те можно сделать, что будут работать только одни реплики. (Хотя изначильно этого не получилось).

Наверно получается так при первом запросе все равно где по распределенной таблице -- все хотят, чтоб были доступны все мастера для нее. А после можно уже вместо мастеров оставлять реплики -- будет работать.

h2. Вместе с Zookeeper

Устанавливаем на 3 сервера zk01-[1-3] 
Делаем 1 шaблон с явой и zk отдельностосящий. Шаблон размножаем.

Раскомментил для сетевой конфигурации там где сам сервер нужно писать 0.0.0.0 и записал
в myid циферки.

interserver_http_host -- пишут нужно выставить, чтоб без сюрпризов

autopurge -- нужно чтоб работало -- обратить внимание на этот параметр.

Нужно думать о ключе шардирования

В простейшем случае ключом шардирования может быть случайное число, т. е. результат вызова функции rand(). Однако в качестве ключа шардирования рекомендуется брать значение хеш-функции от поля в таблице, которое позволит, с одной стороны, локализовать небольшие наборы данных на одном шарде, а с другой — обеспечит достаточно ровное распределение таких наборов по разным шардам в кластере. Например, идентификатор сессии (sess_id) пользователя позволит локализовать показы страниц одному пользователю на одном шарде, при этом сессии разных пользователей будут распределены равномерно по всем шардам в кластере (при условии, что значения поля sess_id будут иметь хорошее распределение). Ключ шардирования может быть также нечисловым или составным. В этом случае можно использовать встроенную хеширующую функцию cityHash64.
Более сложный способ заключается в том, чтобы вне ClickHouse вычислять нужный шард и выполнять запись напрямую в шардированную таблицу. Сложность здесь обусловлена тем, что нужно знать набор доступных узлов-шардов. Однако в этом случае запись становится более эффективной, и механизм шардирования (определения нужного шарда) может быть более гибким.



https://groups.google.com/forum/#!topic/clickhouse/0I7ohN8k5Ns -- интересно



