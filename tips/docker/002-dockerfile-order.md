# Docker: Optimiza el orden de las instrucciones en tu Dockerfile

## Problema
Cuando las instrucciones en un Dockerfile no están ordenadas eficientemente, cada cambio en el código fuente puede invalidar la caché de capas innecesariamente, resultando en builds más lentos.

## Solución
Organizar las instrucciones de menos a más cambiantes para aprovechar el sistema de caché por capas de Docker.

### ❌ Dockerfile NO optimizado

```Dockerfile
FROM node:14
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```

### ✅ Dockerfile optimizado

```Dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

## Explicación

1. Docker construye imágenes capa por capa
2. Cada instrucción crea una nueva capa
3. Las capas se cachean y se reutilizan si no hay cambios
4. Si una capa cambia, todas las capas siguientes deben reconstruirse

### Orden recomendado de instrucciones:

1. `FROM`: Define la imagen base
2. `ARG`: Variables solo para el build
3. `ENV`: Variables de entorno
4. `WORKDIR`: Directorio de trabajo
5. `COPY/ADD`: Archivos que cambian poco (package.json, requirements.txt)
6. `RUN`: Comandos de instalación de dependencias
7. `COPY/ADD`: Resto del código fuente
8. `CMD/ENTRYPOINT`: Comando de inicio

## Beneficios

- ⚡ Builds más rápidos
- 💾 Mejor uso de la caché
- 🔄 Menos reconstrucciones innecesarias
- ⌛ Ahorro de tiempo en el proceso de desarrollo

## Ejemplo práctico

Supongamos que modificas un archivo de código fuente:

### Con Dockerfile NO optimizado:
- Cambias `src/app.js`
- `COPY . .` invalida la caché
- `npm install` se ejecuta de nuevo 😫
- Build tarda varios minutos

### Con Dockerfile optimizado:
- Cambias `src/app.js`
- Solo se invalida la caché después de `COPY . .`
- Las dependencias ya instaladas se mantienen en caché ✨
- Build tarda segundos

## Tips adicionales

1. Usa `.dockerignore` para excluir archivos innecesarios
2. Combina comandos RUN relacionados usando `&&`
3. Limpia la caché de package managers en la misma capa donde se usan
4. Considera usar multi-stage builds para casos más complejos

## Referencias

- [Documentación oficial de Docker sobre mejores prácticas](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Guía de optimización de Docker builds](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache)
- [Docker Layer Caching](https://docs.docker.com/build/cache/)
- [Optimizing Docker Images](https://docs.docker.com/develop/develop-images/guidelines/)