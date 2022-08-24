# CloudFront

CloudFront is a `Content Delivery Network` (CDN) that distributes static resources from edge locations around the world. CloudFront reduces impact on backend services and serves content faster to global locations for `better user experience`.

A CloudFront `origin` is the source location of your content. Origins can either be an `S3 Origin` or a `custom origin`.

A CloudFront `distribution` is the configuration unit of CloudFront.

`Edge locations` are the global points-of-presense (POP) around the globe in which your data is cached locally. Edge locations are responsible for serving content to users. A `regional edge cache` is a larger version of an edge location that provides another layer of caching.

1. When a user makes a request, it is routed to the closest `edge location`. If that edge location has the requested data cached locally, it is served to the user.
2. **Custom Origin** - If the edge location does not have the data cached locally, a request is made to the closest `regional edge cache` (custom origins only). If the data is cached on the regional edge, it is returned to the edge location, cached, and served to the user.
2. **S3 Origin** - CloudFront distributions using an S3 origin do not use regional edge locations.
3. If the regional edge location does not contain the data, it is fetched from the `origin`. The resulting data is cached on the regional edge cache and edge location, then served to the user.

CloudFront caches read requests only. It does not support **write caching**.

CloudFront integrates with ACM to enable **custom domains**.

![CloudFront](../static/images/cloudfront.png)

## Behaviors

A CloudFront distribution can have one or more `cache behaviors`. 

CloudFront distributions are not linked to origins directly. Instead, an origin is defined by a behavior and a distribution is associated with one or more cache behaviors.

A cache behavior defines an origin a request pattern that identifies requests to route to the origin.

![CloudFront - Cache Behaviors](../static/images/cloudfront_behavior.png)