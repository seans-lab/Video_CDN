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

## Redirector
The purpose of the redirector is to provide a central point to resolve requests for the video stream, live or on-demand, and redirect the session to the server allocated to serve content to the users IP range.

`# STREAMING - Redirector V1.8
worker_processes  2;
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

`

## Load Balancer
The load balancer will proxy the connections to the back end servers.

## Cache
The  cache is in place to deliver the content from memory or fetch content from the origin server if the content has not yet been cached and load it into the cache according to the caching rules applied.
