# configserver-ms

Componente de [LogiTrack Atom](https://github.com/LogiTrack-Atom) para el Sprint 02.

## Responsabilidad

Archivos de configuración de shipment-service, courier-service y security-service por ambiente.

Responsable principal: [@DiazGab](https://github.com/DiazGab).

## Estado

Repositorio preparado para iniciar el Sprint 02. La implementación y sus verificaciones se realizarán mediante las issues correspondientes.

Este repositorio contiene configuración; no es una aplicación Java ni requiere una imagen Docker propia.

## Flujo de trabajo

- Crear las issues de este componente en este repositorio y agregarlas al Project común **LogiTrack Atom** de la organización.
- Trabajar en ramas por tarea y abrir pull requests hacia `develop`.
- Integrar y preparar la entrega desde `develop`.
- Enlazar las tareas relacionadas cuando haya dependencias entre repositorios.

El Docker Compose general y la documentación de integración se mantienen en [shipment-service](https://github.com/LogiTrack-Atom/shipment-service).
# configserver-ms

Repositorio de propiedades que lee `configserver` (Spring Cloud Config Server).
Contiene solo archivos de configuración, sin código. No incluye credenciales,
claves privadas ni tokens.

## Archivos

| Aplicación       | Base                  | dev                       | prod                       |
|------------------|-----------------------|---------------------------|----------------------------|
| shipment-service | shipment-service.yml  | shipment-service-dev.yml  | shipment-service-prod.yml  |
| courier-service  | courier-service.yml   | courier-service-dev.yml   | courier-service-prod.yml   |
| security-service | security-service.yml  | security-service-dev.yml  | security-service-prod.yml  |

El nombre del archivo coincide con `spring.application.name` del servicio.

## Precedencia

Para la aplicación `{app}` y el perfil `{perfil}`, el archivo `{app}-{perfil}.yml`
tiene prioridad sobre `{app}.yml`. El archivo base define lo común (nombre, puerto,
base de datos, URLs, seguridad no sensible) y el de perfil define las diferencias
por ambiente (logging, `show-sql` y el marcador).

## Diferencias por ambiente

| Propiedad                    | dev     | prod |
|------------------------------|---------|------|
| logging.level.root           | INFO    | WARN |
| logging.level.com.logitrack_atom | DEBUG | INFO |
| spring.jpa.show-sql          | true    | false |
| logitrack.config.marker      | `{app}-dev` | `{app}-prod` |

El marcador permite comprobar qué configuración recibió cada servicio.

## Variables requeridas

Los placeholders se resuelven en cada servicio con sus variables de entorno.
Config Server no recibe ni publica los valores.

| Variable                 | Servicios                         | Valor por defecto           |
|--------------------------|-----------------------------------|-----------------------------|
| DB_URL                   | shipment, courier, security       | Ninguno                     |
| DB_USERNAME              | shipment, courier, security       | Ninguno                     |
| DB_PASSWORD              | shipment, courier, security       | Ninguno                     |
| JWT_PUBLIC_KEY_LOCATION  | shipment, courier                 | Ninguno                     |
| JWT_PRIVATE_KEY_LOCATION | security                          | Ninguno                     |
| COURIER_SERVICE_URL      | shipment                          | http://localhost:8081       |
| SECURITY_SERVICE_URL     | shipment, courier                 | http://localhost:8082       |

Los valores por defecto con `localhost` son solo para ejecución local. Entre
contenedores deben definirse con el nombre del servicio en Docker Compose.

## Label

- `develop` es la etiqueta de trabajo (`CONFIG_LABEL`, por defecto `develop`).
- Para la entrega se fijará una etiqueta Git explícita, que se registrará aquí.
