FROM alpine:3.16 AS builder
LABEL maintainer="tindy.it@gmail.com"
ARG THREADS="4"
ARG SHA=""

# build minimized
WORKDIR /
RUN set -xe && \
    apk add --no-cache --virtual .build-tools git g++ build-base linux-headers cmake python3 && \
    apk add --no-cache --virtual .build-deps curl-dev rapidjson-dev pcre2-dev yaml-cpp-dev && \
    git clone --no-checkout https://github.com/ftk/quickjspp.git && \
    cd quickjspp && \
    git fetch origin 0c00c48895919fc02da3f191a2da06addeb07f09 && \
    git checkout 0c00c48895919fc02da3f191a2da06addeb07f09 && \
    git submodule update --init && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make quickjs -j $THREADS && \
    install -d /usr/lib/quickjs/ && \
    install -m644 quickjs/libquickjs.a /usr/lib/quickjs/ && \
    install -d /usr/include/quickjs/ && \
    install -m644 quickjs/quickjs.h quickjs/quickjs-libc.h /usr/include/quickjs/ && \
    install -m644 quickjspp.hpp /usr/include && \
    cd .. && \
    git clone https://github.com/PerMalmberg/libcron --depth=1 && \
    cd libcron && \
    git submodule update --init && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make libcron -j $THREADS && \
    install -m644 libcron/out/Release/liblibcron.a /usr/lib/ && \
    install -d /usr/include/libcron/ && \
    install -m644 libcron/include/libcron/* /usr/include/libcron/ && \
    install -d /usr/include/date/ && \
    install -m644 libcron/externals/date/include/date/* /usr/include/date/ && \
    cd .. && \
    git clone https://github.com/ToruNiina/toml11 --branch="v4.3.0" --depth=1 && \
    cd toml11 && \
    cmake -DCMAKE_CXX_STANDARD=11 . && \
    make install -j $THREADS && \
    cd .. && \
    git clone https://github.com/asdlokj1qpi233/subconverter --depth=1 && \
    cd subconverter && \
    [ -n "$SHA" ] && sed -i 's/\(v[0-9]\.[0-9]\.[0-9]\)/\1-'"$SHA"'/' src/version.h;\
    python3 -m ensurepip && \
    python3 -m pip install gitpython && \
    python3 scripts/update_rules.py -c scripts/rules_config.conf && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make -j $THREADS

# build final image
FROM alpine:3.16
LABEL maintainer="tindy.it@gmail.com"

RUN apk add --no-cache --virtual subconverter-deps pcre2 libcurl yaml-cpp

COPY --from=builder /subconverter/subconverter /usr/bin/
COPY --from=builder /subconverter/base /base/

ENV TZ=Africa/Abidjan
RUN ln -sf /usr/share/zoneinfo/$TZ /etc/localtime
RUN echo $TZ > /etc/timezone

# set entry
WORKDIR /base
CMD subconverter

EXPOSE 25500/tcp
