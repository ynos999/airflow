# Airflow for docker swarm.
###
## Sequence:
**1. Create a network in swarm mode so that it can be accessed by different stacks**
```
docker network create --driver overlay --scope swarm airflow
```
**2. Run postgres, redis and private registry**
```
docker stack deploy -c .\01-airflow-core-services.yaml airflow-core-services
```
**3. Build airflow**
```
docker build -t fid-airflow:2.9.0-001 ./airflow
```
## Add a tag (change the tag each time):
```
docker tag fid-airflow:2.9.0-001 127.0.0.1:5000/fid-airflow:2.9.0-001
```
## Push to private registry:

```
docker push 127.0.0.1:5000/fid-airflow:2.9.0-001
```
or:
```
docker tag fid-airflow:2.9.0-001 registry:5000/fid-airflow:2.9.0-001
```
```
docker push registry:5000/fid-airflow:2.9.0-001
```
## If you use the registry, you need to make changes to the hosts file:
```
sudo nano /etc/hosts
```
```
127.0.0.1 registry
```
## If using registry, then add 02 and 03 to the image name file...
```
Kad tiks veiktas izmaiņas, tad ir atkārtoti jāveic visas operācijas un ir jānomaina
tagi, lai nebūtu konfliktu ar citām versijām.

Kā arī, kad tiek nomainīts tags, tad tas ir jānomaina visos docker compose failos,
kur šis tags ir izmantots.

Ja tiek izmantots localhost, tad privātais reģistrs nestrādās, jo būs
problēmas ar DNS, tāpēc ir jāizmanto 127.0.0.1 vai jārediģē hosts fails.

Pievērs uzmanību otrajai komandai, kur tiek norādīts registry:5000. Tas ir iekšējais
reģistrs, kas ir izveidots ar stack `airflow-core-services`, bet pushots tiek uz
127.0.0.1.

Ja vēlētos veikt push no āras, tad ir divi veidi, kā to var izdarīt:

1. Pieslēdzies ar ssh uz serveri, kur ir pieeja privātajam reģistram un veic push no
   turienes (visdrošāk un visvieglāk) vai
2. Uzstādi kādu reverse proxy, kas ļaus piekļūt privātajam reģistram no ārpasaules ar
   noteiktu IP adresi, autorizāciju, utt.
```
**4. Initialize airflow**
```
docker stack deploy -c .\02-airflow-init.yaml airflow-init
```
When airflow-init has finished its work (determined by `docker stack ps airflow-init`), you need to delete the `airflow-init` stack and start the `airflow-core` stack.

Kad airflow-init ir pabeidzis darbu (nosaka ar `docker stack ps airflow-init`), tad ir jāizdzēš stack `airflow-init` un jāpalaiž stack `airflow-core`

```
docker stack ps airflow-init
```
```
docker stack rm airflow-init
```
**5. Run your airflow**
```
docker stack deploy -c .\03-airflow-core.yaml airflow-core
```
# Useful commands:
##
## Inicializēt docker swarm:
```
docker swarm init
```
## Redzēt stack logus:
```
docker stack logs [folder_name]-[stack_name]-[service_name] -f
```
## Uzzināt node name, lai var uzlikt, ka docker nemaina lokāciju...
```
docker node ls
```
## Ielogoties kontēnerī:
```
docker exec -it <mycontainer> bash
```
## Izveidot tīklu:
```
Creating network airflow
```
## Docker network saraksts:
```
docker network ls
```
## Dzēst docker network:
```
docker network rm airflow
```
## Docker service logs:
```
docker service logs <service_name>
```
## Docker logs:
```
docker logs <dokor_name>
```
## Apskatīties esošos servisus:
```
docker service ls
```
## Apskatīt konkrētu servisu:
```
docker service ps airflow-core-services_postgres
```
## Visus stack servisus:
```
docker stack services airflow-stack
```
## Apskatīties visus dokerus:
```
docker ps -a
```
## Apskatīties strādājošus dokerus:
```
docker ps
```
## Dzēst visus servisus:
```
docker service rm $(docker service ls -q)
```
## Konkrēta servisa dzēšana:
```
docker service rm airflow-stack_airflow-init
```
## Palaist docker compose fonā:
```
docker-compose up -d --build
```
## Dzēst docker compose:
```
docker-compose down
```
## Docker volumes atrašanās lokācija:
```
/var/lib/docker/volumes
```
## Docker volume saraksts:
```
docker volume ls
```
## Dzēst volume:
```
docker volume rm <volume name>
```
## Docker image saraksts:
```
docker images ls
```
## Dzēst docker image:
```
docker rmi 7fc37b47acde
```
## Ja no serviss neuztaisa docker, var paskatīties sīkāk:
```
sudo docker service ps airflow-stack_airflow-triggerer --no-trunc
```
## Dzēst vairākus servisus:
```
docker service rm airflow-core_airflow-scheduler airflow-core_airflow-triggerer airflow-core_airflow-webserver airflow-core_airflow-worker
```
## Docker info:
```
docker info
```
## Uzlikt Doker dashboard: https://portainer.io
```
curl -L https://downloads.portainer.io/ce2-19/portainer-agent-stack.yml -o portainer-agent-stack.yml
```
```
docker stack deploy -c portainer-agent-stack.yml portainer
```
## Docker secret create:
```
echo "mysecretuser" | docker secret create postgres_user -
echo "mysecretpassword" | docker secret create postgres_passwd -
```
## Secret piemērs docker swarm postgres:
```
version: '3.8'

services:
 db:
    image: postgres:13
    secrets:
      - postgres_user
      - postgres_passwd
    environment:
      POSTGRES_USER_FILE: /run/secrets/postgres_user
      POSTGRES_PASSWORD_FILE: /run/secrets/postgres_passwd
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

secrets:
 postgres_user:
    external: true
 postgres_passwd:
    external: true

volumes:
 postgres_data:
```
## Docker secret atrašanās vieta:
```
cat /run/secrets/postgres_user
cat /run/secrets/postgres_passwd
```
## Apskatīties Docker secret:
```
docker secret ls
```
## Docker secret inspect:
```
docker secret inspect <secret_name>
```
## Ielogoties datubāzē:
```
psql -h localhost -U mysecretuser -d postgres --password
\dt
```
## Ja grib dzēst secret, tad sākumā ir jāizdzēš serviss:
```
docker service rm <service_name>
docker secret remove <secret_name>
```