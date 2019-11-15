FROM nginx

COPY index.html /usr/share/nginx/html/index.html
COPY default.conf /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/nginx.conf

RUN mkdir -p /tmp/run && \
  chown -R www-data:www-data /tmp/run && \
  chown -R www-data:www-data /var/cache/nginx && \
  mkdir -p /tmp/log/nginx && \
  touch /tmp/log/nginx/error.log
  chown -R www-data:www-data /tmp/log/nginx
