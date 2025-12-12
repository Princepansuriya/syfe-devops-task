FROM ubuntu:20.04

# Avoid prompts
ENV DEBIAN_FRONTEND=noninteractive

# 1. Install Dependencies (Fixed: added libpq-dev)
RUN apt-get update && apt-get install -y \
    build-essential \
    libpcre3-dev \
    libssl-dev \
    zlib1g-dev \
    libpq-dev \
    wget \
    perl \
    make \
    && rm -rf /var/lib/apt/lists/*

# 2. Build OpenResty with required flags
ENV OPENRESTY_VERSION=1.21.4.1
RUN wget https://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz \
    && tar -xzvf openresty-${OPENRESTY_VERSION}.tar.gz \
    && cd openresty-${OPENRESTY_VERSION} \
    && ./configure \
       --prefix=/opt/openresty \
       --with-pcre-jit \
       --with-ipv6 \
       --without-http_redis2_module \
       --with-http_iconv_module \
       --with-http_postgres_module \
       -j8 \
    && make -j8 \
    && make install \
    && cd .. \
    && rm -rf openresty-${OPENRESTY_VERSION}*

# 3. Copy Config & Lua Lib
COPY nginx.conf /opt/openresty/nginx/conf/nginx.conf
COPY prometheus.lua /opt/openresty/lualib/prometheus.lua

ENV PATH=$PATH:/opt/openresty/nginx/sbin
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
