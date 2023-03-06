# vim: ft=dockerfile
FROM lisaac/luci:latest as frp

RUN wget -c https://github.com/fatedier/frp/releases/download/v0.47.0/frp_0.47.0_linux_amd64.tar.gz -O - | tar -xz --strip-components=1 -C /tmp/

FROM lisaac/luci:latest as ssd
COPY ./rootfs/ /

COPY ./tools/ssd /code
WORKDIR /code
RUN apk update && \
    apk add gcc g++ libc-dev && \
    gcc start-stop-daemon.c -o start-stop-daemon && \
    chmod 750 start-stop-daemon

FROM lisaac/luci:latest as luci

COPY ./rootfs/ /

RUN apk update && \
    apk add git
RUN git clone --depth=1 https://github.com/coolsnowwolf/luci.git /tmp/luci
RUN sed -i '/luci.model.ipkg/d' /tmp/luci/applications/luci-app-frpc/luasrc/model/cbi/frp/basic.lua

FROM lisaac/luci:latest

COPY ./rootfs/ /

RUN apk update && \
    apk add neovim opkg bash dpkg busybox-extras

COPY --from=ssd /code/start-stop-daemon /usr/bin/start-stop-daemon
COPY --from=frp /tmp/frpc /usr/bin/frpc
COPY --from=luci /tmp/luci/applications/luci-app-frpc/ $INTERNAL_PLUGIN_DIR/luci-app-frpc
