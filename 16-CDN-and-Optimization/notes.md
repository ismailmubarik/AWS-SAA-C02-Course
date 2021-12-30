## CloudFront

### Architecture Basics
![image](https://user-images.githubusercontent.com/33827177/147512595-058f8296-79a6-42fa-a8fc-f4a4b5314669.png)

Cloudfront is a global object cache, also known as a content delivery network (CDN)

Content is cached in locations close to customers.
If the content is not available on the local cache when requested, ClouldFront
will pull and cache it then.

This provides lower latency and higher throughput for customers.

Can handle static and dynamic content.

**Origin**  the source location of your content. It can be an S3 origin 
(S3 bucket running a static website) or a Custom Origin (which runs a webserver
& publicly routable IPv4 address).

**Distribution** the configuration unit of CloudFront

Edge locations - local infrastructure which hosts a cache of your data.
There are over 200 edge locations. Edge location is smaller than an AWS region.
They can be one or more racks in a third party data center and they are usually
90% storage with odd bit of compute services. So you can't use an edge location 
for EC2 instance.

Regional Edge Cache - larger version of an edge location. Provides
another layer of caching.
![image](https://user-images.githubusercontent.com/33827177/147513886-e4d9021b-0f7e-47ff-bf58-ec557a2594d0.png)

### CloudFront Behaviours
A single distribution can have multiple behaviors. There is also going to be a default behaviour.
The way behaviors work is that any requests incoming from edge locations are pattern matched against
any behaviors using a path pattern. The default behavior has a wild-card (*) so it would match against anything
which is not matched by any more specific behavior. The default is at Precedence level 0 meaning lowest priority
![image](https://user-images.githubusercontent.com/33827177/147514464-d8a5ea2f-c368-405a-8a6f-603d48578a78.png)

![image](https://user-images.githubusercontent.com/33827177/147514498-ef99141f-e60a-4ac2-9c09-73a6f29081d5.png)

For exam remember that:
All caching controls are set on the behvior as well as the Restrict Viewer Access
![image](https://user-images.githubusercontent.com/33827177/147514628-21617978-5f15-4cea-b59c-070dbe9a718c.png)

### TTL and Cache Invalidations
![image](https://user-images.githubusercontent.com/33827177/147515599-2242f897-1909-4c4f-b44c-76c3e3469860.png)
The old picture is forwarded to the user even though the image has been changed and the old image is treated as a valid one
![image](https://user-images.githubusercontent.com/33827177/147515674-746f9849-88b2-43d1-8f84-5ee8e79296c2.png)
After the Object Expires the request is forward to the S3 bucket for a new version of image. If it has not changed a '304 Not
Modified' is returned. 
![image](https://user-images.githubusercontent.com/33827177/147515801-94193d59-b7b7-4ae4-973a-a7090845c78e.png)
If a newer version of image has been uploaded a '200 OK' is returned

Exam PowerUp:
If the Min. TTL and Max TTL (both Min and Max are attached to the Behavior)
is not used then the default TTL value of 24 HRs attached to the Behavior is used
The origin which can be S3 bucket or a custom origin with a WebServer can both direct 
CloudFront to use object specific TTL values using Headers
![image](https://user-images.githubusercontent.com/33827177/147516177-c37ac70b-ee1a-4f00-8bda-5d0f34743c63.png)
The Min. and Max. TTL are going to still acts as limiters. So values defined in headers below the Min. will mean that
the Min. value be used instead. Same goes for Max TTL.

Cache Invalidation costs the same irrespective of the number of objects you are trying to invalidate and so if you have to
invalidate quite frequently use Versioned File Names instead. Versioned File Names are better for a few reasons caz it involves 
changing the names of the files so the old version of file cached would not be used because the object being pointed it is a different file 
with a different name. Second logging is used so you know which version was used when. Lastly, it is not expensive caz you dont do Object Invalidations

Versioned File Names is different than S3 Object versioning. S3 Object Versioning allows you to use the same name for different objects/data

#### Caching Optimisation

Parameters can be passed on the url such as query string parameter.
An example is `?language=en` and `?language=es`

To get the cached copy, you need to use the same query strings. If you do
use them, ensure the strings are in the same order.

### AWS Certificate Manager (ACM)

- HTTP lacks encryption and is insecure
- HTTPS uses SSL/TLS layer of encryption added to HTTP
- Data is encrypted in-transit
- Certificates allow servers to prove their identity
- Signed by a trusted authority (CA i.e. Certificate Authority). So if a site claiming to be Netflix.com has a Netflix.com certificate signed by the CA then it is Netflix.com
- To be secure, a website generates a certificate and has a CA sign it. The website then uses that certificate to prove its authenticity.

AWS Certificate Manager (ACM) allows to Create, Renew, and Deploy certificates to supported AWS services

Supported AWS services ONLY (CloudFront and ALB, NOT EC2)

If it's not a managed service, ACM doesn't support it. EC2 is self-managed so ACM doesn't support it. So you can't use ACM and deploy a certificate
to an EC2 based webserver but you can deploy a certificate to the Application Load Balancer (ALB) which load-balancing across those EC2 insances

Cloudfront must have a trusted and signed certificate. It can't be self signed.
![image](https://user-images.githubusercontent.com/33827177/147518084-f8a697d0-4d7b-4843-a38e-25fb3c9afcaf.png)

### CloudFront & SSL
Each CloudFront receives a Default Domain Name (CNAME).SSL certificate is also provided by default. 
![image](https://user-images.githubusercontent.com/33827177/147519247-42e00184-8f5c-466f-a256-ce03f962d703.png)
If you want to use your customer Domain Name, it is done via the alternate domain name feature. You can then use
a DNS provider like Route 53 to point at the distribution (with the custom domain name). If you use HTTPs with your
distribution which has the custom/alternate domain name or even if you dont use HTTPs it needs to prove its identity and that is done
by adding a certificate matching the custom/alternate Domain Name to verify the identity. FOr that you need to generate or import a certificate
using AWS ACM. This is a regional service and normally you need to add certificate in the same region in which the service is being used. Exceptions to
this are global services like AWS CloudFront. For CF the certificate will always be in us-east-1.

There are 2 sets of connections when any individual is using CF.
![image](https://user-images.githubusercontent.com/33827177/147519206-7e3cce44-7107-4ef6-bbf2-2a906b9f0ac1.png)
Self signed certificate won't work. Both will need public certificates

![image](https://user-images.githubusercontent.com/33827177/147519697-a62e9046-5583-4b89-ace7-035754f7927c.png)
So SSL and TLS are often used interchangeably but in this context we mean encryption that occurs over a network connection.

Now encryption occurs at TCP layer which is at a much lower layer than the HTTP which is an Application layer protocol.
![image](https://user-images.githubusercontent.com/33827177/147520091-203eb46d-0bda-4e48-bd10-8f9929b204f3.png)

Now a single webserver many website using different names a single IP address. Now if we are using only HTTP there is no
problem. Using the host header the browser tell the Application/Layer-7 which application to access. But this happens at the
application layer after the connection has been established. But TLS the encrypted part of HTTPs happens before this layer i.e.
Layer-7/Application. So before the browser tells the server which application it wants. Part of what TLS does is allow a webserver
to validate its indentity. When you first create an encrypted connections between your browser and the webserver/ip-address, the webserver 
identifies itself. But if you have no way to tell the webserver which application you want to use out of the say 2 it host then it won't be 
able to figure out which application service to serve your browser. So the server can only provide one certificate. This was originally the case and
hence only one site could be hosted and each site/application needed an invidual IP.
![image](https://user-images.githubusercontent.com/33827177/147520804-6ba2ef3d-b2da-437c-9a25-1229ac0c8f94.png)

![image](https://user-images.githubusercontent.com/33827177/147521472-ce1d25ee-4079-40b0-93bf-7a29d2cd0bff.png)
In all cases of origin, the certificate needs to match the DNS name of the origin. Similarly, the certificate for the
CloudFront needs to match the DNS Name

### Origin Types & Architecture
Architecturally, origin is from where CloudFront gets data. That is not cached at the edge, an Origin Fetch occurs to cache the data from the origin on the
edge.

Origin Groups Allows you to have resiliency. If you have 2 or more origins created within a distribution. You can add them both to an Origin Group. And get the
Origin Group used by a CloudFront Behavior. So any request with a specific Path pattern will provide resilience across the Origin Group
![image](https://user-images.githubusercontent.com/33827177/147522462-27f42415-b7b4-4af5-96ad-a337139f1a2f.png)

There are few categories of Origin:
1. AWS S3 Buckets: Important to remember that S3 bucket has 1 set of features but if you configure a static website and use that as an origin then CloudFront views as a webserver and so a custom origin and hence different feature set is availabe to it
2. AWS Media Package Endpoints
3. AWS Media Store Container Endpoints
4. Everything Else (Webservers): Has different restrictions and features vs S3 buckets.

OAI only works for S3 origins. 

Whichever protocol is used between a customer and the edge location. So the viewer protocol policy is also used between the CloudFront and the S3 bucket. So the origin and viewer policy are matched and http/https will be used on the origin and viewer side.
![image](https://user-images.githubusercontent.com/33827177/147523448-40121eb1-5b5b-44d7-827d-4f263abcb9a0.png)

If you are not using S3 origins you cannot use Origin Access Identity and to secure you custome origins you can use Origin Custom Headers that only you are aware of and pass them to Custom Origin to check for those headers. This will allow the your custom Origin to only accept connections from CloudFront.

### Securing S3 Origin via Origin Access Identity (OAI)
You can use S3 as an origin for CloudFront in 2 ways: 1. As an S3 origin and use it with CloudFront2. As a static website using S3 use it with CloudFront(in which case it will be treated like a custom origin). OAI is applicable for S3 origin.

This identity can be associated with a cloud front distribution.

The edge locations gain this identity.

We then remove the explicit allows and only allow the OAI to use it.

Best practice is to create one per distribution to manage permissions.

![image](https://user-images.githubusercontent.com/33827177/147709152-b1de65a3-362f-423c-8c95-b4bac183e0c4.png)

![image](https://user-images.githubusercontent.com/33827177/147709272-db323e79-f307-44cd-be12-2da38ae8f3fc.png)

### Securing Custom Origin
Two ways to handle:
1. gin configured to required custom header. Custom headers are injected at the edge locations so the custom origin knows the request is coming from a legit source. If the custom header isn't present the custom origin will refuse to serve the request.

![image](https://user-images.githubusercontent.com/33827177/147709627-3d572f1c-986c-4bf1-b4ab-6a9d24896fd8.png)

2. Via Traditinal Security Methods: AWS publicizes IPs of all of their services so we can easily determine the IPs of the CloudFront edge locations.

![image](https://user-images.githubusercontent.com/33827177/147709769-607eaf5e-fbcf-4bc1-a79d-24c95ee3cab9.png)

Either or both of these methods can be used to ensure that security cannot be bypassed by a user through direct access to the origin

### Lambda@Edge
![image](https://user-images.githubusercontent.com/33827177/147710242-3739f233-4f7b-4864-9c4b-8cb5cfbdad44.png)

![image](https://user-images.githubusercontent.com/33827177/147710319-9a21390a-2d27-4360-bb41-b2e0da816790.png)

![image](https://user-images.githubusercontent.com/33827177/147710421-0fd33211-2880-4d2c-be9e-a05d603783f7.png)

https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-examples.html#lambda-examples-redirecting-examples

### AWS Global Accelerator
AWS Global Accelerator is a product that is designed to optimize the flow of data from your users to your AWS infrastructure

![image](https://user-images.githubusercontent.com/33827177/147711348-4e913638-5266-4e4d-985e-a138abb7380f.png)

Starts with 2 **anycast** IP address
1.2.3.4 & 4.3.2.1

Anycast IP's allow a single IP to be in multiple locations.
Routing moves traffic to closest GLobal accelerator Edge Locations. 
The key to remember is that all of these global accelerator edge locations 
are using these 2 **anycast** IP address of 1.2.3.4 & 4.3.2.1

![image](https://user-images.githubusercontent.com/33827177/147711528-d344b6fe-40c8-4043-b0d3-8eeb1bd87492.png)


Traffic initially uses public internet and enters a global
accelerator edge location.

Difference between CloudFront & AWS Global Accelerator:
CF move the content closer to the user by caching at the CF edge locations whereas AWS Global Accelerator moves the actual AWS network as close to the user as possible. It can route traffic through a dedicated connection to the original location or to the nearest infrastructure location.

![image](https://user-images.githubusercontent.com/33827177/147711744-3228fa10-1207-4a77-894b-842279202394.png)

Where & Where to use it:
Its a network product so scenarios in which we want to transport network data TCP/UDP then use AWS Global Accelerator. If its content, caching, presigned URL, etc. then use CF

#### Key concepts

Move the AWS network closer to customers.

Global Accelerator moves the AWS network as close as possible.

Connections enter at edge, using anycast IPs. Transit over AWS backbone to 1+
locations.

Global Accelerator is a network product. Can use TCP, UDP.

Caching is mostly CloudFront. TCP and UDP is mostly Global Accelerator
