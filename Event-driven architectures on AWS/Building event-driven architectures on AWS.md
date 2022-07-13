# Building event-driven architectures on AWS
#### Event-driven architectures
 - subscriber / target services do work in respond to publisher / source services
 - **publisher / source**
    - own application
    - \> 200 AWS services including AWS Lambda
    - AWS's SaaS partners (e.g. MongoDB - realtime changes in database trigger execution of serverless server-side business logic, notification alerts, S3 archive of event changes for audit trail)
    - microservices
 - **subscriber / target**
    - component of own app in container
    - AWS Lambda, Amazon Simple Notification Service (SNS) topics, Amazon SQS queues, EC2 instances, ECS tasks, Kinesis streams, Step Functions state machines, Amazon API Gateway
    - monitoring and logging services. Amazon EventBridge is integrated with Amazon CloudWatch and AWS X-Ray.
    - can use CloudWatch as target to test rule during development
 - **event**
    - an input, a change in an environmental variable
    - JSON objects
 - benefits:
    - loose coupling
    - more autonomy for developer to work/iterate on a feature (function) of an application without too much dependences on another part
    - application components can be geographically distributed: more resilient
    - scale component in response to usage demand

 #### AWS services
 - Amazon EventBridge, Amazon SNS, Amazon SQS, AWS Lambda
 - Amazon EventBridge
    - serverless event bus
    - an event router between publishers and subscribers
    - router has rule to filter for specific event pattern
        - route matching event to specific target for processing
        - a single rule can route to multiple targets, all processing in parallel
        - a rule can customize the JSON event passed to target, e.g. passing only a part, or replacing with a constant.
    - Schema Discovery can automatically infer schemas from events and add to Registry
        - can detect changes to event schema and auto generate new versions.  
    - Discovered Schema Registry stores the structure and content of events (data).  
        - schemas for AWS services automatically available
        - can output strongly-typed JSON objects for use in coding (Java, Python, Typescript).
        - integrates with AWS Severless Application Model (SAM) https://aws.amazon.com/serverless/sam/
            - a serverless application is a combination of Lambda functions, event sources, mappings, APIs and databases.
            - start from AWS SAM template specification
    - you can create different event bus for different type of event publishers
    - subscribers need not know publishers
    - can add/remove publisher/subscriber without affecting existing connections
    - auto-scale in response to volume of event ingested
    - native archive and replay capability: can recover from failure or build new app version based on old events

#### Event pattern matching (Amazon EventBridge)
 - for a match, event MUST contain all fields listed in pattern. Same nesting structure.
 - Fields in the event not  listed in pattern are ignored.
 - matching is exact (character-by-character).
 - values follow JSON rules: strings in "", numbers, true/false, null.
 - numbers match at string representational level. 300, 300.0 and 3e2 are not the same. 

#### Step 3: Verify Step Functions workflow execution

**Extra credit**
Implement an event pattern using _Prefix Matching_  that matches events with a location the begins with eu-. Do not modify the Step Function target and verify backwards compatibility by reusing the sample data above. Modify the test data to use a location, eu-south, to verify that additional EU locations trigger execution of the Step Functions state machine.
https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns-content-based-filtering.html#filtering-prefix-matching

Solution
EventBridge -> Rules -> Event pattern:
```
{
  "detail": {
    "location": [{
      "prefix": "eu-"
    }]
  }
}
```
**Extra credit**
Implement an event pattern using _Prefix Matching_  and _Numeric Matching_  that matches events with a location the begins with eu- AND an Order value greater than 1000. Do not modify the Step Function target and verify backwards compatibility by reusing the sample data above.
https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns-content-based-filtering.html#filtering-numeric-matching

