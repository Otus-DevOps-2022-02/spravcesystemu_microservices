# spravcesystemu_microservices
spravcesystemu microservices repository

# Выполнено ДЗ logging-1. Логирование и распределенная трассировка

1.Подготовлено окружение, собраны образы с новыми тэгами

2.Создана docker-machine logging

3.Подняты контейнеры на удаленной машине kibana, fluentd, zipkin, elasticsearch, post, blackbox-exporter, ui, mongo-exporter, prometheus, node-exporter

4.Знакомство с системой логирования, проведен сбор структурированных/неструктурированных логов

5.Просмотр трейсов в Zipkin


# Выполнено ДЗ monitoring-1. Введение в мониторинг. Системы мониторинга

1.Создана ветка monitoring-1

2.Подготовлено окружение, запущен контейнер docker run --rm -p 9090:9090 -d --name prometheus prom/prometheus

3. docker-machine ip docker-host заходим на веб-интерфейс Prometheus

4. Создаем docker file

5. Определим конфигурацию в prometheus.yml

6.Создадим образ

```
export USER_NAME=username
$ docker build -t $USER_NAME/prometheus .
```

7.Собираем images
```
/src/ui $ bash docker_build.sh
/src/post-py $ bash docker_build.sh
/src/comment $ bash docker_build.sh
```
или сразу из корня репозитория
```
for i in ui post-py comment; do cd src/$i; bash docker_build.sh; cd -; done
```

8.Определяем docker/docker-compose.yml

9.Тестим docker-compose up -d

10.Изучаем Prometheus

11. Пушим образы
```
docker login
Login Succeeded
$ docker push $USER_NAME/ui
$ docker push $USER_NAME/comment
$ docker push $USER_NAME/post
$ docker push $USER_NAME/prometheus
```

 https://hub.docker.com/repository/docker/65435345345/prometheus
 
 https://hub.docker.com/repository/docker/65435345345/ui
 
 https://hub.docker.com/repository/docker/65435345345/post
 
 https://hub.docker.com/repository/docker/65435345345/comment


# Выполнено ДЗ gitlab-ci-1. Устройство Gitlab CI. Построение процесса непрерывной поставки

1.Создана ветка gitlab-ci-1

2.Создаем новую ВМ
```
yc compute instance create \
--name docker-host \
--zone ru-central1-a \
--network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
--create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=50 \
--ssh-key ~/.ssh/id_rsa.pub
```
3.Устанавливаем Docker

4.Создаем необходимые директории м docker-compose файл
```
mkdir -p /srv/gitlab/config /srv/gitlab/data /srv/gitlab/logs
cd /srv/gitlab
touch docker-compose.yml
```
```
web:
  image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: 'gitlab.example.com'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://<YOUR-VM-IP>'
  ports:
    - '80:80'
    - '443:443'
    - '2222:22'
  volumes:
    - '/srv/gitlab/config:/etc/gitlab'
    - '/srv/gitlab/logs:/var/log/gitlab'
    - '/srv/gitlab/data:/var/opt/gitlab'
```
https://docs.gitlab.com/ee/install/docker.html

5.В той же директории, где docker-compose.yml ( /srv/gitlab ) запускаем docker-compose up -d

6.Заходим, проверяем

7.cd /etc/gitlab
gitlab-rake "gitlab:password:reset[root]"

8.Создаем группу и тестовый проект

9.Добавляем ещё один remote к своему локальному infra-репозиторию
```
git checkout -b gitlab-ci-1
git remote add gitlab http://<your-vm-ip>/homework/example.git
git push gitlab gitlab-ci-1
```
10.Создаем в корне репозитория .gitlab-ci.yml следующего содержания
```
stages:
  - build
  - test
  - deploy

build_job:
  stage: build
  script:
    - echo 'Building'

test_unit_job:
  stage: test
  script:
    - echo 'Testing 1'

test_integration_job:
  stage: test
  script:
    - echo 'Testing 2'

deploy_job:
  stage: deploy
  script:
    - echo 'Deploy'
```
11.Пушим и проверяем статус pipeline

12.Получаем token Settings -> CI/CD -> Pipelines -> Runners

