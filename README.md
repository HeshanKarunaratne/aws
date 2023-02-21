#AWS
This repo will be dedicated to AWS 

#AWS Services

1. Hosting a Static Website on AWS with S3, CloudFront and Route53 | Web | Hosting | AWS
  
![Diagram](resources/images/services-1.PNG "Diagram")

- Below service will be used
    - S3: Upload website code
    - CloudFront: CDN to cache request
    - Route53: Act as a DNS to redirect request to Cloudfront
    - Certificate Manager: Apply the certificate to CloudFront so that it would support SSL
    
- Steps
    1. Get a domain. 
    2. Create a hosted zone in route53 and copy namespace values to domain nameservers.
    3. Create a S3 bucket with domain name and configure it as a static website hosting.
    4. Add a bucket policy so that it can be accessed.
        ~~~json
            {
              "Version":"2012-10-17",
              "Statement":[
                {
                  "Sid":"AddPerm",
                  "Effect":"Allow",
                  "Principal": "*",
                  "Action":["s3:GetObject"],
                  "Resource":["arn:aws:s3:::{bucket_name}/*"]
                  }
              ]
            }
        ~~~
    5. Now you should be able to access the site from S3, but we need to cache our site through cloudfront
    6. Create a cloudfront distribution and under the origin domain name paste S3 website url without http://
    7. Update domain name as Alternate domain name in cloudfront.
    8. Acquire a certificate and apply to cloudfront(ACM).
    9. You need to apply the CNAME given by certificate manager to route53
    10. You need to configure route53 to cloudfront using an A record with alias target to cloudfront domain name.
    
    
#AWS Theories