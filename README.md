#AWS
This repo will be dedicated to AWS 

#AWS Services

1.Hosting a Static Website on AWS with S3, CloudFront and Route53 | Web | Hosting | AWS
  
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

2.Deploying React, Angular, Vue.js apps on AWS

Check angular version
- ng version

Create new angular project
- ng new angularApp --style=scss --routing

- Steps
    1. Create a user that has permission to upload build files to S3.
        - Navigate to IAM
        - Create a new user with Programmatic access(AWS API,CLI,SDK access) and Management console access
        - Best practice to give only the access that the user needs rather than giving all the access(List access, GetObject and PutObject access)
        - You need to download Access key Id, Secret Access Key and Password for that user
    2. Configure AWS Cli in machine and Cli with created user
        - aws configure --profile {name}
    3. Write a simple script to deploy to S3
        ~~~json
           {
             "deploy-s3" : "aws s3 --profile {name} sync ./dist/ s3://{bucket-name} --region us-east-1"
           }   
        ~~~
    4. Invalidating CloudFront cache 
        ~~~json
           {
             "cache-bust" : "aws cloudfront --profile {name} create-invalidation --distribution-id {dist-id} --paths '/*'"
           }   
        ~~~
    5. Make sure to add permission for the above user to interact with cloudfront as well
 
3.Developing an API backend - AWS Lambda - API Gateway - DynamoDB

![Diagram](resources/images/services-2.PNG "Diagram")
    
Serverless Framework
- npm install -g serverless
- serverless -v

Create a new service
- serverless create --template aws-nodejs --path my-service

~~~yml
service: my-service
provider:
    name: aws
    runtime: nodejs6.10
    profile: {username}
functions:
    hello:
        handler: handler.hello
        events:
            - http:
                path: users/hello
                method: get
~~~
   
Deploy a serverless app
- serverless deploy -v
    
Remove Serverless stack and everything provisioned
- serverless remove

- steps
    1. Need to create another path pattern that directs apis to API-Gateway
        - Navigate to cloudfront(copy api-gateway url)
        - Create another origin
            - domain as api-gateway url
        - Create another behaviour with path pattern as '/dev/*' 
 
 
4.Enabling gzip compression on S3 websites with Cloudfront

- steps
    1. If the website is being host through s3 bucket and cached through cloudfront we can inspect the site and check main.js bundle to see the 'Content-Type' of the served application which is 'application/javascript'
    2. We will enable compression that the website will load fast.
    3. Navigate to cloudfront
        - Click behaviour tab
        - Choose the default route that linked to s3 web application
        - Choose 'Compress Objects Automatically' and set it to 'Yes'
    4. Navigate to S3 and click the bucket
        - Select the Permission tab
        - Select CORS Configuration
        - Add below script, otherwise cloudfront will not know the size of the content 
        ~~~xml
       <AllowedHeader>Content-Length</AllowedHeader>
        ~~~
    5. We can enable compression inside S3 as well.
        - Navigate to properties of the object.
        - Click on Metadata
        - 'Content-Encoding' as 'gzip'
        - But if you are doing in this manner you need to upload files in gzip format
 
 
5.Building Web Apps with AWS Mobile Hub

- AWS Mobile hub supports iOS, Android, Web and React Native
- Create an app with preconfigured frontend and backend, set up your backend with AWS services configured(generates a cloud configuration file), connect your backend from our client.
- Features
    - Secure Authentication with AWS Cognito
    - Storage: S3
    - Serverless Functions: Lambda and API Gateway
    - Database: DynamoDB
    - Bots: Amazon Lex
    - Host web sites: CDN
    
- Steps
    1. Generate angular/react app
    2. Navigate to AWS Mobile Hub
    3. Setup AWS credentials
        - awsmobile configure
    4. Use below scripts
        - npm install -g awsmobile-cli
        - awsmobile --version
        - awsmobile init {token} - to link aws project with the generated app
    5. Push changes to aws mobile backend and run frontend locally
        - awsmobile run
    6. Any changes that are made from awsmobile hub side needs to be pulled back to client side using,
        - awsmobile pull
    7. Publish to s3 hosting bucket
        - awsmobile publish

6.User Authentication for Angular with AWS Cognito & Amplify

- Amazon Cognito Features
    - Secure and scalable user directory (Managed service)
    - Social and enterprise identity federation (Social: Google, Facebook, Amazon; Enterprise: Active Directory via SAML)
    - Supports Oauth2.0, SAML2.0, OpenID Connect
    - Fine grain access control
    
