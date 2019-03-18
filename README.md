# Video CDN

NGINX has the ability to deliver Live and VOD streams across an enterprise network or as a global CDN with the following configurations.

# Infrastructure
There are 2 scanarios covered in this configuration environment. Cloud services providers will host and deliver content from their own URL's and in some cases it may be an option to deliver media content using a custom URL based on the customers domain but still delivered from the cloud.

On premise content delivered from internal media streaming servers can be an easier implementation of this solution but may require additional resources for streaming media live and hosting video content from internal web servers and content management systems.

## Cloud Service Providers
In order to achieve the delivery of live and VOD content from cloud service providers, it is recommened to use Split DNS to resolve the External DNS name of the service to internal servers. This will allow for there to be no changes made to the service configurations of the incoming applications content.

## On Premise
For an on premise implementation of this solution, a streaming media server for live streaming will need to be implemented and a cms or web server will need to be in place to deliver HLS adaptive VOD assets.

# Configuration Components

## Considerations

### CORS
Cross Origin Resource Sharing (CORS)

### Players

JWPlayer, Video.js

### CDN's

Akamai, Cloudfront

### Caching

Headers

## Redirector
The purpose of the redirector is to provide a central point to resolve requests for the video stream, live or on-demand, and redirect the session to the server allocated to serve content to the users IP range.

/etc/nginx/nginx.conf
```
# STREAMING - Redirector V2
worker_processes  auto;
worker_rlimit_nofile 30000;
events {
    worker_connections  200000;

}

# START HTTP BLOCK
http {
server_names_hash_bucket_size  128;
include       mime.types;
default_type  application/octet-stream;

#SET THE IP ADDRESS RANGE OF THE LOCATION TO BE REDIRECTED FOR LOCATION A
geo $location _A
{ default 0; 10.0.0.0/24 1;}
#SET THE IP ADDRESS RANGE OF THE LOCATION TO BE REDIRECTED FOR LOCATION B
geo $location_B
{ default 0; 10.0.1.0/23 1;}


# START LOG FORMATTING
log_format upstream_time '$remote_addr - $remote_user [$time_local] ' '"$request" $status $body_bytes_sent ' '"$http_referer" "$http_user_agent"' 'rt=$request_time uct="$upstream_connect_time" uht="$upstream_header_time status="$upstream_cache_status" urt="$upstream_response_time"';
# END LOG FORMATTING


# START CACHING VARIABLES
proxy_cache_path /usr/share/nginx/cache levels=1:2 keys_zone=hls-test1-cache:200m max_size=1000m inactive=600m;
client_body_temp_path /spool 1 2;
client_body_buffer_size 16k;
# END CACHING VARIABLES

include /etc/nginx/conf.d/*.conf;
}

```

/etc/nginx/conf.d/default.conf

```

server{
listen 80;
listen 443 ssl;
server_name *.<redirector host name> ;
 # START CERTIFICATE CHAINS
        #ssl on;
        ssl_certificate         /etc/nginx/ssl/<certificate name>.crt;
        ssl_certificate_key     /etc/nginx/ssl/<certificate key name>.key;
       	ssl_trusted_certificate /etc/nginx/ssl/<ca cert name>.crt;
       	ssl_session_cache shared:SSL:10m;
       	ssl_session_timeout 5m;
      	ssl_prefer_server_ciphers       on;
       	ssl_protocols                   TLSv1 TLSv1.1 TLSv1.2;
       	ssl_ciphers                     ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS;
        # START CERTIFICATE CHAINS
location / {
	add_header "Who_did_it_hit" "Director";
	add_header "Access-Control-Allow-Origin" "*";
location ~* .(xml)$ {
        root	/usr/share/nginx/html;
        add_header "X-Hls-Cache-Status" "Cross Domain XML File";
}

if ($location_A)
{  
return 301 http://<oad balancer host location A>$request_uri;
}
if ($location_B)
{  
return 301 http://<load balancer host location B>$request_uri; }
}
}

```

