# Cloud Native Design Patterns
### GateKeeper Pattern
  * provides additional layer of security to attack surface
  * must run in limited privileges
  * no business logic in gatekeeper
  * SSL offloading/termination (improve performance)
  * latency is expected due to additional layer
  * single point of failure
  * use when sensitive info to be handled
  * use when request validation in dist apps
  * Ex: APIGW & WAF
### Gateway Aggregation Pattern
  * like load balancer
  * Use when client needs to communicate with multiple backend services to perform an operation
  * Use when client may use networks with significant latency, such as cellular networks
### Gateway Offloading Pattern
  * load balancer with TLS termination
  * LB must be highly avail & scalable
  * Use when shared concern such as SSL certificates or encryption
  * Use when move security responsibility to other teams
### Gateway Routing Pattern
  * simplify client applications by using a single endpoint
  * use when client needs to consume multiple services
  * use when external requests needs multiple endpoints
  * example ALB, IngressController
### Priority Queue Pattern
  * process high priority requests quickly
  * define priorities, muliple queues 
  * need monitoring, scale it & requires cost
  * use when multiple tasks with priority
  * example is SQS
### Publisher-Subscriber Pattern
  * Publisher, message broker and subscriber
  * decouples,scalabile, reliable, async, scheduled processing
  * simple to integrate multile systems (suscribers)
  * use when broadcast information
  * use when near real time sending of info to muliple consumers
  * use when eventual consistency model
### Queue-Based Load Leveling Pattern
  * message queue one way communication
  * queue is in between service and database
  * use when app uses services & need overloading
  * example is SQS
### Asynchronous Request-Reply Pattern
  * client makes sync call to API, 
  * API responds quickly(202)
  * then API callback another service and returns 302(found) & then URI
  * use in browser applications and async reply
  * example SQS
### Bulkhead Pattern
  * if single app expose multiple services and excessive load
  * partiion the service instance to multiple groups
  * isolate critical consumers
  * can use with retry and circuit breaker
### Retry Pattern
  * sending the request one more time if transient failures
  * Cancel, Retry & Retry after delay strategies
  * 
### Static-Content Hosting Pattern
### Claim-Check Pattern
### Ambassador Pattern
### Anti-Corruption Layer Pattern
### Strangler Fig Pattern
### Backends-For-Frontends Pattern
### Sidecar Pattern
### Throttling Pattern
### Valet Key Pattern
### Federated Identity Pattern
### Deployment Stamp Pattern
### Geode Pattern
### External Configuration Store Pattern
### Choreography Pattern
### Competing Consumers Pattern
### Cache-Aside Pattern
### Sequential Convoy Pattern
### Compensating Transactions Pattern

## Linkes
- http://en.clouddesignpattern.org/index.php/Main_Page
- https://aws.amazon.com/architecture/reference-architecture-diagrams/?achp_addrcs5
- 