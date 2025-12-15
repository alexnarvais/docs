Run the docker compose file.
```shell
docker compose -f docker-compose.yml --project-name xander_automation up -d
```

Shows the container with exit status.
```shell
docker ps -a 
```

Stop a single service
```shell
docker-compose stop jenkins
```
Access the docker shell
```shell
docker exec -it jenkins bash
```
