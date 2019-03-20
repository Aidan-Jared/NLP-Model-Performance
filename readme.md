# NLP Processing Data Featurization

![alt_text](images/headImage.png)

For this second capstone I decided to investigate how well the different data featurization methods perform. In order to test this out I decided to use the Amazon reveiw dataset provided by Amazon because the data included many different writen reveiws along side the star reveiws given by each users. This alowed me to work with scalable data and to test how well the methods were performing.

## EDA

The Amazon Reveiw data set is a collection of over 130 million reveiws split up into multiple catigories. For this project I decided to just look at the first set of book reveiws. After downloading the data I found that there was over 2 million rows and my ccomputer was struggling to even run basic scripts with it. So I wrote a script to take in the data and do a stratified train test split to perserve the ratios of reveiws and massivly reduce the amount of data so I could write and develope code with it.

I then decided to look at the data and understand the distributions and formating I was working with. The main features I decided to work with was the reveiw_body which was each users writen review of the data and the star_rating which was the numerical value each user gave.

The review body was filled with markdown notation and other figures that could throw off any NLP model built on it so I started by removing all markdown notation from the text and then using spacy I built my own function to remove puntuation, contractions, stop words, and urls. With this done I lemmed all the words and but the resulting strings through tfidf vecorizor.

After this it was time to explore the star ratings and I found the following distribution:

![alt_text](images/starting_class_distributions.png)

As you can see the data is very skewed towards high ratings with just about 2/3 of the data having a 5 star rating. After running some basic tests, I ended up finding that because of this skew, models loved to predict that no matter what the text was that a 5 star rating sounded really good.

To fix this probelm I decided on SMOTE the data. In order to do this I train test split the data and ran then applied SMOTE to the training data in order to add more ratings for the less used ratings. This produced the following distribution which I know will produce better models:

![alt_text](images/SMOTE_class_distributions.png)

## Data Vectorization

Even though this project is focused on strings, we still need to vectorize the data so that the models we use to predict will understand the data its looking at. The most common way to do this is the Bag-of-Words model but there has been a new development called Doc2Words.

### Bag-of-Words

Bag-of-Words works by creating a matrix where the rows are the documents of the corpus and the columns are the vocabulary. After you build the Bag-of-Words model you tend to make it into a tfidf (Term Frequency Inverse Document frequency) where the terms are the frequency a word apears in a document times how much information the word provides across the corpus.

### Doc2Vec

Word2Vec and Doc2Vec are recent developments in data science and NLP and I hope to show its power in this project. Word2Vec is decomposing any given word into a vector of a defined size where the larger the vector the more accurate it is. The advantages is that the vector retains document context and simular words will be represented simularly. For example in Bag-of-Words, Love and Like would be to seperate values in the matrix with no relation to eachother, but with Word2Vec, Love and like would be two simular vectors with a high simularity. On top of this mathmatical operations can be performed on the vectors such that <b>King</b> - <b>Man</b> + <b>Woman</b> should equal <b>Queen</b> on a well trained model.

![alt_text](images/Word2VecExample.png)

In order to produce a word Vector an Autoencoder Neural Network is used where the inputs are feature vectors of the document and the output is a prediction for a word. A window is moved over the document and selects the suronding words and trys to predict a word based on the context of the words around it. For example, if we give the model the sentance <b>"the cat sat on"</b> the model would read it in as the vectors <b>[1,0,0,0], [0,1,0,0], [0,0,1,0], [0,0,0,1]</b>. The window when then move over the inputs and choose a word to predict such as <b>sat</b>. The prediction would then be made off of the suronding words in this case <b>cat</b> and <b>on</b>. After traning, the weights that were found to work the best will be taken out of the autoencoder.

![alt_text](images/Doc2VecExample.png)

Doc2Vec is very simular but it adds in the additional input of a document vector. This then produces all the word vectors for the entire corpus as well as the document vector.

A good comparison for Bag-of-words and Doc2Vec is that Bag-of-words is the frequensist aproach to nlp while Doc2Vec and Word2Vec are the Bayesin aproach.

## Creating the Featurizations

In order to study how well these different featurizations perform I first had to figure out what tools and methods to use.

### Bag-of-words

For the Bag-of-words I decided to just use sklearns prebuilt tools and use PCA to reduce the dementionality. I first used the count vectorizor and then the Tfidf Transformer and I only looked at the top 5000 words to reduce the size. After this I trained the model on the training data and then transformed the test data. After this I looked at what the best choice for the number of componets in PCA would be to speed up the calculation.

![alt_text](images/Screeplot.png)

From this scree plot it looks like 100 componets will work best which is a large reduction in dimentionality which should speed up the computation.

### Doc2Vect

