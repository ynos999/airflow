## Secība

**1. Jāizveido tīkls swarm režīmā, lai tam var piekļūt dažādi stacks**

```
docker network create --driver overlay --scope swarm airflow
```

**2. Palaiž postgres, redis un private registry**
```
docker stack deploy -c .\01-airflow-core-services.yaml airflow-core-services
```

**3. Build airflow**

```
docker build -t fid-airflow:2.9.0-001 ./airflow
```

```
docker tag fid-airflow:2.9.0-001 127.0.0.1:5000/fid-airflow:2.9.0-001
```

vai:

```
docker tag fid-airflow:2.9.0-001 registry:5000/fid-airflow:2.9.0-001
```

```
docker push 127.0.0.1:5000/ringolds-airflow:2.9.0-001
```
vai:

```
docker push registry:5000/ringolds-airflow:2.9.0-001
```

## Ja lieto registry:5000, tad hosts failā ir jāveic izmaiņas:
```
sudo nano /etc/hosts
```
```
127.0.0.1 registry
```
## Ja izmanto registry, tad pievieno 02 un 03 failā image name...

```
Kad tiks veiktas izmaiņas, tad ir atkārtoti jāveic visas operācijas un ir jānomaina
tagi, lai nebūtu konfliktu ar citām versijām.

Kā arī, kad tiek nomainīts tags, tad tas ir jānomaina visos docker compose failos,
kur šis tags ir izmantots.

Svarīgi! Ja tiek izmantots localhost, tad privātais reģistrs nestrādās, jo būs
problēmas ar DNS, tāpēc ir jāizmanto 127.0.0.1 vai jārediģē hosts fails.

Pievērs uzmanību otrajai komandai, kur tiek norādīts registry:5000. Tas ir iekšējais
reģistrs, kas ir izveidots ar stack `airflow-base-services`, bet pushots tiek uz
127.0.0.1.

Ja vēlētos veikt push no āras, tad ir divi veidi, kā to var izdarīt:

1. Pieslēdzies ar ssh uz serveri, kur ir pieeja privātajam reģistram un veic push no
   turienes (visdrošāk un visvieglāk) vai
2. Uzstādi kādu reverse proxy, kas ļaus piekļūt privātajam reģistram no ārpasaules ar
   noteiktu IP adresi, autorizāciju, utt.

**4. Inicializē airflow**

```
docker stack deploy -c .\02-airflow-init.yaml airflow-init

```

Kad airflow-init ir pabeidzis darbu (nosaka ar `docker stack ps airflow-init`), tad ir jāizdzēš stack `airflow-init` un jāpalaiž stack `airflow-core`

```
docker stack ps airflow-init
```

```
docker stack rm airflow-init
```

**5. Palaiž pašu airflow**

```
docker stack deploy -c .\03-airflow-core.yaml airflow-core

```

## Noderīgas komandas:

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
docker logs <docker name>
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