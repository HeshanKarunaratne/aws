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
        
19 . DynamoDB, API Gateway and Amplify

- In this we'll be calling some API endpoints in API gateway which talks to a Dynamodb database using a lambda function
- VPC Endpoint will be added in upcoming session
- Steps
    1. Adding the API backend
        - amplify add api
    2. All the APIs will be generated for the given API endpoint with authentication and all the validation
    3. Check dynamodb for newly created table and API Gateway console for newly created API and endpoints
    4. Create home component
        - ng g c home
    5. Using Dynamodb streams to index records from dynamodb to ElasticSearch

20 . Connecting Azure Active Directory with Cognito
- How to setup Azure Active Directory as a 3rd party Identity provider for the application
- Steps
    1. Create a user pool and login with Cognito Hosted UI
        - Add an app client
        - Make a note of client ID
        - Check for a domain prefix
        - Copy the url format for 'Using the Hosted Domain'
        - Setup the redirect url
    2. Azure AD over SAML protocol
        - Need to provide metadata url and identifiers
        - Need to provide a user pool in AD
        - Need to provide configurations from cognito 
            1. Sign-On url
            2. Identifier uri
            3. Logout url
            4. Reply url
        - Update the manifest with above values
        - Need to get Azure metadata document and apply to cognito side
        - Add attribute mappings
    3. When user types corporate email they will be redirected to Azure AD
    4. Authenticate against AD
    5. If success redirect to application

21 . Working with DynamoDB and Elasticsearch
![Diagram](resources/images/services-7.PNG "Diagram")
 - Steps
    1. When a product is added to dynamodb stream lambda function will trigger
        - Need to enable dynamoDB streams in dynamoDB
    2. It will index the record in ElasticSearch
    3. This is an asynchronous operation
    4. Dynamodb streams and lambdas are managed services
    5. Need to create an ElasticSearch service inside the private subnet of VPC
    6. Create a lambda function with an IAM role so that it can communicate with ElasticSearch service
        ~~~json
        {
          "Version": "2012-10-17",
          "Statement": [
                  {
                    "Effect":"Allow",
                    "Action":[
                      "es:ESHttpPost",
                      "es:ESHttpPut",
                      "dynamodb:DescribeStream",
                      "dynamodb:GetRecords",
                      "dynamodb:GetShardIterator",
                      "dynamodb:ListStreams",
                      "logs:CreateLogGroup",
                      "logs:CreateLogStream",
                      "logs:PutLogEvents"
                    ],
                    "Resource": "*"
                  }
          ]
        }
        ~~~
    7. Create a security group in ElasticSearch domain and apply the same in lambda so that communication can happen

22 . GraphQL in AWS with AppSync, Amplify and Angular 
- GraphQL
    - Fast and flexible compared to REST
    - GraphQL client decides what data to receive
    - Operate over HTTP
    - Fewer HTTP requests
    - Flexible data querying and less code to maintain    

- Operations
    - Query: Fetch the data from GraphQL - Get
    - Mutation: Change data - Create/Update/Delete
    - Subscription: Watch data for changes in Real time
        - Happens through web sockets

23 . Modeling Relationships in GraphQL
- Relationships
    - 1:1: Department has a Manager
    - 1:M: Department has many employees
    - M:N: Project has many Employees and Employees can be assigned to many projects

- Steps
    1. 'amplify api add' and choose graphql
        ~~~graphql
            type Department @model{
                id: ID!
                name: String 
                manager: Employee @connection
                employees: [Employee] @connection(name: "DeparmtentEmployees")
            }
            type Employee @model{
                id: ID!
                name: String 
                age: Int
                department: Department @connection(name: "DeparmtentEmployees")
                projects: [EmployeeProjects] @connection(name: "EmployeeProjects")
            }
            type EmployeeProjects @model (queries: null){
                id: ID!
                employee: Employee @connection(name: "EmployeeProjects")
                project: Project @connection(name: "ProjectEmployees")
            }
            type Project @model{
                id: ID!
                name: String 
                employees: [EmployeeProjects] @connection(name: "ProjectEmployees")
            }
        ~~~
    2. 'amplify api gql-compile' will recompile the graphql stack in appSync

24 . How to send realtime updates with GraphQL subscriptions
- AppSync is a managed graphQL server
- Subscription are watching over changes in graphQL
- Steps
    1. When have a subscription it will send all the changes to all the clients who are listening


25 . 

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
    
2 . Types of Cloud Computing

- Virtualization
    - Ability to host multiple clients on the same infrastructure while maintaining isolation among each other
    
- Types of Cloud
    - IAAS: Infrastructure As A Service
        - All the Infrastructure related changes will be managed by the user(Installation and patches)
    - PAAS: Platform As A Service
        - Infrastructure managed by AWS, we only need to focus on code and other stuff
        - eg: Elastic Beanstalk
    - SAAS: Software As A Service
        - Zero administration
    
3 . AWS VPC - Networking on AWS
- On Prem Architecture
    ![Diagram](resources/images/theory-1.PNG "Diagram")
    - Explanation
        - IP Range is divided to 2 sub-networks
        - Web servers are in 1 sub-network and it is protected by a firewall and users can directly talk to the web server
        - Application server and database server can only be accessed through web servers only
        
