FROM node:16.17 as build-deps
WORKDIR /app/
COPY . .
COPY package.json yarn.lock nginx.conf.template ./
RUN yarn install \
    && yarn build

FROM nginx:1.23.1-alpine
COPY --from=build-deps /app/dist /usr/share/nginx/html
COPY --from=build-deps /app/nginx.conf.template /etc/nginx/conf.d/default.conf
COPY . .
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

