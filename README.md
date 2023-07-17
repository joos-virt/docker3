# docker3
Docker практика

Без докера имеем:
- Проблемы с совместимость
- Проблемы зависимости
- Длительная настройка среды
- Разные среды (Разработка, тестирование, продакшн)

Основное назначение докер - контейнеризация приложений

Виртуальная машина в отличии от докера потребляет больше ресурсов, занимает больше места, дольше грузится, 

Docker образ это пакет или шаблон аналогичной шаблону виртуальной машины

Контейнер - запущенный образ
```
sudo docker run docker/whalesay cowsay Hi, JooS!
```
```
docker run alpine
docker run -it alpine sh
docker ps
docker run -d apline sleep 20
docker attach <container id / name>
docker ps -a
docker stop <container id / name>
docker rm <container id / name>
docker rm $(docker ps -aq)
docker pull nginx
docker exec <container id / name> cat /etc/nginx/nginx.conf
```
docker run
```
docker run reddis
docker run reddis:6.0.9
-t - terminal
-i - interactive
docker run -it rotorocloud/promt-docker - позволит связать консоль контейнера с нашей и ввести значение
echo Joos | docker run -i rotorocloud/promt-docker - подставить значение  автоматом
-p - проброс портов
docker run -p 80:5000 rotorocloud/webapp - проброс порта
-v - volume mapping - директория для хранения
docker run -v /opt/data:/var/lib/mysql mysql - директория для сохранения данных
docker inspect <container id / name>
docker logs <container id / name>
```
```
-d - detach - запуск контейнера в фоновом режиме
docker run -d apline sleep 20
docker attach <container id / name> - присоединиться к контейнету
docker stop <container id / name> - останавливаем контейнер
docker run mingrammer/flog flog -d 1 -500 - контейнер гененирует события 500 сек и выводит через 1 сек
docker run jenkins/jenkins:lte-alpine - ставим дженкинс контейнер - доступ только по внутреннему адресу конейнера (и так как обысно сеть - мост, то можно зайти с хоста)
docker run -p 8080:8080 jenkins/jenkins:lte-alpine - пробрасываем порт и получаем доступ по адресу хоста
- docker inspect <container id / name> - появляется PortBindings - указывает какие порты и куда проброшены
docker run -p 8080:8080 -v /home/joos:/var/jenkins_home -u root jenkins/jenkins:lte-alpine - добавляем том и запускаем от рута
```
**Docker image - Docker file**

Пример надо
1. OS Ubuntu
2. Обвновить apt репо
3. Установить зависимости apt
4. Установить зависимости pip
5. Скопировать код в /opt
6. Запустить с параметром 'flask'
```Dockerfile
FROM ubuntu - назначаем базовый образ
RUN apt-get update
RUN apt-get install -y python3 python3-pip - настраиваем зависимости
RUN pip3 install flask - устанавливаем нужные версии, бибилиотеки
COPY app.py /opt/app.py - копируем исходный код
ENTRYPOINT FLASK_APP=/opt/app.py flask run --host=0.0.0.0 --port=5000 - указывает точку входа
```
**Dockerfile**

ИНСТРУКЦИЯ - АРГУМЕНТ
```
docker build . -t rotoro/webapp - собирает Dockerfile
docker push rotorocloud/webapp
docker history rotorocloud/webapp - смотрим слои
```
Докер собирает все слоями, при пересборке берет уже готовые слои из кэша

export DOCKER_HOST=IP_Adress удаленного сервера - переключаем  докер-консоль на удаленный сервер

cat > app.py - Пишем в файл
```
код
Ctrl + C
```
```
docker build . - собираем образ
dicker tag <container id> rotorocloud/webapp:ubuntu - даем имя и тег образу
docker login
dovker push rotorocloud/webapp:ubuntu - грузим образ в докер хаб
```
export DOCKER_HOST= - возвращаем стандартный (локальный) докер хост

**Переменные**
```
-e - задает переменную
docker run -e ROCKET_SIZE=big rotorocloud/simple-webapp-rockets
docker run -e ROCKET_SIZE=small rotorocloud/simple-webapp-rockets
```
docker inspect <container id / name> - в разделе "Env" покажет укзанные переменные

CMD sleep 5 - указываем команду запуска
```
FROM ubuntu
CMD sleep 10
```
docker build . -t ubuntu-sleeper

= docker run ubuntu-sleeper **sleep 20** - при запуске с CMD можем поменять команду
```
FROM ubuntu
ENTRYPOINT ["sleep"]
```
docker build . -t ubuntu-sleeper2 - команда при старте **sleep**

docker run ubuntu-sleeper **10** - при запуске можем только добавить в команду

