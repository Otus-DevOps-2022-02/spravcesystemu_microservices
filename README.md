# spravcesystemu_microservices
spravcesystemu microservices repository

 # Выполнено ДЗ docker-3. Docker-образы. Микросервисы
 
 1. Создана ветка docker-3. 
 2. Смотрим вывод команды docker-machine ls 

```
NAME          ACTIVE   DRIVER    STATE     URL                        SWARM   DOCKER      ERRORS
docker-host   *        generic   Running   tcp://51.250.88.235:2376           v20.10.16
```
3.Копируем каталог src к себе на машину и правим Dockerfiles
4. Скачиваем образ монги
```
docker pull mongo:latest
```
5.Собираем наши образы
```
docker build -t <your-dockerhub-login>/post:1.0 ./post-py
docker build -t <your-dockerhub-login>/comment:1.0 ./comment
docker build -t <your-dockerhub-login>/ui:1.0 ./ui
```

6.Создаем сеть для контейнеров 
```
docker network create reddit
```

7.Запускаем
```
docker run -d --network=reddit \
    --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit \
    --network-alias=post <your-dockerhub-login>/post:1.0
docker run -d --network=reddit \
    --network-alias=comment <your-dockerhub-login>/comment:1.0
docker run -d --network=reddit \
    -p 9292:9292 <your-dockerhub-login>/ui:1.0
```
8.Проверяем, что всё работает:
http://51.250.88.235:9292/

![image](https://user-images.githubusercontent.com/89079372/169828746-417be745-eb38-4d1c-99ca-5a943661be32.png)





  # Выполнено ДЗ docker-2
Технология контейнеризации.Введение в Docker.

1.В новой ветке docker-2 была создана директорию docker-monolith

2.Ставим Docker – 19.03+, docker-compose – 1.26+, docker-machine – 0.16+

3.Используем основные команды docker

4.Создаем docker host в yandex cloud и инициализируем окружение docker

```
yc compute instance create \
  --name docker-host \
  --zone ru-central1-a \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15 \
  --ssh-key ~/.ssh/id_rsa.pub
	
docker-machine create \
  --driver generic \
  --generic-ip-address=84.201.135.110 \
  --generic-ssh-user yc-user \
  --generic-ssh-key ~/.ssh/id_rsa \
  docker-host
  
eval $(docker-machine env docker-host) 
``` 

5.Проверяем, что хост создан командой 
```
docker-machine ls
```
6.И начинаем с ним работу 
```
eval $(docker-machine env docker-host)
```

7. Собираем образ
```
docker build -t reddit:latest .
```
Флаг -t задает тег для собранного образа

8. docker images -a
9. docker run --name reddit -d --network=host reddit:latest
10. Открываем в браузере http://51.250.9.100:9292/
