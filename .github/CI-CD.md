# CI/CD del sistema completo

## Configuración del repositorio raíz

En Settings → Secrets and variables → Actions, crear los secrets
`DOCKERHUB_USERNAME` y `DOCKERHUB_TOKEN` (token con escritura en los tres destinos).
Crear estas variables con los nombres completos de repositorios en Docker Hub:

| Variable | Ejemplo |
| --- | --- |
| `DOCKERHUB_BACKEND_IMAGE` | `tuusuario/elohim-backend` |
| `DOCKERHUB_FRONTEND_IMAGE` | `tuusuario/elohim-frontend` |
| `DOCKERHUB_DOCS_IMAGE` | `tuusuario/elohim-docs` |

Crear los repositorios de destino antes de publicar.

## CI

Clona recursivamente los commits de submódulos registrados en este repositorio.
Ejecuta la suite .NET con cobertura, lint/tipos/Jest de frontend y tipos/build de
docs. Luego construye y levanta los cinco servicios, incluyendo Nginx con TLS
autofirmado temporal. Comprueba respuestas HTTP y que no existan reinicios.
Docs se verifica desde su imagen sin montar el código fuente del host.
Siempre intenta mostrar logs y eliminar contenedores y volúmenes temporales.

Los cambios de un submódulo no disparan automáticamente este CI: se debe actualizar
su referencia en el repositorio raíz. Publicar primero los cambios de cada
submódulo y después registrar aquí sus nuevos commits.

## CD

Solo se ejecuta tras un `Stack CI` exitoso de un push a `main` de este repositorio.
Descarga las tres imágenes de esa ejecución exacta y las publica sin reconstruir.
Todas llevan `stack-<commit completo del repositorio raíz>`, para distinguirlas
de las etiquetas `sha-<commit>` de los CD individuales. No publica PostgreSQL
ni Nginx, que ya usan imágenes oficiales. No modifica `latest` ni despliega.

Las publicaciones se hacen secuencialmente y no son atómicas: si falla un push,
el workflow falla y puede haber imágenes ya publicadas. Volver a ejecutar CD
permite completar la publicación mientras el artefacto siga disponible (tres días).
Después se requiere volver a ejecutar el CI del commit.

Configurar `validate` como check obligatorio para proteger `main`. Los repositorios
privados de submódulos requieren una credencial de checkout con acceso a ellos.
Los valores de `.env.example` son de CI; las imágenes frontend incluyen los valores
públicos usados durante su construcción. Para una publicación destinada a otro
entorno se deben configurar esos valores antes de construir y validar la imagen.