## Load Balancer
The load balancer will proxy the connections to the back end servers.

/etc/nginx/nginx.conf
```
#STREAMING - Load Balancer - LOCATION A V2.0
worker_processes  auto;
events {
    worker_connections  20000;
}

# START HTTP BLOCK
http {
log_format upstream_time '$remote_addr - $remote_user [$time_local] ' '"$request" $status $body_bytes_sent ' '"$http_referer" "$http_user_agent"' 'rt=$request_time uct="$upstream_connect_time" uht="$upstream_header_time status="$upstream_cache_status" urt="$upstream_response_time"';
#gzip on;
#gzip_comp_level 9;

# START UPSTREAM BACKEND VARIABLES
upstream backend {
        #ip_hash;
        #least_conn;
	#least_time last_byte;
	least_time header;
        server 10.0.0.201:80 weight=100;
       	server 10.0.0.202:80 weight=100;
	sticky cookie srv_id expires=1h domain=.example.com path=/;
	keepalive 64;
}
# END UPSTREAM BACKEND VARIABLES
include /etc/nginx/conf.d/*.conf;
}
# END HTTP BLOCK

```

/etc/nginx/conf.d/default.conf
```

# START SERVER BLOCK
    server {
        listen 80;
        listen 443 ssl;
        server_name <HOST NAME>;

        # START CERTIFICATE CHAINS
        #ssl on;
        ssl_certificate         /etc/nginx/ssl/<certificate name>.crt;
        ssl_certificate_key     /etc/nginx/ssl/<certificate name>.key;
       	ssl_trusted_certificate /etc/nginx/ssl/<ca certificate name>.crt;
       	ssl_session_cache shared:SSL:10m;
       	ssl_session_timeout 5m;
      	ssl_prefer_server_ciphers       on;
       	ssl_protocols                   TLSv1 TLSv1.1 TLSv1.2;
       	ssl_ciphers                     ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS;
        # START CERTIFICATE CHAINS

        # START LOGS
        access_log /var/log/nginx/access.log upstream_time;
        error_log /var/log/nginx/error.log;
        # END LOGS

# START ROOT LOCATION BLOCK

location / {
	#add_header "Access-Control-Allow-Origin" "*";
        proxy_pass http://backend;
        proxy_set_header Host            $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        }
# END ROOT LOCATION BLOCK
    }
# END SERVER BLOCK
```


## Cache
The  cache is in place to deliver the content from memory or fetch content from the origin server if the content has not yet been cached and load it into the cache according to the caching rules applied.

/etc/nginx/nginx.conf

