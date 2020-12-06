FROM alpine:3.12
#RUN addgroup -g 1001 -S nginx && \
#    adduser -S -D -H -u 1001  -G nginx -g nginx nginx && \
RUN adduser -D -u 1001 -g 'nginx' nginx && \
    apk update && apk add nginx && \
    mkdir /app && \
    mkdir /run/nginx && \
    mkdir /var/tmp/nginx && \
    chown -R nginx:nginx /run/nginx && \
    chown -R nginx:nginx /var/lib/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    chown -R nginx:nginx /var/tmp/nginx && \
    chown -R nginx:nginx /app
ADD nginx.conf /etc/nginx/nginx.conf
EXPOSE 8000
USER 1001
ENTRYPOINT [ "nginx" ]
CMD [ "-c", "/etc/nginx/nginx.conf"]
