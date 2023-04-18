## 问题描述
因为双向封锁的原因，现在在大陆直接请求已经无法请求到 OpenAI 的API了。这里借助一台海外服务器 + Nginx， 就可以实现请求的转发，让你在大陆的服务器也能正常请求到 OpenAI。

## 准备内容
- 非**中国大陆**和**中国香港地区**的海外服务器一台。可以选择各大云服务厂商的轻量应用服务器，每个月的费用较低，最低大概20到40元每月；对于个人或者小型应用，每个月的流量应该也是足够使用了。
- 在海外服务器上安装 Nginx，我这里安装的版本是1.22.x。

## 修改 Nginx 的配置文件
```shell
# 下面为 Nginx的nginx.conf文件的 server 的配置
server
    {
        listen 80;
        server_name www.host.com
        index index.html index.htm index.php;
        root /www/server/nginx/html;
        charset 'utf-8';
        access_log  /www/wwwlogs/access.log;

        location / {
          try_files $uri $uri/ /index.html/#/chat;
        }

        # 本地请求地址: http://www.host.com/v2/chat/completions
        # 转发后的地址: https://api.openai.com/v1/chat/completions
        location /v2/ {
          proxy_pass https://api.openai.com/v1/;
          proxy_ssl_server_name on;
          proxy_intercept_errors off;
          add_header X-Nginx-Proxy "true";
          # 如果要启用 SSE，需要讲下面注释掉的代码启用
          # add_header Content-Type text/event-stream;
          # add_header Cache-Control no-cache;
          # proxy_buffering off;
          # proxy_cache off;
        }

        # 本地请求地址: http://www.host.com/v3/v1/chat/completions
        # 转发后的地址: https://api.openai.com/v1/chat/completions
        location /v3/ {
          proxy_pass https://api.openai.com/;
          proxy_ssl_server_name on;
          proxy_intercept_errors off;
          add_header X-Nginx-Proxy "true";
          # 如果要启用 SSE，需要讲下面注释掉的代码启用
          # add_header Content-Type text/event-stream;
          # add_header Cache-Control no-cache;
          # proxy_buffering off;
          # proxy_cache off;
        }
    }
```