13. На сервере, где работает Gitlab CI, выполняем команду:
```
docker run -d --name gitlab-runner --restart always -v /srv/gitlab-
runner/config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock
gitlab/gitlab-runner:latest
```

14.Регистрируем runner 
```
docker exec -it gitlab-runner gitlab-runner register \
--url http://<your-ip>/ \
--non-interactive \
--locked=false \
--name DockerRunner \
--executor docker \
--docker-image alpine:latest \
--registration-token <your-token> \
--tag-list "linux,xenial,ubuntu,docker" \
--run-untagged
```
docker exec -it gitlab-runner gitlab-runner register --help

15.Push с тэгами
```
git commit -am '#4 add logout button to profile page'
git tag 2.4.10
git push gitlab gitlab-ci-1 --tags
```





# Выполнено ДЗ docker-4. Docker: сети, docker-compose

1. Создана ветка docker-4.
2. Следуем командам из методички
```
docker run -ti --rm --network none joffotron/docker-net-tools -c ifconfig
docker run -ti --rm --network host joffotron/docker-net-tools -c ifconfig
docker-machine ssh docker-host ifconfig
docker run --network host -d nginx
docker kill $(docker ps -q)
sudo ln -s /var/run/docker/netns /var/run/netns
sudo ip netns
```
Создадим bridge network
```
docker network create reddit --driver bridge
```
И запустим проект 
```
> docker run -d --network=reddit mongo:latest
> docker run -d --network=reddit <your-dockerhub-login>/post:1.0
> docker run -d --network=reddit <your-dockerhub-login>/comment:1.0
> docker run -d --network=reddit -p 9292:9292 <your-dockerhub-login>/ui:1.0
```
3. Остановим старые копии контейнеров
```
> docker kill $(docker ps -q)
```
Запустим новые
```
> docker run -d --network=reddit --network-alias=post_db --network-
alias=comment_db mongo:latest
> docker run -d --network=reddit --network-alias=post <your-login>/post:
1.0
> docker run -d --network=reddit --network-alias=comment <your-login>/
comment:1.0
> docker run -d --network=reddit -p 9292:9292 <your-login>/ui:1.0
```
4.Заходим по http://51.250.88.235:9292/ и проверяем, что всё ок.

5.Создадим docker-сети
```
docker network create back_net --subnet=10.0.2.0/24
docker network create front_net --subnet=10.0.1.0/24
```
Запустим контейнеры и проверяем, что всё ок
```
> docker run -d --network=front_net -p 9292:9292 --name ui <your-login>/ui:1.0
> docker run -d --network=back_net --name comment <your-login>/comment:1.0
> docker run -d --network=back_net --name post <your-login>/post:1.0
> docker run -d --network=back_net --name mongo_db \
--network-alias=post_db --network-alias=comment_db mongo:latest
```
Подключим контейнеры ко второй сети
```
> docker network connect front_net post
> docker network connect front_net comment
```
6.Далее заходим по ssh на docker-host и ставим пакет bridge-utils
```
docker-machine ssh docker-host
sudo apt-get update && sudo apt-get install bridge-utils
docker network ls
```
```
NETWORK ID     NAME        DRIVER    SCOPE
04c727d9191e   back_net    bridge    local
6673c455c2e6   bridge      bridge    local
055192f26457   docker-4    bridge    local
bbcf771799ed   front_net   bridge    local
5d78a632abd2   host        host      local
42e06b8d43db   none        null      local
44e5e68c196c   test        bridge    local
7156dd2fa22b   reddit      bridge    local
```

7.Далее выполняем 
```
ifconfig | grep br
brctl show <interface> 
```

Затем 
```
sudo iptables -nL -t nat
ps ax | grep docker-proxy
```

8.Устанавливаем docker-compose и запускаем docker-compose.yml в директории src
9.Выполняем:	
```
docker kill $(docker ps -q)
export USERNAME=<your-login>
docker-compose up -d
docker-compose ps
```
10.Проверяем, что всё запустилось.

11.Редактируем docker-compose file.

12. Чтобы задать имя проекта, его необходимо запустить с ключём -p, или указать назание контейнеров с помощью container_name
```
docker-compose -p project-name up -d
```

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
