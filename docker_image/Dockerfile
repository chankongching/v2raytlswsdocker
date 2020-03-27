FROM centos:7
MAINTAINER chankongching@gmail.com

#====================================================
#	System Request:Linux
#	Author:	chankongching
#	Dscription: V2ray ws+tls Docker
#	Version: 1.0
#	email:jacky.chan@bootdev.com
#	Official document: www.v2ray.com
#====================================================

### Settings
# General settings
ENV GITHUB_BRANCH master
# ENV CAMOUFLAGE 6a105b4b

### Copying files
# Insert supervisord conf file
ADD supervisord.conf /etc/

# V2ray settings
## v2ray version selection is not complete
## ENV V2RAY_VERSION 4.23.1
ENV V2RAY_CONF_DIR /etc/v2ray
ENV V2RAY_CONF ${V2RAY_CONF_DIR}/config.json
ENV V2RAY_BIN_DIR /usr/bin/v2ray
ENV V2RAY_INFO_FILE $HOME/v2ray_info.inf
ENV V2RAY_QR_CONF_FILE /usr/local/vmess_qr.json

# Nginx settings
ENV NGINX_VERSION 1.17.8
ENV NGINX_CONF_DIR /etc/nginx/

# nginx related settings
ENV OPENSSL_VERSION 1.1.1d
ENV JEMALLOC_VERSION 5.2.1

### Start implementation processes
# Install prerequisites
RUN set -x && \
    yum install -y gcc \
    bzip2 \
    cyrus-sasl-devel \
    unzip \
    wget \
    gcc-c++ \
    autoconf \
    automake \
    pcre-devel \
    libtool \
    make \
    cmake \
    zlib-devel

RUN set -x && \
    cd /root && \
    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
    python get-pip.py

# Install supervisord
RUN set -x && \
    pip install supervisor && \
    mkdir -p /var/{log/supervisor,run/{sshd,supervisord}}

# Install v2ray
RUN if [ -z ${camouflage+x} ]; then export camouflage=$CAMOUFLAGE; else export camouflage=$(head -n 10 /dev/urandom | md5sum | head -c 8); fi
RUN set -x && \
    mkdir -p /root/v2ray && \
    wget -N --no-check-certificate https://install.direct/go.sh && \
    bash go.sh --force

# Install openssl
RUN mkdir -p /home/openssl && cd $_ && \
    curl -Lk https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz | gunzip | tar x -C /home/openssl
# Install jemalloc
RUN mkdir -p /home/jemalloc && cd $_ && \
    curl -Lk https://github.com/jemalloc/jemalloc/releases/download/${JEMALLOC_VERSION}/jemalloc-${JEMALLOC_VERSION}.tar.bz2 | tar xj -C /home/jemalloc && \
    cd jemalloc-${JEMALLOC_VERSION} || exit && \
    ./configure && \
    make && make install && \
    echo '/usr/local/lib' >/etc/ld.so.conf.d/local.conf && \
    ldconfig

# Install nginx to handle TLS
RUN mkdir -p /home/nginx && cd $_ && \
    curl -Lk http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz | gunzip | tar x -C /home/nginx

# Make install nginx
RUN set -x && \
    cd /home/nginx/nginx-$NGINX_VERSION && \
    ./configure --prefix=$NGINX_CONF_DIR \
    --user=root --group=root \
    --error-log-path=/dev/stdout \
    --http-log-path=/dev/stdout \
    --pid-path=/var/run/nginx.pid \
    --with-pcre \
    --with-http_ssl_module \
    --with-http_gzip_static_module \
    --with-http_stub_status_module \
    --with-http_realip_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_secure_link_module \
    --with-http_v2_module \
    --with-cc-opt='-O3' \
    --with-ld-opt="-ljemalloc" \
    --with-openssl=/home/openssl/openssl-"$OPENSSL_VERSION" && \
    make && make install

# Configuring nginx
RUN sed -i 's/#user  nobody;/user  root;/' ${NGINX_CONF_DIR}/conf/nginx.conf
RUN sed -i 's/worker_processes  1;/worker_processes  3;/' ${NGINX_CONF_DIR}/conf/nginx.conf
RUN sed -i 's/    worker_connections  1024;/    worker_connections  4096;/' ${NGINX_CONF_DIR}/conf/nginx.conf
RUN sed -i '$i include ../conf.d/*.conf;' ${NGINX_CONF_DIR}/conf/nginx.conf
RUN sed -i '1 a error_log  /dev/stdout;' ${NGINX_CONF_DIR}/conf/nginx.conf

ADD startup.sh /root/startup.sh
RUN chmod +x /root/startup.sh

# Starting the application set
ENTRYPOINT ["/root/startup.sh"]
