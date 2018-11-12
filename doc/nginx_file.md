
file_server.conf

内容：

  server {

    client_max_body_size 4G;
    
    charset utf-8;
    
    ##注意80端口的占用问题
    listen  8391;  ## listen for ipv4; this line is default and implied
    
    server_name    XXX.XXX.XXX;  ##你的主机名或者是域名
    
    root /opt/maitao;
    
    # /etc/nginx/nginx_http_passwd.conf
    location / {
    	  auth_basic "MyPath Authorized";
	      auth_basic_user_file /etc/nginx/nginx_http_passwd.conf; # 指定密码文件
        autoindex on; ##显示索引
        autoindex_exact_size off; ##显示大小
        autoindex_localtime on;   ##显示时间
    }
}



生成nginx 密码文件：
wget -c http://www.huzs.net/soft/htpasswd.sh
bash htpasswd.sh

使用NgxFancyIndex美化nginx 文件服务器页面
# 官网： http://wiki.nginx.org/NgxFancyIndex
# wget http://gitorious.org/ngx-fancyindex/ngx-fancyindex/archive-tarball/master
# tar -xzvf master
# wget http://nginx.org/download/nginx-1.4.2.tar.gz
# tar -xzvf nginx-1.4.2.tar.gz
# cd nginx-1.4.2
# ./configure --prefix=/usr/local/nginx-1.4.2  --add-module=../ngx-fancyindex-ngx-fancyindex
# make
# make install
