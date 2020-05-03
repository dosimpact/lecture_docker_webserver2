# lecture_docker_webserver2

# 1. Dockerfile

```js
# ----------------------------------node react setting
FROM node:13.12.0-alpine as builder

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
ENV PATH /usr/src/app/node_modules/.bin:$PATH
COPY package.json /usr/src/app/package.json
RUN npm install --silent
RUN npm install react-scripts@3.4.1 -g

COPY . /usr/src/app
RUN npm run build
# ----------------------------------nginx setting

FROM nginx:1.13.9-alpine
# nginx의 기본 설정을 삭제하고 앱에서 설정한 파일을 복사
RUN rm -rf /etc/nginx/conf.d
COPY conf /etc/nginx

# 위에서 생성한 앱의 빌드산출물을 nginx의 샘플 앱이 사용하던 폴더로 이동
COPY --from=builder /usr/src/app/build /usr/share/nginx/html

# 80포트 오픈하고 nginx 실행
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

# 2. .dockerignore

```js
server {
  listen 80;
  location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
    try_files $uri $uri/ /index.html;
  }
  error_page   500 502 503 504  /50x.html;
  location = /50x.html {
    root   /usr/share/nginx/html;
  }
}
```

# 3. 파일생성 ./conf/conf.d/default.conf

```js
server {
  listen 80;
  location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
    try_files $uri $uri/ /index.html;
  }
  error_page   500 502 503 504  /50x.html;
  location = /50x.html {
    root   /usr/share/nginx/html;
  }
}
```

## cf)

```js
sites-available :
가상 서버 환경들에 대한 설정 파일들이 위치하는 부분입니다.
가상 서버를 사용하거나 사용하지 않던간에 그에 대한 설정 파일들이 위치하는 곳이다.

sites-enabled :
sites-available 에 있는 가상 서버 파일들중에서 실행시키고 싶은 파일을 symlink로 연결한 폴더입니다.
실제로 이 폴더에 위치한 가상서버 환경 파일들을 읽어서 서버를 세팅합니다.


nginx.conf :
Nginx에 관한 설정파일로 Nginx 설정에 관한 블록들이 작성되어 있으며
이 파일에서 sites-enabled 폴더에 있는 파일들을 가져옵니다.
```

# 4. scripts

```
docker build -t webserver_2:0.0.1 .
docker run -it -p 3001:80 --name webserver_2 webserver_2:0.0.1
```
