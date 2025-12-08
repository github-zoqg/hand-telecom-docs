# distributed-nginx 源码: https://gitee.com/hzerojs/distributed-nginx
FROM registry.hand-china.com/hzero-fe-public/distributed-nginx:0.0.1

RUN rm /etc/nginx/conf.d/default.conf
COPY cors.conf /etc/nginx/conf.d/

COPY dist /usr/share/nginx/html


