# Guía de configuración de corteza con zrok para tener acceso remoto al sistema

## Archivos empleados

| Archivo | Descripción |
| --- | --- |
| .env | Contiene variables de entorno empleadas en el sistema |
| docker-compose.yaml | Archivo de docker compose que contiene la configuración de las máquinas virtuales a emplear. |

## Configuración de Corteza

Debe tener Docker instalado y ejecutándose. Docker Compose te facilita la vida a la hora de ejecutar varias imágenes de Docker, cada una de las cuales se puede configurar de forma arbitraria. Puedes consultar la [documentación oficial](https://docs.docker.com/compose/install/) para configurarlo.

## Variables de entorno

Las variables de entorno se definen en el archivo `.env`. Se pueden consultar la configuración actual en el archivo [.env](.env).

## Configuración de docker compose

El archivo de docker compose se puede consultar en el [repositorio de github](https://github.com/iver/corteza), el archivo contiene la definición de dos instancias: el servidor de corteza (`server`) y el servidor de base de datos (`db`). Se puede tener acceso a ambos desde la máquina host gracias a la configuración definida en la sección de red.

Los servicios están expuestos en sus respectivos puertos:

- Puerto 80 para el servidor de corteza
- Puerto 5432 para el servidor de base de datos

> **Nota**: Se omite la configuración de nginx para facilitar su publicación. En caso de que se requiera configurar nginx se debe definir otro proyecto.
>

## Configuración de zrok

La información de configuración de `zrok` se puede encontrar en el [sitio oficial](https://docs.zrok.io/docs/getting-started/), sin embargo se hace un pequeño resumen para facilitar su uso.

### 1. Invitación

Para poder registrarse en zrok, se debe bajar primero el software y después ejecutar el comando `zrok invite`

```bash
$ zrok invite
enter and confirm your email address...

> user@domain.com
> user@domain.com

[ Submit ]

invitation sent to 'user@domain.com'!
```

### 2. Obtener el hash

Por seguridad, se requiere de un hash de autenticación con el servicio de `zrok` en la nube. Para conocer el hash que nos permite habilitar `zrok`, es necesario ingresar en el navegador web en la dirección: <https://api.zrok.io/>

Nos debe mostrar algo como: `zrok enable klFEoIi0QAg7`

### 3. Habilitar el entorno

La instrucción del paso anterior, se debe ejecutar en la terminal y debe mostrar algo como:

```bash
$ zrok enable klFEoIi0QAg7
  ⣻  the zrok environment was successfully enabled...
```

Si ejecutamos `zrok status` podemos ver algo como:

```bash
$ zrok status
Config:

CONFIG           VALUE                SOURCE
apiEndpoint      [https://api.zrok.io](https://api.zrok.io/)  env
defaultFrontend  public               binary

Environment:

PROPERTY       VALUE
Secret Token   <<SET>>
Ziti Identity  <<SET>>
```

### 4. Compartir entornos

Por defecto los entornos son efímeros, lo que significa que cuando se comparten se genera un identificador único para ese entorno y estará disponible mientras el servicio esté funcionando. En caso de falla o en caso de detener el servicio, se generará otro identificador para poder ingresar al enlace compartido. Para saber más se recomienda la lectura de la [documentación oficial](https://docs.zrok.io/docs/getting-started/#ephemeral-by-default).

En este caso se requiere compartir una dirección que sea relativamente estática, para que esto sea posible se deben seguir los siguientes pasos:

### Crear un subdominio reservado

Esto permite utilizar el servicio de `zrok` para definir nuestro propio subdominio, por ejemplo, para poder reservar el nombre `dominiobonito.share.zrok.io`  es necesario ejecutar el comando:

```bash
$ zrok reserve public --unique-name dominiobonito localhost:8080
[   1.996]    INFO main.(*reserveCommand).run: your reserved share token is 'dominiobonito'
[   1.996]    INFO main.(*reserveCommand).run: reserved frontend endpoint: https://dominiobonito.share.zrok.io
```

Con lo anterior se reserva el nombre y se direcciona la petición a [localhost:8080](http://localhost:8080) que es la ruta configurada en los archivos de docker compose. Para poder habilitar el servicio, es necesario ejecutar otro comando más:

```bash
$ zrok share reserved dominiobonito
[   0.370]    INFO main.(*shareReservedCommand).run: sharing target: 'http://localhost:8080'
[   0.370]    INFO main.(*shareReservedCommand).run: using existing backend target: http://localhost:8080
[   0.927]    INFO sdk-golang/ziti.(*listenerManager).createSessionWithBackoff: {session token=[548c5da3-538d-4a21-9d42-66015fb5d534]} new service session
```

Con lo anterior, ahora podemos ingresar al sitio [`dominiobonito.share.zrok.io`](http://dominiobonito.share.zrok.io) en nuestro navegador.
