# Gestor de incidencias con Docker Compose

Despliegue de una aplicación web con tres servicios (frontend Nginx, backend API y PostgreSQL), dos redes (`frontend` y `datos`) y un volumen (`postgres_data`).

## Cómo arrancarlo

```bash
cp .env.sample .env      # y rellena los valores
docker compose up -d
```

Abrir http://localhost:8080 (el puerto se cambia con `PUERTO_WEB` en `.env`).

## Preguntas

**¿Por qué el frontend no necesita conectarse a PostgreSQL?**
Porque el frontend solo sirve la página y reenvía las peticiones `/api/` al backend. Quien habla con la base de datos es el backend. Así el frontend no necesita acceso a los datos.

**¿Por qué el backend pertenece a dos redes?**
Porque es el intermediario: recibe peticiones del frontend (red `frontend`) y consulta a PostgreSQL (red `datos`). Es el único que necesita ver a los dos.

**¿Qué servicio necesita publicar un puerto en el anfitrión?**
Solo el frontend (`127.0.0.1:8080`), porque es lo que usa el usuario desde el navegador. Backend y base de datos no se publican.

**¿Por qué publicar PostgreSQL hacia fuera no es la solución adecuada para un fallo de comunicación interno?**
Porque el problema es interno (redes, nombres, orden de arranque) y abrir el puerto no lo arregla, solo expone la base de datos a otras máquinas o programas y aumenta el riesgo de seguridad. Lo correcto es revisar redes y logs.

**¿Qué ventaja ofrece Docker Compose respecto a ejecutar los tres contenedores manualmente?**
Todo queda descrito en un solo archivo: servicios, redes, volúmenes, variables y orden de arranque. Se levanta o se borra todo con un comando, es repetible y se puede versionar, en vez de teclear muchos `docker run`.

**¿Por qué hemos dividido la arquitectura en dos redes?**
Para aislar. Cada servicio solo ve a quien necesita ver: el frontend no puede llegar a PostgreSQL. Si alguien compromete el frontend, la base de datos sigue sin ser accesible directamente.

**¿Por qué el backend utiliza `db` como dirección de PostgreSQL?**
Porque `db` es el nombre del servicio y Docker lo resuelve por DNS interno a la IP del contenedor. Las IP cambian al recrear contenedores; el nombre no. `localhost` sería el propio contenedor del backend, donde no hay base de datos.

**¿Qué diferencia existe entre detener un proyecto y eliminar sus volúmenes?**
`docker compose stop` o `down` paran o eliminan contenedores, pero el volumen sigue y los datos se conservan. `docker compose down -v` borra también el volumen y se pierden todas las incidencias.

**¿Qué consecuencias tendría para la empresa perder el volumen de PostgreSQL?**
Se perderían todas las incidencias registradas: historial, estados y seguimiento. Habría que reconstruirlas a mano o trabajar sin historial, con pérdida de tiempo y de confianza. Por eso hacen falta copias de seguridad.
