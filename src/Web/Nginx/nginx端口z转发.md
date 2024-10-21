
stream {
        upstream postgresql {
                server 10.240.53.58:5432;
        }
        server {
                listen 20001;
                proxy_connect_timeout 600s;
                proxy_pass postgresql;
        }
}