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
problem. Using the host header the browser tell the Application/Layer-7 which application to access

### Origin Access Identity (OAI)

This identity can be associated with a cloud front distribution.

The edge locations gain this identity.

We then remove the explicit allows and only allow the OAI to use it.

Best practice is to create one per distribution to manage permissions.

### AWS Global Accelerator

Starts with 2 **anycast** IP address
1.2.3.4 & 4.3.2.1

Anycast IP's allow a single IP to be in multiple locations.
Routing moves traffic to closest location.

Traffic initially uses public internet and enters a global
accelerator edge location.

#### Key concepts

Move the AWS network closer to customers.

Global Accelerator moves the AWS network as close as possible.

Connections enter at edge, using anycast IPs. Transit over AWS backbone to 1+
locations.

Global Accelerator is a network product. Can use TCP, UDP.

Caching is mostly CloudFront. TCP and UDP is mostly Global Accelerator