- AWS Architecture
    ![Diagram](resources/images/theory-2.PNG "Diagram")
    - Explanation
        - When we spin up an EC2 server if we don't specify a network or a VPC it will be assigned to the default VPC
        - Virtual Private Cloud(VPC) we have full control over network, security, resources and it is isolated from other AWS customers
        - VPC is only associated with 1 region in AWS(Cannot have a VPC spanning over different regions) and we need to define an IP range for VPC
        - IRC 1918 standard
            1. 10.0.0.0/16
            2. 172.16.0.0/16
            3. 192.168.0.0/16
            
        - CIDR Notation
            1. eg: 10.0.0.0/16 
                - Total IP Count: 2^(32-16)=65536
            2. eg: 192.168.0.0/24 
                - Total IP Count: 2^(32-24)=256
            - AWS will reserve 5 addresses from these(first 4 and last address)
            - Maximum cider that AWS supports is /16 and minimum is /28
        - Can create multiple sub-networks inside a VPC and it corresponds to an Availability Zone
        - Subnetwork IP ranges cannot overlap
        - VPC Router allows communication between sub-networks
            - eg: Destination: 10.0.0.0/16 Target: local
            - Web server can communicate to Backend server and vise versa if the ip range falls into the Destination
        - We need internet access for web server(Need to setup an internet gateway for subnet1)
        - Internet gateway is a managed service and you dont find any single point of failure
        - Create another route entry in the route table associated for subnet1 that points out to internet gateway
            - eg: 0.0.0.0/0 internet-gatewat-id
        - If any subnetwork doesn't have an internet gateway associated with or any routing rule for internet gateway that subnetwork is a Private subnetwork
        - Application server and the database server needs to run updates for that we can setup a NAT Gateway in public subnet
        - We can add a route to the NAT gateway which points to 0.0.0.0/0
        
4 . Security Groups and NACL in VPC
- Steps
    1. We need to define firewall rules so that who can access which resources on which ports
    ![Diagram](resources/images/theory-3.PNG "Diagram")
        - Security Groups(Firewall)
            1. EC2 server communicates with the subnet via ENI(Elastic Network Interface) - virtual network interface card attached to a virtual server
            2. Security Groups works at ENI level(Instance level)
            3. Allow or block traffic in and out of EC2 instance
            4. EC2 instance can have upto 5 security groups
            5. Stateful
            6. Can have only 'Allow' rules
            7. If no rule is defined, traffic is blocked to the instance
        - NACLs(Network Access Control List)
            1. NACL works at subnet level
            2. Stateless
            3. Both 'Allow' and 'Deny' rules are possible
            4. Rules are evaluated from Lowest to Highest
            5. Lower the rule number, higher the priority
            6. Rules are applied to all the instances within the subnet
            7. Default NACL -> traffic is allowed for both inbound and outbound
    ![Diagram](resources/images/theory-4.PNG "Diagram")    
     
     
5 . Amazon Virtual Private Cloud (VPC)   
    
![Diagram](resources/images/theory-5.PNG "Diagram")

- Steps
    1. Spin up a VPC in a selected region
        - For every region there is a default VPC
        - Give a cider block for VPC eg: 10.0.0.0/16
        - You will get a Main Route table, Main Network ACL and DHCP options set
    2. Allocate subnetworks
        - Private subnet with cider block of 10.0.0.0/24
        - 1 subnet should have an availability zone
        - Public subnet with cider block of 10.0.1.0/24
    3. Check default NACL created with the VPC
        - It is associated with 2 subnetworks
        - Inbound rule and Outbound rule
            1. ALL Traffic ALLOW for any source(0.0.0.0/0) for all the port ranges and for all the protocols
    4. Check default Route table associated with the VPC
        - Click Subnet Associations and edit and add public subnet
    5. Create an Internet Gateway
        - Attach the Internet Gateway to the VPC
        - For public route table add the internet gateway for all 0.0.0.0/0 destination
        - Create a private route table but dont attach the internet gateway because private subnet should not be directly accessed through internet
    
6 . Security Groups and Launching EC2 Instances
- Steps
    1. Launch EC2 instances in both private and public subnet
        - WebServer
            - Enable http, https and ssh(only my ip) for the 'webserver' instance that resides in public subnet
            - Add public ip while configuring so that the instance can be accessed via internet
            - When you restart the instance public ip address will be changed to a different address from aws ipv4 pool
            - If we want to mitigate earlier issue we need an elastic IP
            - Permission Levels
                - chmod 777(owner, group, public)
                    - 1: execute
                    - 2: write
                    - 4: read
        - Backend
            - Do not add public ip while configuring
            - To talk from web-server to backend allow access for 'web-server-security-group' as the source
            - Still we cannot ssh into backend server from web-server bacause we do not have the pem file there
            - Its not a security best practice to copy private .pem file to webserver
            - Best practice is to use SSH agent forwarding

7 . SSH Agent Forwarding - Connecting to EC2 Instances
- Steps
    1. Navigate to local machine
        - For windows use 'putty' and for MAC use terminal
        - Mac
            - Add .pem file to ssh agent
                - eg: ssh-add -K my-private-key.pem
            - Connect to web server
                - eg: ssh -A ec2-user@35.123.48.34
            - After you connect to webserver ssh to database server using the forwarded ssh agent 
    - Even though we connected to database server we cannot communicate with the internet(NAT gateway or Instance)
    
    
# Exam Guides
- Define solutions using architectural design principals based on customer requirements
- Provide implementations based on best practices to the organization throughout the lifecycle of the project