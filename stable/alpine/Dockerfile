FROM alpine:3.8

LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"

ARG userid=1001
ARG username=nginx
# Group must be specified as root for OpenShift and Kubernetes
ARG group=root

COPY --from=nginx:1.14.0-alpine /usr/local/bin/envsubst /usr/local/bin/envsubst
COPY --from=nginx:1.14.0-alpine /usr/sbin/nginx /usr/sbin/nginx
COPY --from=nginx:1.14.0-alpine /usr/sbin/nginx-debug /usr/sbin/nginx-debug
COPY --from=nginx:1.14.0-alpine /etc/nginx /etc/nginx
COPY --from=nginx:1.14.0-alpine /usr/lib/nginx/modules/ /usr/lib/nginx/modules/
COPY --from=nginx:1.14.0-alpine /var/log/nginx /var/log/nginx
COPY --from=nginx:1.14.0-alpine /usr/share/nginx/html/ /usr/share/nginx/html/

# Search for and install runtime dependencies:
# scan all required .so files, put them into a format that apk can understand, then install them.
# Also bring in tzdata so users could set the timezones through the environment variables
RUN runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' /usr/sbin/nginx /usr/lib/nginx/modules/*.so /tmp/envsubst \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)" \
    && apk add --no-cache --virtual .nginx-rundeps $runDeps tzdata

# symlink nginx modules,
# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx.vh.default.conf /etc/nginx/conf.d/default.conf

RUN adduser -D -S -h /var/cache/nginx -s /sbin/nologin -u ${userid} ${username} \
    && chown -R ${userid}:${group} /var/cache/nginx/ /var/log/nginx/ usr/share/nginx/html/

EXPOSE 8080

STOPSIGNAL SIGTERM

# Run with userid instead of username for OpenShift/Kubernetes
USER ${userid}

CMD ["nginx", "-g", "daemon off;"]
