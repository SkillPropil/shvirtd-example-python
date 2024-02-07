### ЗАДАЧА 1
Написал образ, используя указанный образ:
```
root@msk1wst405n:/opt/shvirtd-example-python# cat Dockerfile.python
FROM python:3.9-slim

LABEL AUTHOR=n.uchaev@omp.ru

WORKDIR /app

COPY requirements.txt .

RUN   pip install -r requirements.txt \
      && python3 -m venv venv

COPY main.py .

EXPOSE 5000

CMD ["python", "main.py"]
```

### Задача 2(*)
Так как долго отсутствовал - возникли трудности с логином и доступом до реджистри, но я все таки смог:)
```
docker build -f Dockerfile.python -t cr.yandex/crp2vti56sp972lnd866/web-python:1.0.0 .
```
Список контейнеров
```
[root@uchaev-netology-2 ~]# yc container image list
+----------------------+---------------------+---------------------------------+-------+-----------------+
|          ID          |       CREATED       |              NAME               | TAGS  | COMPRESSED SIZE |
+----------------------+---------------------+---------------------------------+-------+-----------------+
| crp8priga190arc4vntj | 2024-02-07 19:07:25 | crp2vti56sp972lnd866/web-python | 1.0.0 | 95.9 MB         |
+----------------------+---------------------+---------------------------------+-------+-----------------+
```
Просканировал образ
```
[root@uchaev-netology-2 ~]# yc container image scan crp8priga190arc4vntj
done (31s)
id: chepsm70ugf39pjih0m9
image_id: crp8priga190arc4vntj
scanned_at: "2024-02-07T22:57:38.647Z"
status: READY
vulnerabilities:
  critical: "1"
  high: "4"
  medium: "26"
  low: "65"
```
### Задача 3
```
[root@uchaev-netology-2 shvirtd-example-python]# cat compose.yaml
version: "3"
services:
  db:
    image: mysql:8
    container_name: db
    restart: always
    networks:
      backend:
        ipv4_address: ${DB_HOST}
    environment:
      - MYSQL_ROOT_HOST="%"
      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}
      - MYSQL_DATABASE=${DB_NAME}
      - MYSQL_USER=${DB_USER}
      - MYSQL_PASSWORD=${DB_PASSWORD}
    volumes:
      - ./db_data:/var/lib/mysql
    ports:
      - "3306"


  web:
    image: cr.yandex/crp2vti56sp972lnd866/web-python:1.0.0
    container_name: web
    depends_on:
      - db
    restart: always
    environment:
      - DB_HOST=${DB_HOST}
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_USER=${DB_USER}
      - DB_NAME=${DB_NAME}
    volumes:
      - ./venv:/bin/activate
    networks:
      backend:
        ipv4_address: 172.20.0.5
    ports:
    - "5000"
include:
  - proxy.yaml
```
файл окружения
```
[root@uchaev-netology-2 shvirtd-example-python]# cat .env
DB_HOST=172.20.0.10
DB_USER=app
DB_PASSWORD=passw0rd
DB_NAME=netology_db
DB_ROOT_PASSWORD=panin
```
Курлы
```
[root@uchaev-netology-2 shvirtd-example-python]# curl -L http://127.0.0.1:8080
TIME: 2024-02-07 23:11:16, IP: None[root@uchaev-netology-2 shvirtd-example-python]#
[root@uchaev-netology-2 shvirtd-example-python]# curl -L http://127.0.0.1:8090
TIME: 2024-02-07 23:11:21, IP: 127.0.0.1[root@uchaev-netology-2 shvirtd-example-python]#
```
mysql> use netology_db; select * from requests;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
БД
```
Database changed
+----+---------------------+---------------+
| id | request_date        | request_ip    |
+----+---------------------+---------------+
|  1 | 2024-02-07 21:17:04 | 90.154.15.122 |
|  2 | 2024-02-07 23:11:16 | NULL          |
|  3 | 2024-02-07 23:11:21 | 127.0.0.1     |
+----+---------------------+---------------+
3 rows in set (0.00 sec)
```
### Задача 4
Написал скриптец, который создает .env файл и записывает в него переменные. Далее запускает compose 
```
root@msk1wst405n:/opt# cat netology.sh
#!/bin/bash
git clone https://github.com/SkillPropil/shvirtd-example-python /opt/shvirtd-example-python
cd /opt/shvirtd-example-python
array=(DB_ROOT_PASSWORD DB_USER DB_PASSWORD DB_NAME DB_HOST)
rm -f .env
for key in ${array[*]}
do
  keyupd=$(printf "%s\n" $key)
  echo "Please fill variables $keyupd"
  read value
  echo "$keyupd=$value" >> .env
done
docker compose up -d
```
Запускаем
```
root@msk1wst405n:/opt# ./netology.sh
Cloning into '/opt/shvirtd-example-python'...
remote: Enumerating objects: 214, done.
remote: Counting objects: 100% (181/181), done.
remote: Compressing objects: 100% (40/40), done.
remote: Total 214 (delta 141), reused 173 (delta 137), pack-reused 33
Receiving objects: 100% (214/214), 6.04 MiB | 7.49 MiB/s, done.
Resolving deltas: 100% (143/143), done.
Please fill variables DB_ROOT_PASSWORD
123456
Please fill variables DB_USER
panin
Please fill variables DB_PASSWORD
123456
Please fill variables DB_NAME
dog
Please fill variables DB_HOST
172.20.0.10
[+] Running 34/34
 ✔ ingress-proxy 6 layers [⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                                                                                8.5s
   ✔ 9b0163235c08 Pull complete                                                                                                                                                                          4.4s
   ✔ f24a6f652778 Pull complete                                                                                                                                                                          4.6s
   ✔ 9f3589a5fc50 Pull complete                                                                                                                                                                          4.9s
   ✔ f0bd99a47d4a Pull complete                                                                                                                                                                          5.0s
   ✔ 398157bc5c51 Pull complete                                                                                                                                                                          5.2s
   ✔ 1ef1c1a36ec2 Pull complete                                                                                                                                                                          5.5s
 ✔ web 9 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                                                                                       6.6s
   ✔ c57ee5000d61 Pull complete                                                                                                                                                                          1.1s
   ✔ be0f2e005f57 Pull complete                                                                                                                                                                          0.4s
   ✔ e1109e6b1ef2 Pull complete                                                                                                                                                                          1.0s
   ✔ 01b5e78a2f3c Pull complete                                                                                                                                                                          0.8s
   ✔ f46622efb352 Pull complete                                                                                                                                                                          1.6s
   ✔ 51404d11ae24 Pull complete                                                                                                                                                                          1.2s
   ✔ 2ad910d6947a Pull complete                                                                                                                                                                          1.3s
   ✔ 849f14c168d6 Pull complete                                                                                                                                                                          2.7s
   ✔ a0ddcec26894 Pull complete                                                                                                                                                                          1.5s
 ✔ reverse-proxy 5 layers [⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                                                                                11.1s
   ✔ 341cf697875f Pull complete                                                                                                                                                                          5.6s
   ✔ 5bb447692b62 Pull complete                                                                                                                                                                          5.7s
   ✔ 64e73485cc2c Pull complete                                                                                                                                                                          7.4s
   ✔ 9d2a73a9b5e1 Pull complete                                                                                                                                                                          6.2s
   ✔ 4f4fb700ef54 Pull complete                                                                                                                                                                          6.3s
 ✔ db 10 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                                                                                     14.6s
   ✔ b8307a22608d Pull complete                                                                                                                                                                          1.6s
   ✔ 32f3542f2ba9 Pull complete                                                                                                                                                                          0.6s
   ✔ 795385976c83 Pull complete                                                                                                                                                                          1.1s
   ✔ 03b553142f26 Pull complete                                                                                                                                                                          3.9s
   ✔ d8ce6174541f Pull complete                                                                                                                                                                          1.7s
   ✔ 681c64901273 Pull complete                                                                                                                                                                          2.0s
   ✔ 27db2241c611 Pull complete                                                                                                                                                                          4.1s
   ✔ db8e96eb91f6 Pull complete                                                                                                                                                                          2.3s
   ✔ 28b560af9a2a Pull complete                                                                                                                                                                          4.4s
   ✔ d0c9925abfdf Pull complete                                                                                                                                                                          2.9s
[+] Running 5/5
 ✔ Network shvirtd-example-python_backend            Created                                                                                                                                             0.2s
 ✔ Container shvirtd-example-python-reverse-proxy-1  Started                                                                                                                                             0.7s
 ✔ Container db                                      Started                                                                                                                                             0.7s
 ✔ Container shvirtd-example-python-ingress-proxy-1  Started                                                                                                                                             0.7s
 ✔ Container web                                     Started
 ```
