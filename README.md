# Video_CDN

NGINX has the ability to deliver Live and VOD streams across an enterprise network or as a global CDN with the following configurations.

# Infrastructure
In order to achieve the delivery of live and VOD content from cloud service providers, it is recommened to use Split DNS to resolve the External DNS name of the service to internal servers. This will allow for there to be no changes made to the service configurations of the incoming applications content.

## Cloud Service Providers

# Redirector
The purpose of the redirector is to provide a central point to resolve requests for the video stream, live or on-demand, and redirect the session to the server allocated to serve content to the users IP range.

# Load Balancer
The load balancer will proxy the connections to the back end servers.

# Cache
The  cache is in place to deliver the content from memory or fetch content from the origin server if the content has not yet been cached and load it into the cache according to the caching rules applied.