Here I decided to use two different systems, spacy and gensim. Spacy is a strong package for nlp and comes with its own general Doc2Vec model which I used to transform all my documents to vectors of a length of 300. Gensim is a package that includes tools to build your own Doc2Vec model. I personaly decided to set the vector size to be 300, the window to look at the 15 words infront and behind the word, only consider words that apear more than 5 times and to train over 800 epochs (Thanks to [This](http://www.aclweb.org/anthology/W16-1609) paper on Doc2Vec). I used the training set to build this model and then transformed the traning and testing data.

Because you can directly compare vectors here are 5 documents along with the most and least simular documents acording to Spacy:

|Original Document|Most Simular|Least Simular|
|-----------------|------------|-------------|
|Not only has Brooke McEldowney captured the pure essence of 'cat'|In the book Ginger Pye Jerry is just some normal boy who is trying to figure out how to buy a puppy.|Excelente libro, te capta, te entretiene, aprendes, te ries|
|Having worked with Brant for years and having had the pleasure of learning from him on a daily basis|This book delivers way more than one would expect. Let me explain that. First of all it is not that long of a book.|Excelente libro, te capta, te entretiene, aprendes, te ries|
|How wonderful to be able to write something for kids to do that is so appealing to read about as well. Makes scientific ideas easily acccessible.|This book is an eye-opener for those of us who like to think of themselves as rational people.|Ik heb nooit veel van pendelen gehouden, maar samen met dit boek is het een hele mooie|
|Prayer Rain is very inspirational.|Through this book you will experience the TRUE story of the Barrick family.  Their faith in God is truly inspirational.|WaWaWeeWa|
|I really like the way this book is set up. I feel like it is gonna be a good tool next year in nursing school|I've never been good at keeping a journal, but this has kept me excited and intrigued to see what the next day holds.|Este autor en general lo recomiendo ampliamente. Todos sus cuentos tienen moralejas|

and acording to my gensim:

|Original Document|Most Simular|Least Simular|
|-----------------|------------|-------------|
|Good info.|or what this book actually is, it needs to be downsized and released in an affordable paperback printing.|Great!|
|We ordered this through the publisher, Scholastic.  Since it was for a six-year old, I was disappointed|District Ranger Anna Pigeon of Rocky Mountains National Park is going to have wolves in the park;|CLASSIC!|
Kids enjoyed doodle book.|I got this as a gift for my son. Iwould reconmend it anyone. It arrived on time and in god condition.|will be great for baby gift|
|The content is accessible and well organized for quick acquisition of knowledge for teaching students, residents and colleagues not to mention for the reader as well. The references are either classic or outdated. A lot of work and time went into the preparation of this book. Are the authors and editors planning to revise and publish a more recent edition?|Review: FORGED (World of the Nightwalkers, Book 4) by Jacquelyn Frank Release Date: April 29, 2014 Reviewed by: Reading in Pajamas/ Donna Rated: 4 Stars Blog Post: [...]|CLASSIC!|
|I'm very pleased with this little Bible. I like that I can easily carry it with me most anywhere. I wanted a SMALL Bible, and this is perfect.|So many travelers dream of visiting or regularly visit Paris!  We were last there for a week in 2013 and I used that year's edition of the Lonely Planet Paris guide|CLASSIC!|

As you can see there is some understadable realationship between each of these documents for what spacy produces with non-english reviews always beeing the most dissimular. With Gensim however the results seem a bit more random which I belive comes from how small the training set was (less than 2000 documents) which makes it a weaker model at the moment. But since I can train the gensim model I think that increasing the amount of data it trains on would inprove its results.

## Performance

In order to test the performance of these models I decided to use the Random Forest Regressor and find the Mean Square Error for Bag-of-Words and Doc2Vec as well as the computational time in order to find which data vectorization method works the best. By using gridsearch to tune the hyperparameters for each data factorization. My results are in the table bellow:

|Factorization|Bootstrap|Max Depth|Max Features|Min Samples Leaf|Min Samples Split|Number of Estimators|
|-------------|---------|---------|------------|----------------|-----------------|--------------------|
|Tfidf|False|None|sqrt|1|3|80|
|Spacy|False|None|sqrt|1|2|90|
|Gensim|False|None|sqrt|1|3|80|

(parameter Explanation)

After training the predicting off of these modes I found the following results

![alt_test](images/Model_MSE.png)

As you can see Tfidif had the lowest MSE but it is only slightly different than spacy while gensim performed the words. But this is only one metric so now lets look at the time

![alt_test](images/Model_Time.png)

From this you can see that spacy had the fastest build time and its predict time is tied with gensim. As a point of comparison rember that both of the Doc2Vec methods produce vecors of length 300 while tfidf is only 100 items long.

If we look at both of these metrics, I would choose to use spacy because it performs only slightly worse than Tfidf but it trains the model the fastest and is tied for predict time. I belive that if the data was scalled up further or no pca being implimented spacy would massivly out perform Tfidf on the time while producing comparable MSE.

With Gensim I think that it is performing the worst because I am building my own model which means that it only has so many data points to train on. Spacy performs well because it is a model that has already been built on a large amount of data while gensim has only been built on less than 2000 points of data. If I increased the size of my data set I belive that gensim would start to out perform spacy and Tfidf when looking at MSE and run in a simular amount of time to both spacy and Tfidf.

## Conclusion

In conclusion, it seems that in smaller datasets Tfidf performs the best but that Doc2Vec will inprove with larger datasets and more traning time.

## Future Direction

I plan on taking this dataset and the tools that I have learned to start applying Doc2Vec into deep learning in order to build much stronger models on larger data.