```
# STREAMING - Cache Version 2.0

worker_processes  auto;
error_log  /var/log/nginx/error.log debug;
events {
	worker_connections  1024;
}


# START HTTP BLOCK
http {
	server_names_hash_bucket_size  128;
	include       mime.types;
	default_type  application/octet-stream;
	directio 10m;

# START LOG FORMATTING
log_format upstream_time '$remote_addr - $remote_user [$time_local] ' '"$request" $status $body_bytes_sent ' '"$http_referer" "$http_user_agent"' 'rt=$request_time uct="$upstream_connect_time" uht="$upstream_header_time status="$upstream_cache_status" urt="$upstream_response_time"';
# END LOG FORMATTING


# START CACHING VARIABLES
	proxy_cache_path /usr/share/nginx/cache levels=1:2 keys_zone=hls-test1-cache:200m max_size=1000m inactive=600m;
	client_body_temp_path /spool 1 2;
	client_body_buffer_size 16k;
# END CACHING VARIABLES

include /etc/nginx/conf.d/*.conf;
}
# END HTTP BLOCK

```
/etc/nginx/conf.d/default.conf
```

# START SERVER BLOCK for all streaming VOD and live streaming traffic requests.
server {
        listen 80;
	listen 443 ssl;
        server_name *.<HOST NAME>;
# SERVE CROSSDOMAIN 
location / {
	#add_header "Access-Control-Allow-Origin" "*";
        
#ALL XML REQUESTS WILL BE SERVED FROM THE CACHES HTTP FOLDER
location ~* .(xml)$ {
        root	/usr/share/nginx/html;
        add_header "X-Hls-Cache-Status" "Cross Domain XML File";
}
# END XML FILE LOCATION

#####VOD#####.
# START LOCATION CDN VOD
location ~* /(au|br|ie|jp|sg|us)/ {
# START M3U8 VOD
location ~* .(m3u8)$ {
        resolver 8.8.8.8;
        proxy_pass $scheme://<VOD origin host>$uri$is_args$args;
       	proxy_cache hls-test1-cache;
        proxy_cache_valid 200 10s;
	add_header X-Proxy-Cache $upstream_cache_status;
        add_header "Who did it hit?" "streaming VOD Cached M3U8 on Cache"; 
  }
  # END M3U8 VOD
  
  #START TS VOD
location ~* .(ts|trp)$ {
		resolver 8.8.8.8;
        proxy_pass $scheme://<VOD origin host>$uri$is_args$args;
       	proxy_cache hls-test1-cache;
        proxy_cache_valid 200 7d;
		add_header X-Proxy-Cache $upstream_cache_status;
        add_header "Who did it hit?" "streaming VOD Cached M3U8 on Cache"; 
		proxy_ignore_headers Cache-Control;
        #proxy_ignore_headers Set-Cookie;
		proxy_ignore_headers "Expires";
		proxy_hide_header "Expires"; 
		proxy_hide_header "Cache-Control";
		proxy_hide_header "Set-Cookie";
}
location ~* .(aac)$ {
        resolver 8.8.8.8;
        proxy_pass $scheme://<VOD origin host>$uri$is_args$args;
       	proxy_cache hls-test1-cache;
        proxy_cache_valid 200 7d;
		add_header X-Proxy-Cache $upstream_cache_status;
        add_header "Who did it hit?" "streaming VOD Cached M3U8 on Cache"; 
  }
location ~* .(mp4)$ {
        resolver 8.8.8.8;
        proxy_pass $scheme://<VOD origin host>$uri$is_args$args;
       	proxy_cache hls-test1-cache;
        proxy_cache_valid 200 7d;
	add_header X-Proxy-Cache $upstream_cache_status;
        add_header "Who did it hit?" "streaming VOD Cached M3U8 on Cache"; 
  }
# END streaming VOD TS FILE LOCATION
}
}
# END streaming ROOT LOCATION 
####VOD#####


####LIVE#####
#START LIVE LOCATION IDENTIFIER FOLDER
#The location /i/ can be any directory or uri however it is used to identify the unique location to defferenciate between live and VOD traffic.
location /i/ {

# START LIVE M3U8 FILE LOCATION
location ~* .(m3u8)$ {
	proxy_http_version 1.1;
        proxy_set_header Connection "";
        resolver 8.8.8.8;
        proxy_pass $scheme://<live origin host>$uri$is_args$args;
        proxy_cache hls-test1-cache;
        proxy_cache_valid 200 9s;
        #proxy_cache_valid 404 5s; 
        proxy_set_header     Host $host; 
        proxy_ignore_headers Cache-Control;
        proxy_ignore_headers Set-Cookie;
	proxy_ignore_headers "Expires";
	proxy_hide_header "Expires"; 
	proxy_hide_header "Cache-Control";
	proxy_hide_header "Set-Cookie";
        proxy_cache_key $scheme$proxy_host$request_uri;
        #proxy_cache_lock on;
        #proxy_cache_lock_age 10s;
        proxy_cache_use_stale updating;
        proxy_ignore_client_abort on;
        add_header X-Proxy-Cache $upstream_cache_status;
        add_header "X-Hls-Cache-Status" "streaming Live Cached M3U8 on Cache"; 
 
}
# END LIVE M3U8 FILE LOCATION

# START streaming LIVE TS FILE LOCATION
location ~* .(ts|trp)$ {
		proxy_http_version 1.1;
        proxy_set_header Connection "";
	    resolver 8.8.8.8;
        proxy_pass $scheme://<live origin host>$uri$is_args$args;
        proxy_cache hls-test1-cache;
        proxy_cache_valid 200 1m;
	proxy_buffering on;
	proxy_buffers 16 640k;
	proxy_busy_buffers_size 9m;
	proxy_buffer_size 9m;
        proxy_set_header     Host $host; 
        proxy_ignore_headers "Cache-Control";
	proxy_ignore_headers "Set-Cookie";
	proxy_ignore_headers "Expires";
	proxy_hide_header "Expires"; 
	proxy_hide_header "Cache-Control";
	proxy_hide_header "Pragma"; 
	proxy_hide_header "Set-Cookie";
	proxy_cache_key $scheme$proxy_host$request_uri;
	#proxy_cache_lock on;
	#proxy_cache_lock_age 60s;
        proxy_ignore_client_abort on;
	proxy_cache_use_stale updating;
        proxy_cache_revalidate on;
	add_header X-Proxy-Cache $upstream_cache_status;
        add_header "X-Hls-Cache-Status" "streaming Live Cached TS on Cache";  
}
# END streaming LIVE TS FILE LOCATION
}
# END LIVE SERVER BLOCK
location /live/ {

# START LIVE M3U8 FILE LOCATION
location ~* .(mpd)$ {
	proxy_http_version 1.1;
        proxy_set_header Connection "";
        resolver 8.8.8.8;
        proxy_pass $scheme://<live origin host>$uri$is_args$args;
        proxy_cache hls-test1-cache;
        proxy_cache_valid 200 9s;
        #proxy_cache_valid 404 5s; 
        proxy_set_header     Host $host; 
        proxy_ignore_headers Cache-Control;
        proxy_ignore_headers Set-Cookie;
	proxy_ignore_headers "Expires";
	proxy_hide_header "Expires"; 
	proxy_hide_header "Cache-Control";
	proxy_hide_header "Set-Cookie";
        proxy_cache_key $scheme$proxy_host$request_uri;
        #proxy_cache_lock on;
        #proxy_cache_lock_age 10s;
        proxy_cache_use_stale updating;
        proxy_ignore_client_abort on;
        add_header X-Proxy-Cache $upstream_cache_status;
        add_header "X-Hls-Cache-Status" "streaming Live Cached M3U8 on Cache"; 
 
}
# END LIVE M3U8 FILE LOCATION

# START streaming LIVE TS FILE LOCATION
location ~* .(m4s)$ {
	proxy_http_version 1.1;
        proxy_set_header Connection "";
	resolver 8.8.8.8;
        proxy_pass $scheme://<live origin host>$uri$is_args$args;
        proxy_cache hls-test1-cache;
        proxy_cache_valid 200 1m;
	proxy_buffering on;
	proxy_buffers 16 640k;
	proxy_busy_buffers_size 9m;
	proxy_buffer_size 9m;
        proxy_set_header     Host $host; 
        proxy_ignore_headers "Cache-Control";
	proxy_ignore_headers "Set-Cookie";
	proxy_ignore_headers "Expires";
	proxy_hide_header "Expires"; 
	proxy_hide_header "Cache-Control";
	proxy_hide_header "Pragma"; 
	proxy_hide_header "Set-Cookie";
	proxy_cache_key $scheme$proxy_host$request_uri;
	#proxy_cache_lock on;
	#proxy_cache_lock_age 60s;
        proxy_ignore_client_abort on;
	proxy_cache_use_stale updating;
        proxy_cache_revalidate on;
	add_header X-Proxy-Cache $upstream_cache_status;
        add_header "X-Hls-Cache-Status" "streaming Live Cached TS on Cache";  
}
# END streaming LIVE TS FILE LOCATION
}
# END LIVE SERVER BLOCK
}
# END  SERVER BLOCK

```
