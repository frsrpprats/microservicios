# Redis
## Estructura para la carpeta Redis
```
carpeta_del_usuario
├── redis
│   └── data # Carpeta con la configuración de Redis
│   └── docker-compose.yml
│   └── .env # Renombrar .env-example y cambiar los valores de las variables de entorno
```
## Ejecución de Redis

desde la terminal en la carpeta (redis) ejecutar:
```
docker compose up
```

## Documentación
- **maxmemory**: [100mb](https://redis.io/docs/latest/develop/reference/eviction/)
- **maxmemory-policy**:  [volatile-lfu](https://redis.io/docs/latest/develop/reference/eviction/#lfu-eviction)


