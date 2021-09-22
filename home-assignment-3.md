## Физический уровень PostgreSQL
- создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE типа e2-medium в default VPC в любом регионе и зоне, например us-central1-a

```bash
gcloud beta compute instances create postgres-hw2 \
--machine-type=e2-medium \
--image-family ubuntu-2004-lts \
--image-project=ubuntu-os-cloud \
--boot-disk-size=10GB \
--restart-on-failure
```

- поставьте на нее PostgreSQL через sudo apt
```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update && sudo apt-get -y install postgresql
```

- проверьте что кластер запущен через sudo -u postgres pg_lsclusters **[✓]**
- зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым postgres=# create table test(c1 text); postgres=# insert into test values('1'); \q **[✓]**
- остановите postgres например через `sudo -u postgres pg_ctlcluster 13 main stop` **[✓]**
- создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB **[✓]**
- добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk **[✓]**
- проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux **[✓]**
- сделайте пользователя postgres владельцем /mnt/data - `chown -R postgres:postgres /mnt/data/` **[✓]**
- перенесите содержимое /var/lib/postgres/13 в /mnt/data - `sudo mv /var/lib/postgresql/13 /mnt/data` **[✓]**
- попытайтесь запустить кластер - `sudo -u postgres pg_ctlcluster 13 main start` **[✓]**
```bash
sudo -u postgres pg_ctlcluster 13 main start

Error: /var/lib/postgresql/13/main is not accessible or does not exist
```

- напишите получилось или нет и почему
**как видно выше - не получилось. Потому что данные перенесли.**

- задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/10/main который надо поменять и поменяйте его
напишите что и почему поменяли
```
# в /etc/postgresql/13/main/postgresql.conf ищем data_directory
# и менеяем /var/lib/postgresql/13/main -> /mnt/data/13/main 
```

- попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 13 main start **[✓]**
- напишите получилось или нет и почему

**кластер запустился, потому что указали в конфиге не новые данные**

- зайдите через через psql и проверьте содержимое ранее созданной таблицы
```bash
sudo -u postgres psql
```

```sql
select c1 from test;
 c1 
----
 1
(1 row)
```