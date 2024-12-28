## Deploy commands
docker stack deploy -c docker-compose.yml traefik
docker stack rm traefik

docker stack ls