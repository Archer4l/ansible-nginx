## Example nginx config(with tls-certificate) 

Next we configure nginx server with 3 virtual hosts.
site-proxy.in.example.com - it's a proxy to jabber.example.com:1111
site-0.in.example.com - just site
site-1.in.example.com - just site

And redirect host from 80 to 443

0. Setup tls_cetificate, and when we play tls_certificate_role, we get variable: tls_certificate_path_crt_full, tls_certificate_path_key,tls_certificate_path_crt,tls_certificate_path_ca


    tls_certificate_name: "star.in.example.com"


Configure proxy, part 1

    nginx_upstreams:
      - name: backend
        strategy: "ip_hash" # "least_conn", etc.
        keepalive: 16 # optional
        servers: {
          "jabber.example.com:1111"
        }
    
Redirect host from 80 to 443
    
    nginx_vhosts:
      - listen: "80"
        server_name: "site-proxy.in.example.com www.site-proxy.in.example.com"
        return: "301 https://site-proxy.in.example.com$request_uri"
        filename: "site-proxy.in.example.com.80.conf"
      - server_name: "site-1.in.example.com www.site-1.in.example.com"
        return: "301 https://site-1.in.example.com$request_uri"
        filename: "site-1.in.example.com.80.conf"
        access_log: "/var/lib/nginx/site-1.in.example.com.access.log"
        error_log: "/var/lib/nginx/site-1.in.example.com.error.log"
      - server_name: "site-0.in.example.com www.site-0.in.example.com"
        return: "301 https://site-0.in.example.com$request_uri"
        filename: "site-0.in.example.com.80.conf"
        access_log: "/var/lib/nginx/site-0.in.example.com.access.log"
        error_log: "/var/lib/nginx/site-0.in.example.com.error.log"

Configure proxy, part 2. Configure proxy with http2 and ssl

!! Waring http2 not working on default nginx in ubuntu 14.04. If you don't now what is it - pls skip http2 
    
      - listen: "443 ssl http2"
        server_name: "site-proxy.in.example.com"
        server_name_redirect: "www.site-proxy.in.example.com"
        extra_parameters: |
              ssl_certificate     {{ tls_certificate_path_crt_full }};
              ssl_certificate_key {{ tls_certificate_path_key }};
              ssl_protocols       TLSv1.1 TLSv1.2;
              ssl_ciphers         HIGH:!aNULL:!MD5;
              location / {
                  proxy_pass http://backend/;
                  proxy_http_version 1.1;
                  proxy_set_header Upgrade $http_upgrade;
                  proxy_set_header Connection "upgrade";
                  proxy_set_header Host $http_host;
    
                  proxy_set_header X-Real-IP $remote_addr;
                  proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
                  proxy_set_header X-Forward-Proto http;
                  proxy_set_header X-Nginx-Proxy true;
    
                  proxy_redirect off;
              }

Configure vhost on port 443 with http2 and ssl
       
      - listen: "443 ssl http2"
        server_name: "site-0.in.example.com"
        root: "/var/www/site-0.in.example.com"
        access_log: "/var/lib/nginx/site-0.in.example.com.access.log"
        error_log: "/var/lib/nginx/site-0.in.example.com.error.log"
        index: "index.php index.html index.htm"
        extra_parameters: |
              ssl_certificate     {{ tls_certificate_path_crt_full }};
              ssl_certificate_key {{ tls_certificate_path_key }};
              ssl_protocols       TLSv1.1 TLSv1.2;
              ssl_ciphers         HIGH:!aNULL:!MD5;
    
      - listen: "443 ssl http2"
        server_name: "site-1.in.example.com"
        root: "/var/www/site-1.in.example.com"
        index: "index.php index.html index.htm"
        access_log: "/var/lib/nginx/site-1.in.example.com.access.log"
        error_log: "/var/lib/nginx/site-1.in.example.com.error.log"
        extra_parameters: |
              ssl_certificate     {{ tls_certificate_path_crt_full }};
              ssl_certificate_key {{ tls_certificate_path_key }};
              ssl_protocols       TLSv1.1 TLSv1.2;
              ssl_ciphers         HIGH:!aNULL:!MD5;

              

tls_certificate_path_crt_full and tls_certificate_path_key depended from tls_certificate_name: "star.in.example.com"