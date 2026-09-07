# Design ChatGPT
## Functional requirements
* Temperature
* Max tokens
* Search load and run existing presets

## Non-functional requirements
* Low latency:
	* Time to first token <=300ms
	* e2e completion <= 2sec
	* Q: How do we prevent slo token starts or long tails from degrading UX?
* Availability:
	* p99 SUccess rate acroos prompt submission flow
	* Q: How do we prevent a prtial outagte from breaking all completions?
	
* Rate limiting
	* <= 60 requests per user per minute
	* <= Concurrent generations per user
	* Q: How do we prevent spams from one user choking the sysmte?
	* 

* Scalability
	* Suport 10 QPS and 100k concurrent streaming sessions
	* A: How do we prevent sudden traffic spikes from overwhelming stream infra?
	
## High-level design
### API
```json
{

	"event":"generate",
	"request_id": "123",
	"prompt": "",
	"temperature": 12.3
}
```


### Entities
This can be a data class that could later be transformed into a database entitie. In this case it's more of a concept and state carrier

### User journey
1. Establish websocket connection via API-GW
2. user type prompt
3. rate limits and routing on the API gateway side
4. TODO

## Deep dives
### Model cold starts
* Time to first token
* We can avoid cold starts by
	* Doing nothing and hope OpenAI is always warm
	* Warm-up prompts
	* Create a model proxy layer: A wrapper around the model inference service
		* Smarter warmup
		* better routing
		* shield from upstream changes
		* future-proof hook


### Delays at the Model API
* Our own services can introduce delay
* Thread-per-request model
* FIFO queue + worker pool
* Adaptive load shedding ???

### Streaming Flush and Jitter in the websocket path
* Once the model is streaming tokens, how do we avoid choppy, bursty output?

### How do we prevent a partial outage to fail all the completions?
* Solve this like above: Resilient model proxy with circuit breakers: We can have multiple metrics like:
	* per endpoint health tracking circuit breakers prescribing to different rules like "if region a is b try region b instead". This can be done by using
	* multi-region / multi-model routing
	* 

### Preset service fails because of DB failures
* Preset resiliency layer

### Avoid multi-tab/multi-devide spam
* Centralized distributed rate limiting

### Queue flooding inside the completions service
* 