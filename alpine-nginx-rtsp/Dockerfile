##### HOW TO BUILD
## docker build -t alpine-nginx-rtsp --no-cache .

#### HOW TO RUN
## docker run -ti -v /mnt/c/:/mnt/guest --name alpine-ffmpeg-01 alpine-ffmpeg

FROM alpine:latest
LABEL maintainer="Artur Nebot"

RUN apk add alpine-sdk pcre-dev libressl-dev zlib-dev 
RUN apk add vim curl bash

RUN mkdir -p /tmp/build/nginx
WORKDIR /tmp/build/nginx

RUN git clone https://github.com/arut/nginx-rtmp-module.git
RUN git clone https://github.com/nginx/nginx.git

WORKDIR /tmp/build/nginx/nginx

RUN  ./auto/configure --add-module=../nginx-rtmp-module
RUN make
RUN make install

# Embedded nginx config file for HASH+HLS
RUN printf "\
worker_processes  1;                       \n\
events {                                   \n\
    worker_connections  1024;              \n\
}                                          \n\
rtmp {                                     \n\
    server {                               \n\
        listen 1935;                       \n\
        application live {                 \n\
            live on;                       \n\
            interleave on;                 \n\
            hls on;                        \n\
            hls_path /tmp/hls;             \n\
            hls_fragment 15s;              \n\
                                           \n\
            dash on;                       \n\
            dash_path /tmp/dash;           \n\
            dash_fragment 15s;             \n\
                                           \n\
        }                                  \n\
    }                                      \n\
}                                          \n\
http {                                     \n\
    default_type application/octet-stream; \n\
    server {                               \n\
        listen 80;                         \n\
        location /hls {                    \n\
            root /tmp;                     \n\
        }                                  \n\
        location /dash {                   \n\
            root /tmp;                     \n\
        }                                  \n\
    }                                      \n\
    types {                                \n\
        application/vnd.apple.mpegurl m3u8;\n\
        video/mp2t ts;                     \n\
        text/html html;                    \n\
    }                                      \n\
}                                          \n\
"> /usr/local/nginx/conf/nginx.conf

WORKDIR /usr/local/nginx
EXPOSE 80
CMD ["/usr/local/nginx/sbin/./nginx","-g daemon off;"]