если не задать параметр то будет ошибка - missing operand
```
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD sleep 10
```
можно объединить и тогда ENTRYPOINT возмет значение из CMD

docker run --entrypint super-sleep ubuntu-sleeper 10 - заменим команду на super-sleep 10

**Docker Compose**

Пример
1. voting.app - python
2. in-memory DB - redis
3. worker - NET
4. db - postgresql
5. result-app - nodd
```
docker run -d --name=redis redis
docker run -d --name=db postgres:9.4
docker run -d --name=vote -p 5000:80 --link redis:redis voting-app
docker run -d --name=result -p 5001:80 --link db:db result-app
docker run -d --name=worker --link db:db --link redis:redis worker
```
--link - вносит запись в /etc/hosts - ip хост - устаревшая конструкция - надо использовать netwoks
```docker-compose.yml
redis:
  image: redis
db:
  image: postgres:9.4
vote:
  image: dockersamples/voting-app # - если репозитория нет то - buld: ./vote
  ports:
    - 5000:80
  links:
    - redis #(=redis:redis)
result:
  image: dockersamples/result-app  # -  buld: ./result (папка с dockerfile и зависимостями)
  ports:
    - 5001:80
  links:
    - db #(=db:db)
worker:
  image: dockersamples/worker  # -  buld: ./worker
  links:
    - db
    - redis
```
docker-compose up

**version 1** - много ограничений (например нельзя развернуть в разных сетях кроме мостовой, нельзя указать зависимости - например поднять redis посде db)

**version 2** - links не нужен - все что в одном services будет объединено в сеть, появился depends_on
**version 3** - схожа с ver 2, отличие появления оркестрации - docker swarm и работы нескольких хостов
```
version 2 (3)
services:
  redis:
    image: redis
  db:
    image: postgres:9.4
  vote:
    image: dockersamples/voting-app
    ports:
      - 5000:80
    depends_on:
      - redis
```
Разделяем сети
```
version 2 (3)
services:
  redis:
    image: redis
    networks:
      - backend
  db:
    image: postgres:9.4
    networks:
      - backend
  vote:
    image: dockersamples/voting-app
    ports:
      - 5000:80
    networks:
      - frontend
      - backend
  result:
    image: dockersamples/result-app
    ports:
      - 5001:80
    networks:
      - frontend
      - backend
  worker:
    image: dockersamples/worker
    networks:
      - backend
networks:
  frontend:
  backend:
```
```
docker-compose up -d
docker-compose down
docker-compose ps
docker-compose logs
docker-compose up -d --scale vote=3
```

**Docker Engine**

Docker Host = Docker deamon + REST API + Docker CLI

Докер демон это фоновый процесс который управляет объектами докер такими как образы, контейнеры, тома и сети

API Server поддерживает API которой программы могут использовать для общения с демоном и предоставления инструкций

Docker CLI это утилита командной строки которую мы использовали до сих пор для выполнения таких действий как запуск остановка контейнеров уничтожения образов и так далее она использует вызовы REST API для взаимодействия с демоном докер
```
docker -H=remote-docker-engine:2375 - можно со своего компа присоединиться к другому серверу для работы с докером
docker -H=10.10.10.1:2375 run nginx
```
Докер используют пространство имен **namespace** для изоляции идентификаторов процессов рабочего пространства, подключение к сети, система разделения времени. Unix процессы контейнера выполняются в их собственном пространстве имен тем самым обеспечивая изоляцию с другими процессами

**Cgroups** - способ ограничить объем центрального процессора или памяти который может использовать контейнер докер используют контрольные группы си groups чтобы ограничить количество аппаратных ресурсов выделяемых каждому
```
docker run --cpus=.5 nginx
docker run --memory=100m nginx
```

**Хранилища**

В контейнере - /var/lib/docker
- /overlay2  /containers  /image  /volumes - здесь докер по умолчанию хранит все свои данные

Докер все пишет по слоям, берет существующие слои из кэша.

Есть слои образа, есть слой контейнера - все изменения при работающей контейнере попадают в слой контейнера, который живет пока жив контейнер.

Мы можем изменить файл внутри контейнера, но прежде чем я сохраню измененный файл докер автоматически создаст копию файла на уровне чтения и записи и затем я буду изменять и работать уже с другой версии файла которая будет находиться в другом слое, слое контейнера. И чтение и записи все будущие изменения будут внесены в эту копию файла на уровне чтение записи, это называются механизмом копирования при записи copy on write - слои образа доступны только для чтения это означает что файлы в этих слоях не будут изменены на самом деле, образ будет оставаться неизменным все время пока ты не перестроишь его с помощью команды docker build

