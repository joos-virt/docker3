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
1. voting.app
2. in-memory DB
3. 








