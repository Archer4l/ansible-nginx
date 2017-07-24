If you want redirect all from 80 to 443 use next syntax:
    	

       nginx_remove_default_vhost: true

      - listen: "80  default_server"
        server_name: "_"
        return: "301 https://$host$request_uri"
        filename: "default.conf"