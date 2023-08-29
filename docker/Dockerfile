FROM alpine AS Builder
RUN apk add --no-cache --virtual .build-deps \
        libffi-dev \
        gcc \
        musl-dev \
        libxml2-dev \
        libxslt-dev \
    && apk add --no-cache $(echo $(wget --no-check-certificate -qO- https://raw.githubusercontent.com/vay1314/nas-tools-unlock/master/package_list.txt)) \
    && ln -sf /usr/bin/python3 /usr/bin/python \
    && curl https://rclone.org/install.sh | bash \
    && if [ "$(uname -m)" = "x86_64" ]; then ARCH=amd64; elif [ "$(uname -m)" = "aarch64" ]; then ARCH=arm64; fi \
    && curl https://dl.min.io/client/mc/release/linux-${ARCH}/mc --create-dirs -o /usr/bin/mc \
    && chmod +x /usr/bin/mc \
    && pip install --upgrade pip setuptools wheel \
    && pip install cython \
    && pip install -r https://raw.githubusercontent.com/vay1314/nas-tools-unlock/master/requirements.txt \
    && apk del --purge .build-deps \
    && rm -rf /tmp/* /root/.cache /var/cache/apk/* \
    && set -x && \
    apk add --no-cache nginx php-fpm php-curl && \
    rm -f /etc/nginx/http.d/default.conf && \
    sed -i 's|worker_processes|#&|' /etc/nginx/nginx.conf && \
    mkdir -p /douban && \
    mkdir -p /etc/services.d/nginx && \
    mkdir -p /etc/services.d/php-fpm && \
    cp /etc/services.d/NAStool/notification-fd /etc/services.d/php-fpm/notification-fd && \
    cp /etc/services.d/NAStool/notification-fd /etc/services.d/nginx/notification-fd
COPY --chmod=755 ./rootfs /
COPY ./douban/douban.php /douban/
COPY ./douban/douban.conf /etc/nginx/http.d/
COPY ./douban/nginx_run /etc/services.d/nginx/run
COPY ./douban/nginx_finish /etc/services.d/nginx/finish
COPY ./douban/php_run /etc/services.d/php-fpm/run
COPY ./douban/php_finish /etc/services.d/php-fpm/finish
FROM scratch AS APP
COPY --from=Builder / /
ENV S6_SERVICES_GRACETIME=30000 \
    S6_KILL_GRACETIME=60000 \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0 \
    S6_SYNC_DISKS=1 \
    HOME="/nt" \
    TERM="xterm" \
    PATH=${PATH}:/usr/lib/chromium \
    LANG="C.UTF-8" \
    TZ="Asia/Shanghai" \
    NASTOOL_CONFIG="/config/config.yaml" \
    NASTOOL_AUTO_UPDATE=false \
    NASTOOL_CN_UPDATE=true \
    NASTOOL_VERSION=master \
    PS1="\u@\h:\w \$ " \
    REPO_URL="https://github.com/vay1314/nas-tools-unlock.git" \
    PYPI_MIRROR="https://pypi.tuna.tsinghua.edu.cn/simple" \
    ALPINE_MIRROR="mirrors.ustc.edu.cn" \
    PUID=0 \
    PGID=0 \
    UMASK=000 \
    WORKDIR="/nas-tools"
WORKDIR ${WORKDIR}
RUN mkdir ${HOME} \
    && addgroup -S nt -g 911 \
    && adduser -S nt -G nt -h ${HOME} -s /bin/bash -u 911 \
    && python_ver=$(python3 -V | awk '{print $2}') \
    && echo "${WORKDIR}/" > /usr/lib/python${python_ver%.*}/site-packages/nas-tools.pth \
    && echo 'fs.inotify.max_user_watches=5242880' >> /etc/sysctl.conf \
    && echo 'fs.inotify.max_user_instances=5242880' >> /etc/sysctl.conf \
    && echo "nt ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers \
    && git config --global pull.ff only \
    && git clone -b master ${REPO_URL} ${WORKDIR} --depth=1 --recurse-submodule \
    && git config --global --add safe.directory ${WORKDIR} \
    && chmod +x /nas-tools/docker/entrypoint.sh \
    && chmod +x /etc/services.d/nginx/* && \
    && chmod +x /etc/services.d/php-fpm/*
EXPOSE 3000
VOLUME ["/config"]
ENTRYPOINT [ "/init" ]
