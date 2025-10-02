# Stage 1: Build React App
FROM node:18-alpine AS build

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

ARG REACT_APP_API
ENV REACT_APP_API=$REACT_APP_API

RUN npm run build

# Stage 2: Serve with NGINX
FROM nginx:alpine

COPY default.conf /etc/nginx/conf.d/default.conf

COPY --from=build /app/build /usr/share/nginx/html

RUN chmod -R 755 /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]