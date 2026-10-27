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
| JWT_PUBLIC_KEY_LOCATION  | shipment, courier, security       | Ninguno                     |
| JWT_PRIVATE_KEY_LOCATION | security                          | Ninguno                     |
| BOOTSTRAP_ADMIN_ENABLED  | security                          | false                       |
| BOOTSTRAP_ADMIN_EMAIL    | security, si bootstrap está activo | Vacío                       |
| BOOTSTRAP_ADMIN_PASSWORD | security, si bootstrap está activo | Vacío                       |
| COURIER_SERVICE_URL      | shipment                          | http://localhost:8081       |
| SECURITY_SERVICE_URL     | shipment, courier                 | http://localhost:8082       |

Los valores por defecto con `localhost` son solo para ejecución local. Entre
contenedores deben definirse con el nombre del servicio en Docker Compose.

## Label

- `develop` es la etiqueta de trabajo (`CONFIG_LABEL`, por defecto `develop`).
- Para la entrega se fijará una etiqueta Git explícita, que se registrará aquí.

## Contrato de security

Security necesita ambas rutas de claves RSA: firma con la privada y valida los
JWT con la pública. Las dos variables contienen URI `file:` de archivos externos;
las claves no se guardan en este repositorio ni se entregan a Config Server.
La privada se monta exclusivamente en security.

El bootstrap del primer administrador está desactivado por defecto. Si se activa,
el servicio exige email y contraseña válidos, recibidos únicamente por su entorno.
Los valores vacíos no habilitan una cuenta ni constituyen credenciales de respaldo.
El registro público continúa creando exclusivamente usuarios CLIENTE.

La configuración base rechaza propiedades JSON desconocidas y roles numéricos
con Jackson 3, conserva `ddl-auto=validate`, desactiva Open Session in View y limita
Actuator a salud agregada sin detalles ni componentes. Los errores no exponen
mensajes internos, trazas ni errores de binding del framework.

Dev y prod usan el mismo puerto 8082 y tiempos JWT; sus marcadores y logging
siguen diferenciados. El cambio se verifica ejecutando el Config Server de
[configserver](https://github.com/LogiTrack-Atom/configserver) contra los commits
Git de estas propiedades, con la imagen de security, MySQL aislado y secretos
de prueba externos. Los resultados y commits exactos se registran en el PR de
integración; los fixtures internos de security no sustituyen esa comprobación.


## Contrato de courier

Courier valida JWT con la clave pública externa y no recibe material privado.
La configuración base conserva JPA validate, Open Session in View desactivado y
auditoría UTC; rechaza campos JSON desconocidos y estados numéricos; expone solo
salud agregada y omite detalles internos de error.

El cliente administrativo usa SECURITY_SERVICE_URL y timeouts positivos:
SECURITY_CONNECT_TIMEOUT (2s por defecto) y SECURITY_READ_TIMEOUT (3s por defecto).
El puerto es 8081 salvo que el cliente defina COURIER_SERVER_PORT. Las propiedades
de MySQL y JWT_PUBLIC_KEY_LOCATION continúan sin valores de respaldo sensibles.

Los perfiles dev/prod conservan sus marcadores courier-service-dev/prod, logging
y show-sql. La comprobación de integración registra la revisión Git exacta del
servidor y de este repositorio y arranca courier con ambos ambientes, secretos
efímeros y recursos aislados; las pruebas locales no sustituyen ese ensayo.
