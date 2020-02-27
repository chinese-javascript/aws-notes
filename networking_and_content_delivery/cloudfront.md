# CloudFront
CloudFront is Amazon's CDN service that uses **Edge Locations** around the world to distribute cached content to users. Requests for your content are automatically routed to the nearest Edge Location to ensure best possible performance. CloudFront is also used by S3 Transfer Acceleration to reduce latency for S3 uploads.

Objects on CloudFront are cached based on the TTL. You can manually **invalidate** (clear and refresh) cached objects for an extra charge.

Whenever there is new content on a CloudFront distribution, the first user request will have a longer response time as the content is not cached yet. Subsequent requests to the same content will be much quicker.

## Terminology
* **Edge Location**: The location where content is cached to be accessed by users. These are READ/WRITE.
* **CDN**: A collection of Edge Locations that can distribute content around the world.
* **Origin**: The origin of all files the CDN will distribute. E.g. 
  * an S3 bucket hosting some images, or hosting a static website
  * an EC2 instance running a website with dynamic content
  * an ELB pointing to several EC2 instances
  * a DNS endpoint using Route53
  * any origin server, even non-AWS
* **Distribution**: The name of the CDN.
  * **Web Distribution**: Used for delivering content over HTTP/HTTPS. Can be either an S3 bucket or a web server (EC2/non-AWS). Cannot serve multimedia content.
  * **RTMP Distrubtion**: Uses RTMP for media streaming and flash multimedia content. Probably what Netflix uses.

## Distribution Settings
* **Restrict User Access**: Require signed URLs or cookies to access cached content e.g. when distributing paid content.

# Lambda@Edge
Lambda@Edge is an extension of AWS Lambda, and lets you execute functions that customize the content that CloudFront delivers. When you associate a CloudFront distribution with a Lambda@Edge function, CloudFront intercepts requests and responses at CloudFront edge locations. You can execute Lambda functions when the following CloudFront events occur:

* When CloudFront receives a request from a viewer (viewer request)
* Before CloudFront forwards a request to the origin (origin request)
* When CloudFront receives a response from the origin (origin response)
* Before CloudFront returns the response to the viewer (viewer response)

![](https://docs.aws.amazon.com/lambda/latest/dg/images/cloudfront-events-that-trigger-lambda-functions.png)

Lambda@Edge use cases include:

* A Lambda function can inspect cookies and rewrite URLs so that users see different versions of a site for A/B testing.
* CloudFront can return different objects to viewers based on the device they're using by checking the `User-Agent` header.
* A function can inspect headers or authorization tokens, and insert a header to control access to your content before CloudFront forwards the request to your origin.
