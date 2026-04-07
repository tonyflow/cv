# Behavioral questions
### Tell me about a conflict and how you resolved it
- We were at a point that out Terraform state was bloating up very aggresively. This was blocking a lot of engineers with everyday tasks, thus we were in a process of refactoring this single one state. Every team was responsible of removing their parts from the monolithic state and moving them to separate ones. The conflict there was about the structure of the new states.
	- I though that certain levels of indirection were not necessary: distinction between batch and realtime was enough and we did not need a further breaking down of the realtime and batch services
	- On top of that, I considered that the state moves were becoming more complex. Especially with the version of Terraform that we had back then,
	- I built a working prototype and presented it to the team. I also created a small Confluence page supporting my case.
	- The team decided that the previously suggested solution documented the system in a better way. Mirrored some business requirements in a better way. They were right and I dropped my argument

- Missing requirements from GDPR pipelines

- MSA error model
	- Created Tech Spec
		- Discussed it with the assigned engineer
		- The PR did not contain what was discussed
		- The specifications of the error model and especially how this was modelled in code - Scala - did not reflect what was discussed


### What excites you?
- Cross team collaborations

### What frustrates you?
- Bad communication and not getting oneself across can be frustrating.
- Communication though is a bilateral process and one has to assess both parties' quality of communication. It takes two to bridge this gap. It takes two to tango.

### How do you tackle challenges? Name a difficult challenge you faced while working on a project, how you overcame it, and what you learned.
- I think that the most difficult challenges arise from cross team collaborations
- Different teams and different people have different opinions
- Requirements are set from a group of people
- There are dependencies to different groups of people
- GDPR pipelines
	- Product requirements
	- Legal requirements

- Another challenge
	- 

### What are the most interesting projects you have worked on and how might they be relevant to this company's environment
- Throughout my career - and especially during my last 3 positions - I have been working on the interface between data analytics and 


### Tell me about a time you had a disagreement with your manager.
- MSA Aggregation
	- Aliasing certain attributes from different data sets so that we can union these two data sets and aggregate them as one
	- My manager thought that creating a new aggregation provider that could aggregate different attributes in a generic way would be more suitable
	-  I was arguing that this approach is definitely doable but much more complex and would endanger the delivery of the project.

### Talk about a project you are most passionate about, or one where you did your best work
- Breaking a monolithic application and allowing the company to proceed with a global rollout of their platform
	- Router
	- Analytics
	- Pollers

### What is the most constructive feedback you have received in your career
- Self-leadership
	- Setting your own goals
	- Pushing for them
	- Being able to employee the correct resources in order to lift all source of ambiguity in a project
	- If you cannot do this for yourself, then how can you can lead others?

### Tell me about a time you met a tight deadline
- Deduplicating data from the Exchange
- There are different ways of deduplicating data:
	- Group them in a time window per user
	- More sophisticated ways like clustering data using a DB scan algorithm
	- Deduplication also differs if you do it on batch or a realtime latency
- This was supposed to be a multi-spring project but the US product managers decided that it was business critical to roll out this feature much sooner than expected. 
- I remember it was Friday
	- Deduplicate only batch trafic
	- Do not ingest realtime traffic apart from certain events: This was a cost issue


- Is it mainteanable?
- Is it testable?
- Is it intuitive?