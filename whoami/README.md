# Ejemplo para comprobar Balanceo de Carga
Ejemplo para verificar si **traefik** está realizando el balanceo de carga.
## Ejecución de whoami
```
docker compose up
```
## Extra
Para cambiar la cantidad de instancias (contenedores) de una imagen modificar en **docker-compose.yml**: 
```
   deploy:
      replicas: 5
```