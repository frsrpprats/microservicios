# PostgreSQL
## Estructura para la carpeta PostgreSQL
```
carpeta_del_usuario
├── postgresql
│   └── data # Carpeta donde se crearan las bases de datos
│   └── sql  # Carpeta para importar archivos con extension sql la primera vez que se ejecuta
│   └── docker-compose.yml
│   └── .env # Renombrar .env-example y cambiar los valores de las variables de entorno
```
## Ejecución de PostgreSQL

desde la terminal en la carpeta (postgresql) ejecutar:
```
docker compose up
```

## Ejecución de scripts
Documentación: https://hub.docker.com/_/postgres en la sección **Initialization scripts**