```
Курлы 2
root@msk1wst405n:/opt# curl -I http://158.160.141.190:8090
HTTP/1.1 200 OK
Server: nginx/1.25.3
Date: Wed, 07 Feb 2024 23:18:05 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 44
Connection: keep-alive
```
Запись в базе
```
mysql> use netology_db; select * from requests;
Database changed
+----+---------------------+---------------+
| id | request_date        | request_ip    |
+----+---------------------+---------------+
|  1 | 2024-02-07 21:17:04 | 90.154.15.122 |
|  2 | 2024-02-07 23:11:16 | NULL          |
|  3 | 2024-02-07 23:11:21 | 127.0.0.1     |
|  4 | 2024-02-07 23:18:05 | 90.154.15.122 |
+----+---------------------+---------------+
4 rows in set (0.00 sec)
```
### Задача 6
Запустил dive, посмотрел информацию об уровнях и docker-save вытащил бинарник

```
docker save b404d8f4983a terraform.tar
tar -xvf terraform.tar
tar -xvf 964bf29117b2de3e4668ea9be8cd353f16d5f06777e7f2fb6d6c7a3eeac5b995/layer.tar
bin/
bin/terraform
```
### Задача 6.1
root@msk1wst405n:/opt# docker run -it -d  --entrypoint=/bin/sh hashicorp/terraform:latest
docker cp terraform c0fc29db863f:/bin/terraform
