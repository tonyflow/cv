# System Design
### Requirements
#### Functional 
	- Use cases
	- Even if you're given a use case you can try to reduce the scope of the requirement. This means that even when you're given a requirement, this might be ambiguous
	- The app should do
		- x
		- y


#### Non functional
	- Scale: How big do we want the system to be:
		- Users
		- Data traffic
		This we where we should do some math

##### Numbers
This is where we should make some detailed assumptions about the app we're building. Run some math based on:
- number of users
- data input
- what kind of data are we handling and what is the amount of data?
	- how many videos/songs/text uploads?
	- what is the insertion frequency per second/minute/hours/week/month/year?
- what kind of metadata do we need?

byte -> 1 byte
kilobyte -> 1,000 bytes or 10^3 thousand
megabyte -> 1,000,000 bytes or 10^6 million
gigabyte -> 1,000,000,000 bytes or 10^9 billion
terabyte -> 1,000,000,000,000 bytes or 10^12 trillion
petabyte -> 1,000,000,000,000,000 bytes or 10^15 quadrillion

Network card capacity:
- Ethernet NICs can handle 100 Gbps or higher

	- Availability
	- Latency
		- Percentiles of latency
	- Monitoring
	- Logging
	- Metrics
		- Success metrics
	- Testing (unit,modular,integration)
	- ...


### Success metrics
This is driven by product and 


### APIs
Rough draft of the endpoints that you want to use
- POST examples/a/b/c with body {} and response code and status

### Start drawing the components
- Load balancers
- App servers

### Database design
- Mutable data: Use a SQL database
- Immutable data: Use a DFS like S3
- What is the traffic patterns related to the data?
- What kind of access control we want to provide to the data?
- Do we need indexes for the data?
- Transactionality
	- 
- We can have metadata stored on SQL. The SQL could contain the links to a DFS like an S3.

### Control flow
Go through the entire control flow for every endpoint and make sure that you have a fully working solution before going into optimizations.

This is case specific and we should accumulate all the knowledge that we drew before in order to make a case for what we're talking about


# Optimizations and Refinenments

### Caching
- Database level
- CDN level

### Load Balancing
How do you ensure that the webservers are NOT overloaded?
- Mem or CPU overload?
- What should be the metric for load balancing?
	- Network traffic?
	- Geolocation?
	- Mem?
	- CPU?
	- Combination of all previous

### Scale

### Monitoring,Logging and Metrics
Another area that should be investigated here are how and where these metrics should be displayed. Where will these dashboards be?

### Bottlenecks and Risks

### Migration
Is there an existing system? Can we directly move to the design that we are describing?

### Replication

### Documentation
- Internal Technical
- Internal non-technical

### Conclusion
Go through the initial requirements and check if everything has been addressed


# Top 10 Principles
## 1. Communicate Efficiently

## 2. Scope the problem

## 3. Start drawing after ~15mins
In a 45 minute interview. After you have set up the 

## 4. Start with a simple design
Establish a working solution before going into any optimization. Focus on creating a vision that does and will solve the problem. You should let the interviewer know about your breadth of knowledge around distributed system but you should only go deep into it if you're explicitly told so.

## 5. Utilize prep resources
- GitHub Systems Design Primer: https://github.com/donnemartin/system-design-primer
## 6. Properly understand the problem

## 7. Practice, practice, practice!
Don't underestimate the time that you'll need for a mock

## 8. Explain your thinking
Database choice: Describe the reason around your choice.

## 9. Get comfortable with the math
Average, min and max 

## 10. Use the drawing tool efficiently
- Draw boxes
- Text
- Arrows
You should definitely know HOW to draw these shapes on any drawing tool.