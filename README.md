# NCF

#Postgresql

- sudo cp -r /var/lib/postgresql/ /efs
- sudo chown -R postgres:postgres /efs/postgresql/9.5/
- in /etc/postgresql/9.5/main/postgresql.conf change data_directory to data_directory = '/efs/postgresql/9.5/main'

#tarantool
- sudo mkdir /efs/tarantool
- sudo cp -r /opt/ntech/var/lib/tarantool/shard-001 /efs/tarantool/
- sudo cp -r /opt/ntech/var/lib/tarantool/shard-002 /efs/tarantool/

nano /etc/tarantool/instances.available/shard-001.lua
nano /etc/tarantool/instances.available/shard-002.lua

- in shards config files (/etc/tarantool/instances.available/*.lua) I changed all the dir paths to new ones:
    --Directory to store data
    vinyl_dir = '/efs/tarantool/shard-001',
    work_dir = '/efs/tarantool/shard-001',
    memtx_dir = '/efs/tarantool/shard-001/snapshots',
    wal_dir = '/efs/tarantool/shard-001/xlogs

#ffsecurity storage

- sudo cp -r /var/lib/ffsecurity /efs/
- sudo chown -R ntech:ntech /efs/ffsecurity/
- edit /etc/ffsecurity/config.py and change STATIC_ROOT and MEDIA_ROOT to
MEDIA_ROOT = "/efs/ffsecurity/uploads"
STATIC_ROOT = "/efs/ffsecurity/static"
- edit /etc/nginx/sites-available/ffsecurity-nginx.conf and replace all /var/lib to /efs, namely:
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /efs/ffsecurity;
...
        location ~ /videos/(?<video_id>[0-9]+)/upload/(.*)$ {
                if ($request_method = 'OPTIONS') {
                        add_header 'Content-Type' 'text/plain; charset=utf-8';
                        add_header 'Content-Length' 0;
                        return 204;
                }
                set $auth_request_uri "http://ffsecurity/videos/$video_id/auth-upload/";
                auth_request /video-upload-auth/;

                alias "/efs/ffsecurity/uploads/videos/$video_id.bin";

#ffupload

- sudo cp -r /var/lib/ffupload/ /efs
- sudo chmod a+w /efs/ffupload/uploads/
- edit /etc/nginx/sites-available/ffupload  and replace all /var/lib to /efs, namely:
...
    location /uploads {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET';
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization';
        add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
        add_header 'Access-Control-Max-Age' 2592000;

        client_max_body_size 15g;
        root /efs/ffupload;
...
    location / {
        root /efs/ffupload;
    }.
