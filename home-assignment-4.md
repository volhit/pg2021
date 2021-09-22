## Установка PostgreSQL
- сделать в GCE инстанс с Ubuntu 20.04
```bash
gcloud beta compute instances create postgres-hw3 \
--machine-type=n1-standard-1 \
--image-family ubuntu-2004-lts \
--image-project=ubuntu-os-cloud \
--boot-disk-size=10GB \
--tags=postgres \
--restart-on-failure
```

- поставить на нем Docker Engine
```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

- сделать каталог /var/lib/postgres
```bash
sudo mkdir -p /var/lib/postgres
```

- развернуть контейнер с PostgreSQL 13 смонтировав в него /var/lib/postgres
```bash
sudo docker network create pg-net
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:13

sudo docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
933c79ea2511   postgres:13   "docker-entrypoint.s…"   12 seconds ago   Up 7 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-server
```

- развернуть контейнер с клиентом postgres
```bash
sudo docker run -it --rm --network pg-net --name pg-client postgres:13 psql -h pg-server -U postgres
```

- подключится из контейнера с клиентом к контейнеру с сервером и сделатьтаблицу с парой строк
```sql
create table t(number int);
insert into t values (1), (2), (3), (4);
select * from t;
 number
--------
      1
      2
      3
      4
(4 rows)
```

- подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP
```bash
# открываем порт 5432 для этой вирт. машины в настройках firewall gcp
psql -h 35.205.234.18 -U postgres
```

- удалить контейнер с сервером
```bash
sudo docker rm -f pg-server
```

- создать его заново
```bash
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:13
```

- подключится снова из контейнера с клиентом к контейнеру с сервером
```bash
sudo docker run -it --rm --network pg-net --name pg-client postgres:13 psql -h pg-server -U postgres
```

- проверить, что данные остались на месте
```sql
select * from t;
 number
--------
      1
      2
      3
      4
(4 rows)
```