- Cognito Divides into
    - Cognito User Pools(for authentication: register users in user pool)
    - Cognito Federated Identities

![Diagram](resources/images/services-3.PNG "Diagram")
- Steps
    1. Authentication
        1. User logs in with Username and Password
        2. Cognito verifies that and if credentials are correct it sends back response(id_token, access_token, refresh_token)
        3. These are temporary tokens and saved in the browser
    2. Authorization
        1. Sends tokens to Cognito and get temporary credentials from Cognito Federated Identities using STS(Security Token Service)
        2. STS sends(access_key_id, secret_key) back to the user
        3. User can use these temporary credentials to access AWS resources
        4. AWS resources grabs the role out of the credentials and checks the IAM policies attached to that role

- AWS Amplify
    - npm install --save aws-amplify
    - npm install --save aws-amplify-angular

7.Building a Secure Application on AWS - Cognito - API Gateway - Dynamodb

- Security
![Diagram](resources/images/services-4.PNG "Diagram")

- Security Layer 1: Authentication Layer
- Security Layer 2: API Gateway Layer
    - For every API call jwt tokens that are stored in local storage will be sent as signed tokens
    - From signed tokens extract credentials and the IAM role.
    - Check against polices whether that IAM role is accessible to access APIs.
- Security Layer 3: DynamoDB Layer
    - Private : Users will not have any access to other users data
    - Protected: Users will read other users data and will not be allowed to update or delete each others items
    - Public: Users will read other users data and will be allowed to update or delete each others items
    - Partition key == users congnito id, so that the logged in user can only see his records from DynamoDB
    
Create API with Amazon API Gateway and Lambda
- awsmobile cloud-api enable --prompt

8.Building a S3 Directory for Your Company - S3 - IAM

- Steps
    1. Create a bucket for organization and create below directories
        - my-organization/Home/peter
        - my-organization/Home/ann
        - my-organization/Home/henry
    
        1. Any user can see all the other directories but cannot see the content
        2. Any user can only access his/her folder and can see the content
    2. Navigate to IAM 
        - Create a user 'Peter' with programmatic access and setup S3 browser with his credentials to S3 bucket
        - Attach below policy
            1. List all the buckets
            2. Get Bucket Locations
            3. Access to /Home
            4. Access to his/her bucket using IAM variables
            5. Allow all the actions
        ~~~json
        {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Effect": "Allow",
              "Action": "s3:ListAllMyBuckets",
              "Resource": "arn:aws:s3:::*"
            },
            {
              "Effect": "Allow",
              "Action": "s3:GetBucketLocation",
              "Resource": "arn:aws:s3:::*"
            },
            {
              "Effect": "Allow",
              "Action": "s3:ListBucket",
              "Resource": "arn:aws:s3::my-organization",
              "Condition": {
                "StringEquals": {
                  "s3:prefix": [
                    "",
                    "Home/"
                  ],
                  "s3:delimeter": [
                    "/"
                  ]
                }
              }
            },
            {
              "Effect": "Allow",
              "Action": "s3:ListBucket",
              "Resource": "arn:aws:s3::my-organization",
              "Condition": {
                "StringLike": {
                  "s3:prefix": [
                    "Home/${aws:username}/*"
                  ]
                }
              }
            },
            {
              "Effect": "Allow",
              "Action": "s3:*",
              "Resource": [
                "arn:aws:s3::my-organization/Home/${aws:username}",
                "arn:aws:s3::my-organization/Home/${aws:username/*}"
              ]
            }
          ]
        }
        ~~~
    3. Add unique files to each of the users S3 directories

9. Building a chatbot with Amazon Lex - AWS

![Diagram](resources/images/services-5.PNG "Diagram")

- Creating a conversational chatbot to retrieve weather information for users utterances using natural language.
- Steps
    1. Create an intent 'FindWeather'
        1. Add some sample utterances
            - "Tell me about the weather today?"
            - "How is the weather in {City}"
            - "What is the weather like in {City}"
        2. Create slots for variables
            - "Sure, Which city?" with {City} name
        3. Add Response message
            - "Temperature in {City} is 25C and Humidity level is 55%"
        4. Add AWS Lambda fulfillment
        5. Build the bot
    2. Create an intent 'GreetUser'
        1. Add some sample utterances
            - "Hi"
            - "Howdy"
            - "Hello"
        2. Create slots for variables
            - "Hi!! What is your {name}" with {Name} as name
        3. Add Response message
            - "Hello {Name}! How can I help you!"
            - "Hi {Name}! What do you want!"
        4. Build the bot
    3. Generate an angular application for weather app bot.
        - amplify init
        - amplify add auth
        - amplify push
        - Navigate to cognito and click 'Federated Identities' and find the role id
        - Navigate to IAM and attach 'AmazonLexFullAccess' policy
            
