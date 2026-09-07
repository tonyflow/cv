# Harmful content detection
* Axis:
	* Research
	* Distributed Systems
	* Systems
	* Data

## The framework changes a bit
The phases can be the following:
1. Problem framing
	* Differentiate between businesss objectives and ML objectives.
	* We might not need to classify the content upon every post. e.g. We might have a piece of profanity on a post but if not many people see it, it might be fine. As the number of views for this post though increases we will want to remove/delete/sensor the post.
2. High level design
	
3. Data and features
	-- DATA
	* Supervised data
		* Labeled data
		* Open NSFQ datasets
	* Semi-supervised data
		* user reports
		* negative user reports
	* Unsupervised data
		* post comments
		* user behavior
	-- FEATURES
	* Content features
		* concatenated text
		* images of the post
	* Behavioral features
		* Real-time tallies of
			* negative reactions/view
			* shares/view
	* Creator features
		* User embeddings
		* age of account
		* last N postings of harmful content
4. Modeling
	* For the classification we might want to have 3 different classifiers
		* Text classifier
		* Image classifier
		* Behavior classifier
		* Average the scores and decide whether this is harmful or not
5. INference and evaluation
6. Deep dives 
7. Wrap up