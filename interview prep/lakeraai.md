# Lakera
- Jolt
	- engineer and manager
	- swe in google
	- search and ads
		- nlp
		- information retrieval
	- 


- structure of the teams
- thomas hoffman
	- chief scientist
	- 

- computer vision
	- tooling for robustness of your model
- pivoted into the security of LLMs because of Gandalf
- lakera guard
	- prompt injection detection
	- classification problem
	- the model is not an LLM
	- tools for securying LLMs
	- deep learning
	- shipping this to various
	- generic
		- mallicious intent
	- hiring
		- 

- more and more 
- data intensive?
	- 


- structure of the teams and the role you're searching for.
	- SF team
	- 15 - 20 engineers 
		- 4 - 5 smaller teams
		- New strong new grads
		- Tech Leads
		- ML team
		- Infra team
		- Product team
- proprietary Gandalf data


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


curl https://api-inference.huggingface.co/models/KoalaAI/Text-Moderation \
	-X POST \
	-d '{"inputs": "I like you. I love you"}' \
	-H 'Content-Type: application/json' \
	-H "Authorization: Bearer <bearer_token>"

	focus on this area

	====	====	====	====	====

### Interfaces with other components
### Caching

### HTTP compression
At this point the API does not cap the input text. Theoretically, the user can sent an arbitrarily long "sentence" to classify.
First of all in order to avoid that, we can cap this - as mentioned on the error model as well - but still even if we cap it 
to 300 words, this would be significant network load and could 
### Risks


- network security and groups with other interfaces
- get vs post for longer texts for classification
- check all docs
- HTTP compression


import helpers

helpers.query_http_endpoint(url='foo',bearer_token='bar',payload={})

I have a list of strings and a new string as an input. What is the fastest way to decide which string in the list of strings is most similar with the input string?


- Applied ML engineer
	- managing the data pipeline
	- how is the data pipeline designed

- Database tools and prototypes
	- platform and the software




- 


