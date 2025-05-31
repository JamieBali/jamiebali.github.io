---
layout: post
title:  "Categorising Support Ticket Quality"
date:   2025-05-24 16:00:00 +0000
categories: AI
---

Part of our termly review where I work, we have a bunch of analysis done on our ticket work including resolve counts, ticket prioritisation, and analysis of the "quality of support ticket comments". This sounds like it'd be quite arbitrary and subjective and yes it kind of is. We have a list of steps we should aim for, but there is no objective way of determining what makes a comment "good" or "bad" when it comes to characteristics like politeness, conciseness, etc...

On top of this, we have only 2 collegues who need to go through and do this quality analysis for the remainder of employees taking around 16 hours between the two of them.

My proposal as a workaround to this solution is, as it always is, train an AI to do it.

<a href="https://github.com/JamieBali/supportTicketCategorisation">Code is all available here. Training datasets not included in repo due to sensitive content.</a>

# Option 1 : Naive Bayes Sentiment Analysis

The Naive Bayes approach is a classic approach to sentence categorisation through Sentiment Analysis. The approach is fairly simple and "training" can be very quick - even with particularly large datasets.

A Naive Bayes classifier works by calculating the probability that a sentence conforms to a specific hypothesis. Obviously we don't know this implicitly, but we can calculate a selection of other parameters and use Bayes' theorem to calculate this probability.

$$P(H|S) = (P(S|H).P(H))\P(S)$$

In this scenario, the probability of the hypothesis (H) given the sentence (S) is equal to the probability that the sentence (S) would occur in the hypothesis (H), multiplied by the probability of the hypothesis (H). In Bayes' theorem, we would then divide by the probability of S alone, but we can skip this step in a Naive Bayes classifier, as the probability of S would be equal across all hypotheses we test.

The system is called a "Naive" bayes approach because we are making a lot of assumptions about the data, and we lose a lot of the context of what is being said.

## Step 1 : Pre-processing

Pre-processing is important to increase the accuracy of our model. This will normalise our data so that we have more consistency - particularly important when the imput data we have will be coming from a variety of users who may have different capitalisation, use different phrasing, or use different types of punctuation in different cases.

I've opted to use 3 different methods of pre-processing - removal of stop words and punctuation, lowering the case, and stemming.

### Removal of Stop Words and Punctuation

Stop words refer to common words like "the", and irrelevant words like pronouns. We can use the python module "NLTK" in order to import lists of punctuation and stop words.

{% highlight python %}
   import nltk
   nltk.download('punkt')
   nltk.download('stopwords')
{% endhighlight %}

We can then use the `re.sub` command to substitute and punctuation with a space, and remove any stop words entirely. This means that a sentence like "Hello. How are you doing? My name is Jamie" will be converted to "Hello How are you doing My name is Jamie"

### Lowering the Case

We can use the inbuilt `string.lower()` command in Python to convert all words to lower-case. This is another important pre-processing point as Python is a Case-Sensitive language - this means that without forcing everything to lower-case it would identify "hello" and "Hello" as two different words.

### Stemming

Stemming is used to remove the variable endings of words to further noramlise them. Words like "running", "runs", and "runner" will all get converted to just "run". 

{% highlight python %}
   from nltk.stem import PorterStemmer
   ps = PorterStemmer()

   stemp = ps.stem(word)
{% endhighlight %}

The alternative option to stemming is "lemmatization" which uses a dictionary to convert all words into their base form. Stemming is unable to figure out that "run" and "ran" are effectively the same word, whereas lemmatization would be able to spot this.

Stemming is a heuristic-based approach which means that it is able to deal with words that aren't in the WordNet dictionary that lemmatization would use (newly formed words like "skibidi"; or very specific technical vocab). Both techniques have their merit, and it's possibly worth testing the difference between these in the future.

After all our pre-processing, our sample sentence will now liike like "hello how do name jamie".

## Step 2 : Tokenization 

We start the approach by tokenising the data. Tokenisation means splitting up each of the sentences (or in our case comments) into individual words by splitting around any spaces. The nltk module we imported previously includes a tokenizer which will allow us to split up our words automatically.

## Step 3 : Testing and Training Split

Once we have our tokens we need to ensure we've made a split in our data before continuing. We need to make sure our testing data is completely separate from our training data.

{% highlight python %}
  def split_data(dataset, split = 0.9):
   test_split = int(len(dataset) * split)
   training_data = dataset[0:test_split]
   testing_data = dataset[test_split:]
   return training_data, testing_data
{% endhighlight %}

I've set the split to 90% training and 10% testing to start with, but made this a passed parameter into the function so that we can vary this later to optimise.

## Step 4 : Compiling Dictionaries

Now that we have our split tokens, we can start assembling some dictioanries to hold all the words used in the dataset and the number of times they occur. This needs to be performed for 

{% highlight python %}
 for word in tokenised_comment:
   dictionary[word] = dictionary.get(word, 0) + 1
{% endhighlight %}

