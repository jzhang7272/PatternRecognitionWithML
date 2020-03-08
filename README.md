# PatternRecognitionWithML

## Inspiration
Patient matching seems like it should be a simple problem, yet hospitals still have a high rate of failure at matching data for patients across different sources. Each hospital in the US pays an average of $1.5 million a year, costing the medical industry $9.2 billion each year. Because of minor discrepancies in the way data is reported by different healthcare workers, a hospital may not be able to match all the necessary data to a patient to provide the right care. Even worse, a patient’s data may be incorrectly matched to a different person, leading to dangerous consequences. Developing a better algorithm that accurately matches data to the right patient could save lives by preventing people from receiving an incorrect medication or diagnosis.

## What it does
Our algorithm is a full solution pattern matching system that was created for matching patient records. Given a dataset of patient records, with each patient record containing information of different features such as birthday, name, sex, address, etc., our algorithm clusters the vectors into clusters where each cluster represents a person, and all records within that cluster are patient records made by the same person. First, our algorithm cleans and processes the data, turning missing values into numerical zeros that can be worked with, standardizing address, state, and other abbreviations, and processing strings for special characters and matching meaning across different date input formats. Next, we select which features in the data should be used in the clustering algorithm, allowing the user both quantitative and qualitative evaluations to support. In this dataset, we based our choices primarily on where missing values were placed because there were so many of them in certain categories, but given more time we would like to implement an automated system using a standardized metric such as PCA or some variation of that. Lastly, we cluster the data using partitioning around medoids, also known as k-medoids clustering. In addition to the benefits that it has over its more commonly known cousin, k-means clustering, in terms of being more robust to noise and outliers, it is a much better fit for mixed-type data. In patient records datasets, and many real life datasets as well, we have numerical, nominal, and ordinal features. This makes it hard to cluster using traditional methods, as clustering works best with numerical data. K-medoids has the advantage of fitting in well with the gower distance algorithm, which can calculate distances between two entities that have a mix of categorical and numerical values using a system of weights based on similarity/type, and the Manhattan distance algorithm. It returns a dissimilarity matrix of the distance values between patient vectors, which then acts as the input for k-medoids. The resulting output, after clustering, will be the id of each cluster that corresponds to a patient, and then the patient records associated with that patient, located within the cluster. 

## How I built it
In the data cleaning portion of the algorithm, we used the pandas library, which has many useful functions for cleaning data. First, we replaced all missing values with zeros. We chose this value because a single zero actually appears very infrequently on patient records, and will not raise errors in algorithms but can be easily type checked against the common data input of type string to determine that it represents a missing value. Next, we created dictionaries of address abbreviations (Drive to Dr, etc.) and state abbreviations (Wyoming to WY), so that after reducing all strings in those features to a common lower or uppercase font, we could search for a substring of the abbreviation and for strings of the non abbreviated form to reduce all strings to either an abbreviated form or expand to a non abbreviated form. We chose to reduce all address/state inputs to an abbreviated form, as searching for abbreviations in strings can yield false results, such as finding “OR” in a word instead of when it is used to represent Wyoming. We also removed characters such as “#” and “/”, which are common among dates and address but cause encoding issues in algorithms. 

Next step is clustering. Before we implement the k-medoids algorithm, we used the silhouette method to determine the number of clusters we should use as a parameter in k-medoids. A silhouette value is a value between -1 and +1, where the higher the value the better the match between an object and its cluster, in comparison to its neighboring clusters. We look for a maximization of objects with a high value, and the clustering configuration that provides that is the one we select. Although the silhouette method can use any distance metric, we employed Manhattan distance over Euclidean distance, since Euclidean distance is easily influenced by unusual values, and since much of our data was originally categorical, unusual values are not uncommon. 

Then, we use gower’s distance algorithm to produce a dissimilarity matrix of our patient vectors. Gower’s algorithm works by assigning weights to categorical variables, and assigning them such that each categorical value can be transformed into a numerical value, but keep its relative comparison with other similar values in its category. Then, the distance is calculated between each categorical to numerical value between different patient vectors, and then combined for that vector, using Manhattan distance (which is what we used) or another distance algorithm. The output is a dissimilarity matrix whose values become data points in the k-medoids clustering algorithm. K-medoids clustering works as the following:

1.	Select *k* of the *n* data points as medoids using a greedy algorithm, where *k* is the predetermined number of clusters, and *n* is number of data points available from the dissimilarity matrix. 
2.	For each remaining data point, calculate their distance to each of the medoids, and selects the medoid the shortest distance away. Distance is defined in this situation as the sum of dissimilarities between the two points. 
3.	For each medoid, take each of the non-medoid data points, and compute the cost of swapping them. Cost is defined in this situation as the new sum of dissimilarities between the new medoid and points in their cluster. If this cost is better than the previous cost, we make the swap permanent and set this value as the new best cost. 
4.	We repeat until we can no longer find a swap that decreases the cost. 



