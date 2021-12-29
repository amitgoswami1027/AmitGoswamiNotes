
# Machine Learning Interviews & System Design 
Machine learning (ML) has been one of the fastest-growing fields, and it is projected to grow from $7.3B in 2020 to $30.6B in 2024. Machine learning aims at solving a multitude of complex problems and has seen rapid progress in several areas such as speech understanding, visual understanding, search ranking, credit card fraud detection, and so on.

In the ML Interviews, candidates are given open ended problems and are expected to build an end to end machine learning system to solve the problems.

# Problems Using ML
* Build a recommendation system that shows relevant products to users.
* Build a visual understanding system for a self-driving car.
* Build a search-ranking system.
* Build a system that shows relevant ads for search engines. 
* Extract all persons, locations, and organizations from a given corpus of documents. 
* Recommendation movies to a user on Netflix. 

In the ML Interviews, candidates are given open ended problems and are expected to build an end to end machine learning system to solve the problems. In order to solve the problem we should consider the following approach.

<img width="980" alt="image" src="https://user-images.githubusercontent.com/13011167/147432203-f96568b3-d612-4358-a047-6a721878d62a.png">

## Understanding scale and latency requirements
### Latency requirements
* Do we want to return the search result in 100 milliseconds or 500 milliseconds?
* Do we want to return the list of relevant tweets in 300 milliseconds or 400 milliseconds?
### Scale of the data
* How many requests per second do we anticipate to handle?
* How many websites exist that we want to enable through this search engine?
* If a query has 10 billion matching documents, how many of these would be ranked by our model?
* How many tweets would we have to rank according to relevance for a user at a time?

## Defining Mertics
### Mertics for Offline Testing
For Binary Classification 
#### AUC, Log loss, Precision, Recall, and F1-score
* In other cases, you might have to come up with specific metrics for a certain problem. For instance, for the search ranking problem, you would use NDCG as a metric.

### Mertics for Online Testingv(Production)
* Require both "component-wise" and "end-to-end metrics".
* A commonly used end-to-end metric for this scenario is the users’ engagement and retention rate.

## Architecture Decissions 
To get an idea, have a peek at how the architecture of a search engine’s ML system may be designed, below
<img width="1085" alt="image" src="https://user-images.githubusercontent.com/13011167/147433638-b12e8d37-1dca-421a-9399-0ebdc8405465.png">

# ML Techniques and Concepts
* Performance based SLA ensures that we return the results back within a given time frame (e.g. 500ms) for 99% of queries. 
* Capacity refers to the load that our system can handle, e.g., the system can support 1000 QPS (queries per second).










