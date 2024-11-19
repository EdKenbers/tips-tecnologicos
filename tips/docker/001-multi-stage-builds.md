# Docker Multi-stage Builds

## Problema
Las imágenes Docker pueden volverse innecesariamente grandes cuando incluyen herramientas y archivos de compilación que solo se necesitan durante el proceso de construcción, no en tiempo de ejecución.

## Solución
Usar Multi-stage builds para crear imágenes más pequeñas y eficientes, separando el proceso de construcción del entorno de ejecución.

### ❌ Dockerfile NO optimizado

```dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
CMD ["npm", "start"]
```

### ✅ Dockerfile optimizado

```dockerfile
# Etapa de compilación
FROM node:14 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Etapa de producción
FROM node:14-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY package*.json ./
RUN npm install --only=production
CMD ["npm", "start"]
```


## Explicación

Multi-stage builds permite usar múltiples etapas en un solo Dockerfile:

1. Primera etapa (build): incluye todas las herramientas necesarias para compilar la aplicación
2. Segunda etapa (producción): solo copia los archivos necesarios para ejecutar la aplicación

### Beneficios

- 📦 Tamaño de imagen reducido
- ⚡ Despliegues más rápidos
- 🛡️ Menor superficie de ataque
- 🚀 Proceso de build simplificado

## Ejemplo práctico

En este ejemplo, usamos Node.js para:
1. Compilar una aplicación en la primera etapa
2. Crear una imagen final ligera solo con los archivos necesarios
3. Reducir significativamente el tamaño de la imagen final

## Tips adicionales

1. Nombra tus etapas para mejor legibilidad
2. Usa `COPY --from=build` para copiar archivos específicos
3. Considera usar imágenes base Alpine para la etapa final
4. Mantén las dependencias de producción separadas de las de desarrollo

## Referencias

- [Docker Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)
- [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker optimization guide](https://docs.docker.com/develop/develop-images/guidelines/)