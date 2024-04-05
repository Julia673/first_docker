0. Ставим Docker
```
curl -sSL https://get.docker.com/ | sh # ставим
for us in $(ls /home/); do usermod -aG docker ${us}; done
```

1. Створюємо Dockerfile

```
FROM bigdatateam/spark-python2

USER root
WORKDIR /root
```
2. build
```
docker build -t velkerr/my_jupyter .
docker build velkerr/my_jupyter . --no-cache  # не учитываем cache
```

3. run в інтерактивному режимі
```
docker run --rm -it velkerr/my_jupyter
```

4. Виконати команду у контейнері:
```
docker exec -it e183e4c6e3a9 /bin/bash
```

5. Монтиорвание volumes
```
mkdir dir
mkdir readonly
docker run --rm -it -v ${PWD}/dir:/dir -v ${PWD}/readonly:/root/readonly:ro velkerr/my_jupyter
```

6. Перекидання портів
Додаємо в Dockerfile `EXPOSE 8888`
```
docker run --rm -it -v ${PWD}/dir:/dir -v ${PWD}/readonly:/root/readonly:ro -p 20000:8888 velkerr/my_jupyter
```
7. Entrypoints – команди, кіт. виконуються при запуску (після збирання) контейнера.

8. Підсумковий Dockerfile
```
FROM ubuntu:18.04

RUN apt update && \
    echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections && \
    apt install git mc python3-dev python3-pip -y && \
    pip3 install jupyter
ADD my_file.txt /root/

ENV usual "noroot"
RUN useradd -m -d /home/${usual} ${usual} && \
    chown -R ${usual} /home/${usual} && \
    chmod -R 755 /home/${usual} && \
    chsh -s /bin/bash ${usual}
ADD entrypoint.sh /home/${usual}
RUN chown ${usual} /home/${usual}/entrypoint.sh && chmod +x /home/${usual}/entrypoint.sh

USER ${usual}
WORKDIR /home/${usual}
RUN echo ${USER} && whoami && pwd

ENTRYPOINT ["/home/noroot/entrypoint.sh"]
EXPOSE 10000

```

Файл `entrypoint.sh`:

```bash
#! /usr/bin/env bash

jupyter notebook --ip 0.0.0.0 --port 10000 --no-browser
```
