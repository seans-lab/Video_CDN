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

## Load Balancer
The load balancer will proxy the connections to the back end servers.

## Cache
The  cache is in place to deliver the content from memory or fetch content from the origin server if the content has not yet been cached and load it into the cache according to the caching rules applied.
