# ==== Stage 1: Build React ====
FROM node:18 as build

WORKDIR /app
COPY frontend/package*.json ./
RUN npm install
COPY frontend/ ./
RUN npm run build   # результат лежит в /app/build

# ==== Stage 2: Runtime Nginx ====
FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html

# Если нужен кастомный nginx.conf для SPA (чтобы отдавать index.html на все роуты)
COPY frontend/default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]