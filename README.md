# Video_CDN

NGINX has the ability to deliver Live and VOD streams across an enterprise network or as a global CDN with the following configurations.

# Redirector
The purpose of the redirector is to provide a central point to resolve requests for the video stream, live or on-demand, and redirect the session to the server allocated to serve content to the users IP range.

# Load Balancer
The load balancer will proxy the connections to the back end servers.

# Cache
The  cache is in place to deliver the content from memory or fetch content from the origin server if the content has not yet been cached and load it into the cache according to the caching rules applied.