Том - Volume
```
docker volume create data_volume - создаем папку в /var/lib/docker/volumes
без это коменды, докер создаст автоматом и подключит такую папку при следующей команде -
docker run -v data_volume:/var/lib/mysql mysql - volume mount - монтируем том внутрь контейнера (создаем если не создан)
docker run -v /data/mysql:/var/lib/mysql mysql - bind mount - подключаем к определенной директории
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql mysql - новый стиль, продвигает Docker
```
**Storaga drivers** - драйвер и хранилища для обеспечения многоуровневой архитектуры - несет ответственность за выполнение всех операций - сохранение многоуровневой архитектуры, создание слоев с возможностью записи, перемещение файлов между слоями, для возможности копирования и записи и так далее.

**Некоторые драйверы хранения** (Выбор зависит от исполоьзуемой ос.)
- aufs
- btrfs
- zfs
- device mapper
- overlay
- overlay2
- fuse overlayfs
```
docker info - покажет Storage driver
```
Докер сам постараются выбрать лучший драйвер хранилища который доступен в конкретной операционной системе но это не всегда работает хорошо в случае проблем нужно настраивать руками

**Сети в Docker**

Сети по умолчанию
```
docker run webapp
```
- Bridge - частная внетренняя сеть созданная докер на хосте, контейнеры подключены к ней по умолчанию (172.17.0.1)
```
docker run webapp --network=none
```
- None - контейнеры не являются членами сети и не имеют у другим контейнетам и узлам
```
docker run webapp --network=host
```
- Host - прямое связывание контейнера с сетью хоста, убирает сетевую изоляцию межде хостом и контейнером - контейнер использует все сетевые ресурсы своего хоста
- Overlay - для нагрузок связанных с оркестрацией, адаптирован для работы одной сети на нескольких узлах
- Macvlan - связана с докеризацией нагрузки с виртуальными машинами, полностью эмулирует физический хост части сети
```
docker network create --driver bridge --subnet 182.18.0.0/16 my-network - создаем свою сеть
docker network ls
```
Docker имеет встроенный dns сервер (127.0.0.11), который разрешает имена контейнеров

**Docker Image**

docker run nginx

nginx = image:docker.io/nginx/nginx
- docker.io - Registry (DockerHub)
- nginx - User/Account
- nginx - Image/Repository

gcr.io - google
```
docker login private-registry.io - логин в свой репозиторий
docker run private-registry.io/app/internal-app - запуск из своего репозитория
```
Создаем и используем свой локальный репозиторий
```
docker run -d -p 5000:5000 --name registry registry:2
docker image tag my-image localhost:5000/my-image
docker push localhost:5000/my-image
docker pull localhost:5000/my-image
```
Для взаимодействия по сети нужны сетрификаты

**Kubernetes**
```
kubectl create deployment node --image=node:v1 - запускаем нужное
kubectl scale --replicas=9 node - масштабируем
kubectl scale --replicas=18 node - настраиваем, можем сделать зависимость от нагрузки
kubectl set image deployment node --image=node:v2 - обновление
kubectl rollout undo deployment node - откатываем обновления
```
Кластер - набор сгруппированныз узлов, нод

Ноды - машина на которой установлен кубернетес

Worker Node - узел на котром кубернетес будет запускать контейнеры с нагрузкой

Мастер нода - нода сконфигурированная как мастер - наблюдает за состоянием других нод и ответственна за оркестрацию контейнеров на воркер нодах.

Компоненты:
- API SERVER - frontend - единый интерфейс Kubernetes, все общаются с api server 
- CONTAINER RUNTIME - среда выполнения контейнера - базовое по используется для запуска контейнеров
- CONTROLLER - мозг оркестрации, смотрят за состоянием нод, контейнеров, endpoint-ов и ответственны за реакцию за события на нодах. Принимают решения о создании новых контейнеров.
- SCHEDULER - ответственнен за распределение работ или контейнеров между нодами, ожидает появления контейнера, чтобы назначить его выполнение на ноду
- Служба KUBELET - агент который работает на каждом узле кластера, агент отвечает за то чтобы контейнеры работали на узлах должным образом
- Служба ETCD - хранилище ключ-значение, распределенная надежная БД используемая kubernetes для хранения информации нужной для управления кластера

kubectl (kubecontrol) - используется для развертывания и управления приложениями в кластере кубернетес, для получения информации о кластере, получения состояния других нод в кластере
```
kubectl run hello-minikube - развертывание приложения в кластере
kubectl cluster-info - информация о кластере
kubrctl get nodes - вывод списка всез узлов в кластере
```