Solution
EventBridge -> Rules -> Event pattern:
```
{
  "detail": {
    "location": [{
      "prefix": "eu-"
    }],
    "value": [{
      "numeric": [">", 1000]
    }]
  }
}
```
#### SNS Challenge
**Extra credit**
Implement an event pattern using _Exists Matching_  that matches events which do NOT require a signature (ie. signature does not exists). Do not modify the SNS target and verify backwards compatibility by reusing the sample data above.
https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns-content-based-filtering.html#eb-filtering-exists-matching

Solution
EventBridge -> Rules -> Event pattern:
```
{
  "signature": [{"exists": false}],
  "detail": {
    "category": ["lab-supplies"],
    "location": [{
      "prefix": "us-"
    }]
  }
}
```
**Extra credit**
Implement an event pattern using _Prefix Matching_  and _Anything-but Matching_  that matches events with a location the begins with us- but NOT a location that is us-east. Do not modify the SNS target and verify backwards compatibility by reusing the sample data above.
https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns-content-based-filtering.html#filtering-anything-but

Solution
EventBridge -> Rules -> Event pattern
```
{
  "detail":{
    "category":["lab-supplies"],
    "location":[{"prefix":"us-"}, {"anything-but": "us-east"}] // not certain, need both filters..
    }
}
```
#### Amazon EventBridge Archive and Replay
https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-archive.html
- record any event processed by any event bus
  - all events, or filter for subset based on pattern matching logic (rules)
- can be used for testing after code fixes, validate new features using historical production data
- Process: EventBridge create archive, have filter for events from xxx source ->  create EventBridge Rule with filter for replay-name attribute, target SQS Queue -> Start Replay -> validate messages received in SQS

#### Event-driven with Lambda
- Asynchronous / Stream invovation of Lambda -> Fail or Success -> Record to AWS resources (SQS queue, SNS topic, Lambda function, EventBridge event bus)
- 2 ways to handle errors during execution with Lambda: Dead Letter Queues (DLQ), Lambda Destinations.
  - Destinations preferred because more useful: pass function execution info, e.g. code exception stack traces

#### Event-driven with Simple Notification Service (SNS) 
 - SNS in place of EventBridge
 - to react to high throughput (HTS) or low latency messages from other microservices
 - for fan-out to thousands/millions of endpoints
 - messages are unstructured, in any format
 - barebone of pub/sub implementation: publisher (SNS topic) -> message -> subscriber/endpoint (Simple Queue Service/SQS, Lambda, HTTP/S, email)
 - SNS topic is logical access point for event generators
 - use message filtering to limit to a subset of messages from a topic
    - SNS message filtering happens before message delivery.
    - by default, SNS subscriber receives all messages published to a topic
    - apply filter policy to subscription
    - "filter policy" is simple JSON object, with attributes that determine which messages a subscriber receives
    - Steps:
        1. create SQS Queue
        2. from created SQS Queue, subscribe to SNS topic of interest
        3. from SNS Topic, select the subscription from SQS. Edit subscription filter policy to add required JSON key-value attribute. Only messages with matching key-value pair will be received at SQS.<br>
        https://docs.aws.amazon.com/sns/latest/dg/sns-subscription-filter-policies.html
  - error handling
    - Dead-Letter Queues (DLQ) exist for SNS, SQS and Lambda
    - increase resiliency and durability of pub/sub system
    - e.g. messages published to a SNS topic, but undeliverable to subscribed endpoints, are sent to DLQ
    - 2 reasons:
      - message sender (client-side, SNS) errors
        - example: stale subscription metadata. SQS queue/Lambda subscribed to SNS topic deleted, but SNS subscription remains.
        - no retry of delivery by SNS
      - message receiver (server-side, system that hosts SQS, Lambda, customer-managed email, mobile, SMS, HTTP) errors
        - example: server hosting system unavailable; failed to process a valid request from SNS
        - SNS retries failed deliveries for certain number of times specified by retry policy
        - https://aws.amazon.com/blogs/compute/designing-durable-serverless-apps-with-dlqs-for-amazon-sns-amazon-sqs-aws-lambda/


