# Full-WebDAV-Nginx-Docker
# Full-WebDAV-Nginx-Docker
一、有什么用？请参阅https://github.com/arut/nginx-dav-ext-module

就是从nginx官方的dockerfile里加入了上面github链接的nginx-dav-ext-module，从而实现完整的webdav。

基于alpine3.10 nginx1.17.5 Image大小17.2MB 

nginx.conf里默认加载了webdav模块，即load_module modules/ngx_http_dav_ext_module.so;

你可以使用dockerfile自己编译，也可以docker hub直接拉取我的Image
docker pull vizshrc/nginx
详情：https://hub.docker.com/repository/docker/vizshrc/nginx


二、使用方式：
1.创建nginx配置文件
vi /etc/nginx/conf.d/davhttps.conf

写入示例文件：
    server {
    listen  443 ssl;
    #ssl on;
    ssl_certificate       /root/.acme.sh/feeling.com_ecc/feeling.com.cer;
    ssl_certificate_key   /root/.acme.sh/feeling.com_ecc/feeling.com.key;
    ssl_protocols         TLSv1 TLSv1.1 TLSv1.2 ;
    ssl_ciphers           HIGH:!aNULL:!MD5;
    server_name           dl.feeleg.com;

#这里说明网站地址
    error_log /var/log/nginx/webdav.error.log error;
    access_log  /var/log/nginx/webdav.access.log combined;
    location / {
        root /root/;
        #root /root/VpsDownload/;
        charset utf-8;
        autoindex on;
        dav_methods PUT DELETE MKCOL COPY MOVE;
        dav_ext_methods PROPFIND OPTIONS;
        create_full_put_path  on;
        dav_access user:rw group:r all:r;
        auth_basic "Authorized Users Only";
        auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
        client_max_body_size 100m;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
   root   /usr/share/nginx/html;
   }
}

2.创建用户，最好把.htpasswd放在/etc/nginx/conf.d下面，所以命令是

htpasswd -c /etc/nginx/conf.d/.htpasswd 用户名

如果提示找不到命令。需要先安装htpasswd命令，debian和ubuntu使用

apt-get install apache2-utils

3.创建并启动docker,至少需要挂载/etc/nginx/conf.d  
  同时container中包含nginx.conf已经加载webdav模块，所以不建议挂载整个/etc/nginx,
  这会覆盖container中的/etc/nginx,这样会宿主机目录没有这个模块或nginx.conf没有写加载模块而失败，如果分开挂载则麻烦

docker run --name nginx -p 443:443 -v /etc/nginx/conf.d:/etc/nginx/conf.d:ro -v /root:/root/:ro -d vizshrc/nginx
到这里正常来说就成功了，可以去访问试试

4.我个人需要的挂载的命令调整如下：
docker run --name nginx -p 443:443 -v /etc/nginx/conf.d:/etc/nginx/conf.d:ro -v /var/www:/var/www:ro -v /root:/root/:ro --net v2-net -d vizshrc/nginx

