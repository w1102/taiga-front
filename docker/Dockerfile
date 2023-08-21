FROM node:19.7.0 as build

WORKDIR /taiga

COPY . .
RUN npm ci
RUN npx gulp deploy


FROM nginx:1.23-alpine
LABEL maintainer="support@taiga.io"

RUN mkdir -p /taiga/dist
COPY --from=build /taiga /taiga 

COPY docker/default.conf /etc/nginx/conf.d/default.conf
COPY docker/conf.json.template /
COPY docker/config_env_subst.sh /docker-entrypoint.d/30_config_env_subst.sh

RUN set -eux; \
    apk update; \
    apk add --no-cache --virtual .build-deps \
       subversion; \
    apk add \
       bash; \
    mv /conf.json.template /taiga/dist/; \
    chmod +x /docker-entrypoint.d/30_config_env_subst.sh; \
    # Install taiga-front contribs
    mkdir /taiga/dist/plugins; \
    cd /taiga/dist/plugins; \
    svn export "https://github.com/kaleidos-ventures/taiga-contrib-slack/tags/6.7.0/front/dist" "slack"; \
    svn export "https://github.com/kaleidos-ventures/taiga-contrib-github-auth/tags/6.7.0/front/dist" "github-auth"; \
    svn export "https://github.com/kaleidos-ventures/taiga-contrib-gitlab-auth/tags/6.7.0/front/dist" "gitlab-auth"; \
    cd /; \
    # Remove unused dependencies
    apk del --no-cache .build-deps; \
    # Ready for nginx
    mv /taiga/dist/* /usr/share/nginx/html; \
    rm -rf /taiga