10. Containers on AWS
- Containers are an isolated resource section in our computer
- Will be talking about Docker with namespaces
- Volume mapping: It references original files rather than copying files to the container(e.g: Live reloading) 
- Steps
    1. Need to install Docker
        - docker --version
        - docker info
    2. Create Dockerfile
        ~~~.dockerfile
           FROM node:alpine
           WORKDIR '/app'
           COPY package.json .
           RUN npm install
           COPY . .
           EXPOSE 4200
           CMD npm run start
        ~~~
    3. Build the Dockerfile
        - docker build .
        - docker images
        - -p host_port: container_port
        - docker run -p 8080:4200
    4. Volume mapping with docker-compose.yml
        ~~~yml
            version: "3"
            services:
                myapp:
                    build: .
                    ports:
                        - "8080:4200"
                    volumes:
                        - "/app/node_modules"
                        - ".:/app"
        ~~~
    5. Run docker compose
        - docker-compose up

11. Deploying a containerized web application to Elastic Beanstalk

- Steps
    1. Navigate to Elastic Beanstalk(PAAS) - deploy, monitor and scale applications easily
    2. Create an Environment with docker
    3. Install EB Cli for relevant O/S
    4. Create multi step docker file
    ~~~.dockerfile
        FROM node:alpine as BUILD
        WORKDIR '/app'
        COPY package.json .
        RUN npm install
        COPY . .
        RUN npm run build
   
        FROM nginx
        EXPOSE 80
        COPY --from=BUILD /app/dist/myapp /usr/share/nginx/html
    ~~~
    5. Push to Elastic Beanstalk
        - docker run -p 80:80 {image_id}
        - eb --version
        - aws configure
        - eb init
        - Commit all the docker file changes before pushing to Elastic Beanstalk
        - eb create myapp
        - eb deploy
  
12 . Redirect URLs in AWS Using S3 and Cloud front
- eg: If we type any sub domains(www.mydomain.com or sub.mydomain.com) instead of Apex domain(mydomain.com) we need to redirect to apex domain
- Steps
    1. Needs to have an already created S3 bucket for apex domain
    2. Need to create a redirect bucket
    3. Select redirect bucket and navigate to properties and static website hosting
    4. Click 'Redirect requests'
        - Target bucket: mydomain.com
        - Protocol: https
    5. Navigate to CloudFront Distribution
        - Need to create another distribution for redirect bucket
        - Create SSL certificate to redirection bucket
        - For CNAME add 'www.mydomain.com'
    6. Navigate to route53
        - We need to create an A record for 'www.mydomain.com' DNS redirection with an ALIAS target for Cloudfront distribution created in step 5(domain name)
   
13 . How to Architect and Design Your Application on AWS Cloud    
- 5 Pillars of Well Architecture Application 
    1. Security
    2. Reliablilty: Ability for the system to recover from a certain disturbance, Horizontal scalability
    3. Performance Efficiency: High performance, hybrid architecture
    4. Cost Optimization
    5. Operational Excellence

- Steps
    1. Frontend
        - Angular Framework
        - AWS Amplify Library: Simple and Secure, make signed requests to backend resources
    2. Frontend Deployment
        - Host the website in S3 bucket: Reliability, 11 9s durability, cost effective, automatically scalable
        - Cloudfront in front of the S3 bucket: Performance efficiency, Caching, CDN
        - SSL certificate from ACM: Enable SSL, Connections will be encrypted
        - WAF on cloudfront: Security against common web attacks
        - Route53 as DNS
    3. Backend
        - Lambda to run business logic: Will scale automatically, serverless
        - API Gateway for API: Scales itself
        - DynamoDB as the database: NoSQL, Managed service, replicate data in different data center
    4. Search
        - Amazon ElasticSearch Service: Managed service, text based service, run in private network
        - AWS Lambda: Asynchronously index documents in ElasticSearch
    5. Authentication & Authorization
        - Amazon Cognito User Pool: Decouple users from application and they will serve in a highly scalable user pool
        - Amazon Cognito Identity Pool: IAM with Cognito users for more granular access
        