## Challenges I ran into
The data given, since it was mock data of real patient records, had many missing values, wrongly inputted values, and values with special characters. Furthermore, there is no universal standard for data collection with regards to different patient features. For example, a patient’s name could be “John Smith”, and could be entered any number of ways: “john smith”, “JOHN SMITH”, “john SMITH”, or even the most dreaded: “Jon smit”. A patient may also input their birth date wrongly in one form, correctly in on form, and not at all in another. Thus, the first challenge we faced was cleaning the data. We had to standardize input formats for different features, and then transform each patient vector to conform to those features. For example abbreviations are used for states and addresses, such as California to CA and Road to Rd, but not consistently. We must be able to take those two separate inputs and turn them into one uniform input. This required an expansive use of regular expressions to identify substrings and matching patterns in input, both to combine equivalent but different presenting values but also to account for typos and user error. Secondly, we were faced with the problem of missing values. Since our dataset was already small, only 200 patient vectors, we take easy ways out like removing vectors with many missing values or removing features with many missing values. Thus, we had to both quantitatively and qualitatively evaluate which features to remove, and test how removal of different features impacted the clustering result. Lastly, although this is a clustering problem at heart, clustering with categorical variables brings its own unique set of challenges, since clustering requires a transformation of categorical values into numerical ones. This transformation has to be done extremely carefully to preserve categorical information, scale, and variance. We approached this problem through using Gower’s formula to create a dissimilarity matrix. However, this is a problem that can be continuously improved on, since the weighting of each variable needs to be evaluated from background information of the topic in addition to quantitative analysis of the data itself. There were instances in which two patient records were extremely similar, except for one feature, such as birthday, or a one letter difference in a name. We had to come up with a weighting system that can catch these discrepancies and cluster accordingly, but also not overweigh them so that those who make a mistake or have inconsistent records will be unable to match with themselves. 

## Accomplishments that I'm proud of
We developed a k-medians clustering algorithm to analyze reported information from different sources and match data from the same patients to each other. Our algorithm achieved a 95% accuracy, which is much higher than the accuracy rate that hospitals currently have of matching data. Furthermore, we achieved this accuracy rate with a small dataset of only 65 patients; most hospitals process thousands of patients each year, and with more data, our algorithm could become even more accurate. We also successfully implemented a new algorithm that we have never worked on before; we attempted to use multiple other clustering and classification methods for this data, but ultimately came to the conclusion that a PAM clustering algorithm is the best fit for this dataset. Our algorithm would significantly reduce the problem of patient matching, and as a result, improve overall healthcare.

## What I learned
Initially, we started with an idea of using neural networks such as the Multi-layer Perceptron classifier; we created measures of similarity between different rows of data, and trained the neural network with similarity data from pairs of rows, with the network learning how to determine whether a pair of rows belonged to the same patient. We created measures of similarity for each measure by extracting the longest common substring between the data for a certain measure, such as the first name, for two rows, and inserted it in a 2D "similarity" array as part of the training data. However, we learned that classification algorithms perform sub-optimally with the provided dataset, as it was not balanced and contained far more rows that corresponded to non-matched pairs. This lead to low sensitivity and F1 score, since the algorithm was biased towards classifying pairs as non-matches, which resulted in a high number of false negatives (incorrectly classifying pairs as non-matches). Attempts to balance the training dataset were made, such as creating a new training dataset with 50% matches and 50% non-matches. However, there are no set guidelines on the ratio of matches to non-matches in a training dataset, so we decided to not use this original idea to avoid discrepancies. 


## What's next for Pattern Recognition with Machine Learning
Our Pattern Recognition program could be applied in a multitude of areas in both the medical and non-medical communities. However, there are also improvements that can be made. A Principle Component Analysis (PCA) could be applied to first filter the dataset before it was fed to the PAM clustering algorithm. The PCA would reduce the dimensionality of datasets consisting of many variables correlated with each other, either heavily or lightly, while still retaining the variation present in the dataset. 

Patient matching arose from the transition of paper medical records to electronic health records (EHRs) that help clinicians order medications, document treatment decisions, and review laboratory results. Digital records introduce numerous efficiencies and provide medical professionals with in depth information to base decisions on. However, the sharing of medical data has led to an increased need for accurate patient matching algorithms, as the records for a client can vary across different medical health care providers, or even across different visits to the same provider as humans are prone to slight errors in providing information. The successful exchange of health data relies on this capacity to successfully search each individuals’ records in disparate locations, and know that it corresponds to the same person. 
Deficiencies in matching patients to their records can lead to safety problems, lost records, and other costly mistakes. Our patient matching algorithm offers a high accuracy in matching clients, thus reducing the flaws in current systems. 

Pattern recognition extends beyond patient matching. For example, a previously trained neural network could be implemented in spelling correction. A pre-trained network would be able to accurately predict what word the user is attempting to spell if it was given a training dataset of misspelled words, and the corresponding correct spelling. This is incredibly similar to the dataset we employed in creating this pattern recognition algorithm, and could easily be applied. 
