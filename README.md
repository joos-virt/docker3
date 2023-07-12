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
docker ps -a
docker stop <container id / name>
docker rm <container id / name>

```