This needs to be run for each of the training sets, for each of the hypotheses we're testing. 

Our dictionaries will end up looking something like : `{"hello":100, "computer": 93, "tech": 74, "support": 45, etc...}`

One final calculation to do on our dictionaries is to calculate the total number of tokens in each set. This will be essential for calculating the probabilities we're working with, and it'll be more efficient to calculate this once at the start, rather than every time we need it.

{% highlight python %}
  for key in dictionary:
    total += dictionary[key]
{% endhighlight %}

## Step 5 : Naive Bayes Classification

The process for predicting which category a sentence belongs to is fairly simple. 

1. Tokenisation and pre-processing.

The testing data now needs to be compiled into tokens using the exact same method used for the training. We don't need to make dictionaries for the testing data.

2. Probability of the Sentence given the Hypothesis P(S|H).

For each sentence, we calculate the probability of that sentence appearing in each hypothesis. This is calculated as the product of probabilities for each word appearing in the dataset.

For example, the word "hello" appears 100 times out of the 10,000 words in the dataset, so that is a 0.01 probability of the word "hello" given the dataset.

{% highlight python %}
   for key in dictionary:
      hypothesis1_probability *= ((hypothesis1_dictionary.get(key, 0) + 1) / hypothesis1_total)
      hypothesis2_probability *= ((hypothesis2_dictionary.get(key, 0) + 1) / hypothesis2_total)
{% endhighlight %}

3. Probability of the Hypothesis P(H).

The probability of Hypothesis A is calculated as the sum of all tokens in hypothesis A, divided by the sum of all tokens across all hypotheses.

{% highlight python %}
  hypothesis1_probability *= (hypothesis1_total / (hypothesis1_total + hypothesis2_total))
  hypothesis2_probability *= (hypothesis2_total / (hypothesis1_total + hypothesis2_total))
{% endhighlight %}

We can multiply this in at the end to get our final probability.

4. Multiplication and Softmax

Once we've finished the product of all the probabilities, we can figure out which hypothesis is more likely.

{% highlight python %}
  if (hypothesis1_probability > hypothesis2_probability):
    return 'Hypothesis1'
  else:
    return 'Hypothesis2'
{% endhighlight %}

## Testing and Analysis

With our 10% testing split, we ran through the naive bayes for all 271 remaining tagged comments and found an accuracy of 72%. This means that just shy of 3/4 of the sentences were being correctly tagged by the Naive Bayes model. Despite having a relatively low accuracy, we did find a higher precision at 80% and a recall of 82%.

These results were less than optimal so I ran a quick test by varying the testing / training split to check for a change.

<img src = "https://imgur.com/O7f0Tv5.png" height = "200px">

As seen in this graph, the accuracy really peaks at this 72% seems to be the highest we get to in terms of accuracy, at an 85% split between training and testing, though I'd suggest that the 90% resuylt is better here due to the higher recall meaning less false negatives.

As a further test, I tried substituting the stemming for lemmatisation.

<img src= "https://imgur.com/38xTD8x.png" height = "200px">

Lematisation looks to have slightly increased the recall at 80%+ trainign data, but seems to have had a negative effect on the Accuracy and Precision of the data.

## Evalutaion + Future Testing

I'd suggest that the best way to improve this system will likely just be the inclusion of more varied data. Our total dataset contains 2700 items, which may sound like a lot, but is actually the lower end of typical AI systems. I expect that increasing this number will result in a more varied data and therefore better generalisation. Furthermore, our training data contained around 30% data for the Negative Hypothesis (bad comments), and 70% data for the Positive Hypothesis (good comments). While the bayesian model should be able to work around this (with the P(H) inclusion in the calculation), I suspect that a closer to 50:50 split of good to bad comments would result in an increased accuracy.


# Option 2 : Use an LLM

A big reason I'm making my own system is due to data sensitivity and security - the data is all internal and confidential which is why everything I'm writing and developing must be used locally. I wouldn't be able to just paste a CSV into chat GPT and ask it to analyse them all - but what stops me from hosting an LLM locally?

I've previously gotten Ollama installed locally and running the 12b parameter model of Gemma, so my plan is to set this up as a service so that I can use a script to ping it with comments and get an analysis back.

## Step 1 : Ollama 12b Parameter Model Requirements

Installing Ollama is easy enough, but I've chosen the 12b parameter model for a reason - it's the most powerful model my machine can run without breaking. The 12b parameter model requires around 8Gb of RAM to run which is the main limiting resource for my machine. While I'd love to use the 27b parameter model, I'm afraid that my machine just won't handle it - obviously this is a great option for future development.








<img src="https://imgur.com/vX0QSzf" height="200px">

{% highlight python %}
{ "Ground": 458, "sky": 728, "Cobblestones": 78, "Hedge": 133, "Bench": 56, "TrashCan": 21, "Streetlamp": 26 }
   At: res://characterHandlers/playerCameraHandler.gd:29:get_objects()
{% endhighlight %}