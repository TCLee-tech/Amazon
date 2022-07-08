## Iterative App Modernization Workshop - monolith to microservices architecture

Lab guide: https://catalog.us-east-1.prod.workshops.aws/workshops/f2c0706c-7192-495f-853c-fd3341db265a/en-US/monolith-app/backend-deployment

### Scenario: 
Unishop sells Unicorn online (e-commerce). <br>
Shopping cart is going to be refactored out as a microservice. Shopping cart needs to be highly available, durable and scalable. For example, on Boxing Day and Black Friday surges.

### Architecture:
#### Monolith
Monolith backend is Java Spring Boot application deployed on EC2 instance, connected to RDS MySQL database. <br>
Network is single VPC with 2 subnets in 2 availability zones. 1 for the EC2, 1 for the RDS database. <br>
Frontend written with bootstrap, pushed to S3 for static web hosting. <br>
Amazon API Gateway deployed to proxy traffic from frontend to backend.

##### The primary controllers:
<ul>
<li>CoreController
<li>BasketController: basket management
<li>UnicornController: inventory management
<li>UserController: user management, registration, login
<li>HealthController: performs basic health checks
</ul>
	
##### Database 
made up of 3 tables:
<ol>
<li>unicorns: holds the inventory of Unicorns
<li>unicorn_basket: an association table between the Unicorns and the user's selection
<li>unicorn_user: represents the users in the system
</ol>
	
#### Microservices
To-be microservice Compute is a basket of 3 serverless AWS Lambda functions.<br>
 Shopping cart functionality broken into 3 distinct microservices, handling 3 business logics
- add unicorn to chart
- remove unicorn from cart
- get cart of unicorn/s <br>

Lambda functions connected to NoSQL database Amazon DynamoDB (fast key/value store, fast read/write). <br>
API Gateway will route traffic to microservice based on HTTP method. <br>
	<ul> 
		<li>add = POST, remove = DELETE, get = GET.
	</ul>

<hr>
There are 2 options to connect frontend and backend: **Refactor Spaces** or **API Gateway**.
<br>
1. Amazon Migration Hub Refactor Spaces
- orchestra networking and re-route shopping cart traffic to microservice.
- frontend -> Refactor Space Proxy (single endpoint that routes traffic to multiple different backend services based on rules) -> Refactor Spaces Services (services that provide application's business capabilities - legacy service and shopping cart service)
- Refactor Spaces enable account level isolation of services. Achieved by bridged VPCs and proxies that communicate via private IP addresses.
	- best practice: 
	- legacy app 1 account.
	- each microservice (shopping cart) to be in own account, managed by seperate team.
	- Refactor Spaces management account.


2. Frontend S3 traffic -> Amazon API Gateway -> network load balancer -> AWS Transit Gateway -> backend
	- API Gateway allows you to define resources and methods
	- API Transit Gateway bridge private endpoints of services in VPC
	- the use of API Gateway allows the use of GraphQL (https://aws.amazon.com/graphql/). Access many schema/data sources, pull from multiple different microservices with single request.

Cloud formation to define infrastructure resources as a code. yaml template file. Can configure, version, deploy, scale/repeat.

Some considereations when migrating to microservices:
 - microservice stack: how to implement? Cloud Functions? Kubernetes?
 - microservice data access: how to ensure consumers currently accessing APIs will continue to do so?
 - data migration: monolith to microservice - how to move existing data?
 - microservice switch over: how to do so seamlessly?
 - microservices and monolith internal communication: focus on performance, latency. inter-process vs inter-thread. chatter on network that impact performance.

Migration to microservices-based architecture using strangler pattern (https://martinfowler.com/bliki/StranglerFigApplication.html)
 - involves 
	- incremental app refactor while in production
	- eventInterception (ingress traffic divert to microservice)
	- assetCapture (move set of resources out, https://martinfowler.com/bliki/AssetCapture.html)
	- event stream -> messaging queue -> service layer / database trigger -> subset of database tables
	- migrate functionality, value transfer
	- reverse migration back to monolith must be possible to reduce risk
	- shorter release cycles

Patterns of Enterprise Application Architecture
https://martinfowler.com/books/eaa.html
Errant Architectures
https://www.drdobbs.com/errant-architectures/184414966
