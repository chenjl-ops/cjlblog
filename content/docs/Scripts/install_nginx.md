+++
title = "Nginx安装"
weight = 4
+++

```bash
# Debian
apt install nginx

# Centos Rock
yum install nginx
```
```bash
# 脚本安装

#!/bin/bash
#====================================================================
# 名称： Nginx 1.30.3 自动化编译安装脚本（含 vhost_traffic_status 模块）
# 描述： 适用于 Rocky Linux 9.6，从源码编译安装 Nginx 并集成 vts 模块
# 用法： 以 root 或 sudo 权限执行：bash install_nginx.sh
# 版本： 2.0
#====================================================================
 
# 设置严格模式：遇到错误即退出，确保管道命令出错即停
set -e
set -o pipefail
 
# ----------------------------- 变量定义 -----------------------------
NGINX_VERSION="1.30.3"
NGINX_USER="nginx"
NGINX_GROUP="nginx"
INSTALL_DIR="/usr/local/nginx"
SOURCE_DIR="/usr/local/src"
NGINX_URL="https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz"
NGINX_TARBALL="nginx-${NGINX_VERSION}.tar.gz"
NGINX_SOURCE="${SOURCE_DIR}/nginx-${NGINX_VERSION}"
 
# VTS 模块变量
VTS_VERSION="0.2.3"
VTS_URL="https://github.com/vozlt/nginx-module-vts/archive/v${VTS_VERSION}.tar.gz"
VTS_TARBALL="v${VTS_VERSION}.tar.gz"
VTS_DIR="${SOURCE_DIR}/nginx-module-vts-${VTS_VERSION}"
 
#https://github.com/vozlt/nginx-module-vts/archive/refs/tags/v0.2.3.tar.gz
 
# 日志文件
LOG_FILE="/var/log/nginx_install.log"
 
# ----------------------------- 函数定义 -----------------------------
 
# 日志函数：同时输出到屏幕和日志文件
log_info() {
    echo "[INFO] $(date '+%Y-%m-%d %H:%M:%S') - $*" | tee -a "${LOG_FILE}"
}
 
log_warn() {
    echo "[WARN] $(date '+%Y-%m-%d %H:%M:%S') - $*" | tee -a "${LOG_FILE}" >&2
}
 
log_error() {
    echo "[ERROR] $(date '+%Y-%m-%d %H:%M:%S') - $*" | tee -a "${LOG_FILE}" >&2
    exit 1
}
 
# 检查命令是否存在
check_command() {
    if ! command -v "$1" &> /dev/null; then
        log_error "命令 '$1' 未找到，请确保已安装。"
    fi
}
 
# ----------------------------- 主程序 -----------------------------
 
# 1. 检查执行权限
if [[ $EUID -ne 0 ]]; then
   log_error "此脚本必须以 root 或 sudo 权限执行。"
fi
 
log_info "========== 开始安装 Nginx ${NGINX_VERSION}（含 VTS 模块）=========="
 
# 2. 检查系统环境
dnf -y install wget tar make gcc
log_info "检查系统环境..."
check_command "wget"
check_command "tar"
check_command "make"
check_command "gcc"
 
# 3. 安装编译依赖（增加 unzip 以防万一）
log_info "安装编译依赖包..."
dnf install -y epel-release || log_error "安装 EPEL 仓库失败。"
dnf install -y gcc gcc-c++ autoconf libtool pcre pcre-devel zlib zlib-devel openssl openssl-devel || {
    log_error "安装依赖包失败，请检查网络或软件源。"
}
log_info "依赖包安装完成。"
 
# 4. 创建 Nginx 用户和组
log_info "创建 Nginx 用户和组..."
if ! getent group "${NGINX_GROUP}" &> /dev/null; then
    groupadd -r "${NGINX_GROUP}" || log_error "创建组 ${NGINX_GROUP} 失败。"
    log_info "组 ${NGINX_GROUP} 创建成功。"
else
    log_info "组 ${NGINX_GROUP} 已存在。"
fi
 
if ! id "${NGINX_USER}" &> /dev/null; then
    useradd -r -g "${NGINX_GROUP}" -s /sbin/nologin "${NGINX_USER}" || log_error "创建用户 ${NGINX_USER} 失败。"
    log_info "用户 ${NGINX_USER} 创建成功。"
else
    log_info "用户 ${NGINX_USER} 已存在。"
fi
 
# 5. 下载 Nginx 源码
log_info "下载 Nginx ${NGINX_VERSION} 源码..."
if [[ -f "${SOURCE_DIR}/${NGINX_TARBALL}" ]]; then
    log_info "源码包 ${NGINX_TARBALL} 已存在，跳过下载。"
else
    wget -P "${SOURCE_DIR}" "${NGINX_URL}" || log_error "下载源码失败，请检查 URL: ${NGINX_URL}"
    log_info "源码下载成功。"
fi
 
# 6. 解压 Nginx 源码
log_info "解压 Nginx 源码到 ${SOURCE_DIR}..."
cd "${SOURCE_DIR}"
tar -xzf "${NGINX_TARBALL}" || log_error "解压 Nginx 源码失败。"
log_info "Nginx 源码解压成功。"
 
# 7. 下载 VTS 模块源码
log_info "下载 VTS 模块源码（版本 ${VTS_VERSION}）..."
if [[ -f "${SOURCE_DIR}/${VTS_TARBALL}" ]]; then
    log_info "VTS 模块源码包 ${VTS_TARBALL} 已存在，跳过下载。"
else
    wget -P "${SOURCE_DIR}" "${VTS_URL}" || log_error "下载 VTS 模块失败，请检查 URL: ${VTS_URL}"
    log_info "VTS 模块源码下载成功。"
fi
 
# 8. 解压 VTS 模块源码
log_info "解压 VTS 模块源码到 ${SOURCE_DIR}..."
if [[ -d "${VTS_DIR}" ]]; then
    log_info "VTS 模块目录已存在，跳过解压。"
else
    tar -xzf "${VTS_TARBALL}" -C "${SOURCE_DIR}" || log_error "解压 VTS 模块源码失败。"
    log_info "VTS 模块源码解压成功。"
fi
 
# 9. 配置编译选项（包含 VTS 模块）
log_info "配置编译选项（含 --add-module=${VTS_DIR}）..."
cd "${NGINX_SOURCE}"
./configure \
    --prefix="${INSTALL_DIR}" \
    --user="${NGINX_USER}" \
    --group="${NGINX_GROUP}" \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_realip_module \
    --with-http_stub_status_module \
    --with-http_gzip_static_module \
    --with-pcre \
    --with-stream \
        --with-stream_ssl_module \
    --with-file-aio \
    --with-http_addition_module \
    --with-http_sub_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_gunzip_module \
    --with-http_auth_request_module \
    --with-http_random_index_module \
    --with-http_secure_link_module \
    --with-http_degradation_module \
    --with-http_slice_module \
    --add-module="${VTS_DIR}" \
    || log_error "配置失败，请检查依赖和参数。"
 
log_info "配置成功。"
 
# 10. 编译与安装
log_info "开始编译 (make)..."
make -j "$(nproc)" || log_error "编译失败。"
log_info "编译成功。"
 
log_info "开始安装 (make install)..."
make install || log_error "安装失败。"
log_info "安装成功，安装目录: ${INSTALL_DIR}"
 
# 11. 创建软链接
log_info "创建 nginx 软链接到 /usr/sbin..."
ln -sf "${INSTALL_DIR}/sbin/nginx" "/usr/sbin/nginx" || log_error "创建软链接失败。"
log_info "软链接创建成功。"
 
# 12. 配置 systemd 服务
log_info "配置 systemd 服务..."
 
cat > /usr/lib/systemd/system/nginx.service << EOF
[Unit]
Description=nginx - high performance web server
Documentation=https://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target
 
[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStart=${INSTALL_DIR}/sbin/nginx -c ${INSTALL_DIR}/conf/nginx.conf
ExecReload=/bin/kill -s HUP \$MAINPID
ExecStop=/bin/kill -s TERM \$MAINPID
LimitNOFILE=655360
 
[Install]
WantedBy=multi-user.target
EOF
 
# 15. 配置nginx.conf
mkdir -p ${INSTALL_DIR}/conf/vhost
 
ln -sf "${INSTALL_DIR}/conf/vhost" "${INSTALL_DIR}/" || log_error "创建软链接失败。"
 
mkdir -p /data/logs/nginx
chown -R nginx:nginx /data/logs/nginx
 
cat >${INSTALL_DIR}/conf/nginx.conf<<'EOF'
user nginx nginx;
worker_processes auto;
worker_cpu_affinity auto;
pid /run/nginx.pid;
worker_rlimit_nofile 65536;
 
events {
    use  epoll;
    worker_connections 65535;
    multi_accept on;
}
 
http {
    vhost_traffic_status_zone;
    vhost_traffic_status_zone shared:vhost_traffic_status:64m;
    server_tokens off;
    include       mime.types;
    default_type  application/octet-stream;
    server_names_hash_bucket_size 128;
    client_header_buffer_size     32k;
    large_client_header_buffers 4 32k;
    client_max_body_size    20m;
    client_body_buffer_size 5m;
    types_hash_max_size     2048;
    sendfile          on;
    tcp_nopush        on;
    tcp_nodelay       on;
    keepalive_timeout 300;
 
    map $http_x_forwarded_for  $clientRealIp {
        ""  $remote_addr;
        ~^(?P<firstAddr>[0-9\.]+),?.*$  $firstAddr;
    }
 
    log_format info
        '[$request_time] $clientRealIp [$time_local] "$request" '
        '$status $request_body $body_bytes_sent "$http_referer" '
        '"$http_user_agent" [$upstream_response_time] $upstream_addr '
        '$upstream_response_length $upstream_bytes_received $upstream_status';
 
    log_format main '{"@timestamp":"$time_iso8601",'
                  '"@source":"$server_addr",'
                  '"hostname":"$hostname",'
                  '"ip":"$remote_addr",'
                  '"client":"$remote_addr",'
                  '"request_method":"$request_method",'
                  '"scheme":"$scheme",'
                  '"domain":"$server_name",'
                  '"referer":"$http_referer",'
                  '"request":"$request_uri",'
                  '"args":"$args",'
                  '"size":$body_bytes_sent,'
                  '"status": $status,'
                  '"responsetime":$request_time,'
                  '"upstreamtime":"$upstream_response_time",'
                  '"upstreamaddr":"$upstream_addr",'
                  '"http_user_agent":"$http_user_agent",'
                  '"https":"$https"'
                  '}';
 
    log_format json '{"@timestamp":"$time_iso8601",'
                  '"@source":"$server_addr",'
                  '"hostname":"$hostname",'
                  '"ip":"$remote_addr",'
                  '"client":"$remote_addr",'
                  '"request_method":"$request_method",'
                  '"scheme":"$scheme",'
                  '"domain":"$server_name",'
                  '"referer":"$http_referer",'
                                    '"request":"$request_uri",'
                  '"args":"$args",'
                  '"size":$body_bytes_sent,'
                  '"status": $status,'
                  '"responsetime":$request_time,'
                  '"upstreamtime":"$upstream_response_time",'
                  '"upstreamaddr":"$upstream_addr",'
                  '"http_user_agent":"$http_user_agent",'
                  '"https":"$https"'
                  '}';
 
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout    300;
    fastcgi_read_timeout    300;
    fastcgi_buffer_size     64k;
    fastcgi_buffers       4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 256k;
 
    proxy_read_timeout 300;
    proxy_send_timeout 300;
    proxy_connect_timeout 300;
    proxy_buffer_size 128k;
    proxy_buffers 4 128k;
    proxy_busy_buffers_size 256k;
    proxy_temp_file_write_size 128k;
 
    access_log /data/logs/nginx/access.log json;
    error_log /data/logs/nginx/error.log;
 
    gzip on;
    gzip_min_length  1k;
    gzip_buffers  4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 4;
    gzip_types text/plain application/javascript application/x-javascript text/javascript text/css application/xml application/xml+rss;
    gzip_vary on;
    gzip_proxied expired no-cache no-store private auth;
    gzip_disable "MSIE [1-6]\.";
    #include vhost/upstream.list;
    include vhost/*.conf;
}
EOF
 
 
systemctl daemon-reload || log_error "systemd 重载失败。"
log_info "systemd 服务文件创建成功。"
 
# 13. 启动 Nginx 并设置开机自启
log_info "启动 Nginx 服务并设置开机自启..."
systemctl enable --now nginx || log_error "启动 Nginx 服务失败。"
log_info "Nginx 服务启动成功。"
 
# 14. 验证安装（包含 VTS 模块）
log_info "验证 Nginx 安装（包含 VTS 模块）..."
nginx -v || log_error "Nginx 版本验证失败。"
log_info "Nginx 版本信息: $(nginx -v 2>&1)"
# 检查 VTS 模块是否已加载
if nginx -V 2>&1 | grep -q "vts"; then
    log_info "✅ VTS 模块已成功集成。"
else
    log_warn "⚠️ VTS 模块未在编译参数中显示，请手动检查。"
fi
 
# 16. 安装完成
log_info "========== Nginx ${NGINX_VERSION} 安装完成 =========="
log_info "安装路径: ${INSTALL_DIR}"
log_info "配置文件: ${INSTALL_DIR}/conf/nginx.conf"
log_info "VTS 模块状态: 已集成 (配置指令 vhost_traffic_status_zone 可用)"
log_info "服务管理: systemctl {start|stop|restart|reload|status} nginx"
log_info "访问测试: curl http://localhost 或通过浏览器访问服务器 IP"
```