14 . Designing the AWS Project Architecture

![Diagram](resources/images/services-6.PNG "Diagram")

- Frontend
    - WebApplication resides in a S3 bucket
    - Website is cached through Cloudfront and delivered through CDN
    - On Cloudfront applied a certificate to enable SSL and HTTPS from Amazon Certificate Manager, Web Application Firewall rules as well
    - Cognito as Authentication and Authorization provider
    - If the users are successfully signed in they are routed to Cloudfront via Route53 using DNS
    - In Cloudfront WAF checks whether they are legitimate requests if not requests are discarded
    - If legit request then checks whether the cache is available otherwise talk to S3 bucket and get the latest content and delivered
 
- Security considerations
    - Lambda must not be accessible over the internet
        - Need to put lambda functions within a VPC
        - It is our responsibility to handle all the network related task
            - Design IP ranges
            - Divide them to sub networks
            - Create different security groups
        - Create a different sub network without an internet access - private subnet(Lambdas cannot be accessed over internet)
        - "A Call Start" time will be longer after we put it inside a VPC
        
    - Database, ElasticSearch must not be accessible over the internet
        - Both these services are regional services and called through public endpoint
        - Lambdas need to connect with these 2 service over internet
        - DynamoDB and ElasticSearch services provide VPC endpoints
        - Through that VPC endpoints lambdas can communicate with these 2 services
        - We need to give access from dynamodb to lambda and vice-versa(for that we need to apply a security group in both places)
        
15 . Hosting a web application with AWS Amplify

- Steps
    1. Create an angular app
    2. Incorporate Amplify with the angular app
        - amplify configure and give programmatic access to AdministratorAccess policy
    3. Initialize Backend
        - amplify init
        - amplify will be using cloudformation stack to create the backend 
        - amplify hosting add
        - amplify publish
 
16 . Authentication & Authorization with AWS Amplify

- Steps
    1. amplify auth add
    2. Add amplify related changes in angular frontend side
   
    
17 . Single Sign On (SSO) with Facebook on AWS Cognito
- We'll be using Amazon Cognito Hosted UI for this
- Steps
    1. Navigate to Cognito
        - Navigate to the correct cognito user pool
        - Click 'Domain name' under 'App integration'
        - Check availability for a domain
        - Add social login with facebook as well
    2. SSO
        - When user logins through cognito authentication happens in Cognito user pool
        - Cognito checks whether the user is available in the pool and sends back access token, id token and refresh token
        - If the user wants to authenticate with facebook, cognito redirects it to facebook
        - After successful authentication facebook sends tokens/code to cognito user pool
        - Cognito checks tokens/code and create a record inside cognito pool and generates its own access, id and refresh token to the user
        - For any Identity provider we add, cognito will make sure for any authorized requests through these Identity providers it will create its own access, id, refresh tokens, so that the backend or frontend will rely on same set of tokens
 
18 . Authentication with Cognito Hosted UI
- Steps
    1. Will go through amazon cognito authentication documentation for configuring Hosted UI
        - Create an object with configuration
            - Domain, scope, redirectSignIn, redirectSingOut, responseType, AdvancedSecurityDataCollectionFlag
        

#AWS Theories

1 . What is Cloud Computing
- Cloud computing is the on demand delivery of compute power, database storage, applications and other IT resources through a cloud services platform via the internet with pay as you go model.

- Vertical Scalability(Scaling UP)
~~~txt
1GB RAM                     Increase of traffic  16GB RAM 
             --------------------------------->
500GB HDD   <---------------------------------   1TB HDD
                            Decrease of traffic                                
~~~
- Horizontal Scalability(Scaling OUT)
~~~txt
1GB RAM                     Increase of traffic  1GB RAM    1GB RAM    1GB RAM
             --------------------------------->
500GB HDD    <---------------------------------  500GB HDD  500GB HDD  500GB HDD
                            Decrease of traffic                
~~~
- Auto Scaling
~~~txt
If CPU > 70% -------------------------> Spin 3 more servers
If CPU < 50% -------------------------> Decrease to 1 server  
~~~

- Cloud Computing
    - Converts Capex to Opex(Capital Expenditures to Operational Expenditures)
    - On demand
    - Pay as you go
    - Elastic services
    - Managed via internet
    - Fast deployment
    