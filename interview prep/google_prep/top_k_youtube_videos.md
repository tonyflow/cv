# Non functional requirements
### Latency
Relate latency numbers 
* 1 minute between the event and it appearing on
* 10s of ms to retrieve the top-k

### Scale
* Billions of videos
* 100s of thousands
* Precise results? Approximation?

Ask before moving forward: Is there anything I need to cover before movign forward? 


### Tumbling window state
VideoViews{Hour, Day, Month}
* video ID
* views count
* timestamp



### Sliding window considerations
* what is the actual shift: 1 min, 2 min, 5 minutes?
* For counts we can get the delta

VideoViewsLast{Hour, Day, Month}
* video ID
* views count
* timestamp

VideoViewsMinute
* video ID
* views count
* timestamp

### Approximations
* Min sketch count to min heap for sorting
* 