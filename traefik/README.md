# Traefik
Traefik es un proxy inverso y balanceador de carga, nativo de la nube y de código abierto que facilita la implementación de microservicios.

![Traefik](https://doc.traefik.io/traefik/assets/img/traefik-architecture.png)
## Estructura de Traefik
```
carpeta_del_usuario
├── traefik
│   └── certs 
│   └── config  # Carpeta de configuración de traefik
│   └── docker-compose.yml
```
# Utilidad para generar Certificados
Para generar e instalar certificados para desarrollo se puede utilizar **mkcert** 
1. Descargar mkcert https://github.com/FiloSottile/mkcert/tags

2. Generar certificados:
```
mkcert -cert-file certs/cert.pem -key-file certs/key.pem "universidad.localhost" "*.universidad.localhost" 127.0.0.1 ::1
```
3. Instalar certificados
```
mkcert -install
```
Los archivos *key.pem* y *cert.pem* deben copiarse en la carpeta **certs** de traefik

## Documentación
- [mkcert](https://github.com/FiloSottile/mkcert)
- [traefik](https://doc.traefik.io/traefik/)

