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

## Step 1 : Hosting an Ollama Instance

Since I began this experiment, I have had an project to set up an in-house Ollama instance which is capable of running larger models. While I originially intended to run this on my homelab as a test with the potential of requesting some moderate hardware to run this on our work infrastructure, I now have the capacity to run this on our newly installed Gemma 3 27b parameter model.

The instance is hosted on its own Virtual Machine with Ollama running as a service from NSSM. Due to the nature of our infrastructure, I can not share information about the specific configuration of our instance.

## Step 2 : Prompt Engineering

While writing a Bayesian classifier allowed me to change various parameters, the LLM I'm using doesn't grant me as much freedom in this area without fine-tuning the model. Instead the specific adjustments to improve performance comes with proper prompt engineering.

Good prompt engineering requires effective choice of context, examples, and constraints so that the LLM can provide you a response which fits what you're expecting with a good degree of accuracy.

### Constraints

Constraints are going to be the easiest part of my model. My output needs to be very rigid, so my constraints can be very explicit.

{% highlight console %}
Could you classify this as a 'good' or a 'bad' comment?
Please answer as a single 'yes' or 'no', followed by a 1 sentence long justification of why you think it is good or bad.
{% endhighlight %}

This prompt allows the LLM to provide an output correctly in 100% of my tests. With the text in this format, it makes it very easy for me to parse the result into a boolean based on whether the prompt starts with 'yes' or 'no' and then split out the justification to include in an output.

Though without any context provided, these results were meaningless.

### Context

Providing context is the next most important step in a meaningful and effective prompt. With just a comment and a question of 'is this good?' the LLM has no indication of what makes a good or bad comment.

As I've described above, identifying a comment as good or bad is fairly subjective, but it's not arbitrary - there is an amount of human review where we can identify if a comment is obviously terrible, and we also have a list of requirements for internal and external comments. 

{% highlight console %}
We categorise an internal comment as 'good' if it meets the following requirements:
. you have provided specifics of which commands you have run, where you've run them, and what the output is
. You have included snippets of any relevant logs and filepaths to them
. You have included error codes and steps on how to reproduce them
. You have provided a conclusion, opinion, or next action based on your findings.
{% endhighlight %}

There is an equivalent list of requirements for external comments which we can provide in a separate prompt. Then, when processing through comments to determine if they are 'good' or 'bad' we can ensure that we are sent to the LLM with the correct prompt based on their visibility.

When executing the prompt with this context, the LLM was able to return 41 out of 300 test comments with the correct status, suggesting that this is not sufficient context. Reviewing the results, I noticed that the LLM was frequently returning comments as bad when we had reviewed them as good. This was typically because it was expecting details of commands, snippets of logs, and error codes all together. 

I attempted to extend the prompt to include more context, specifying that each of these requirements was on a 'where applicable' basis, and including the line:

{% highlight console %}
When analysing internal comments, the primary goal is to provide useful and quality communications to other members of the internal team, and make it easier to review work done in the future when it comes to writing guides.
{% endhighlight %}

With this extension, results increased to 56 / 300 randomly sampled tickets.

After making further changes with an attempt to reduce the focus on each of the requirements and work on a broader and more general level, I eventually opted to remove the bullet points in place of an explanation of what the function of an internal support team is and why you would provide internal or external comments.

{% highlight console %}
An internal comment is one which is only visible to other members within the support team. These users are also support analysts and engineers, so they will have a deeper technical knowledge than external users.

Internal comments are designed to keep other members of internal support up to date with what is happening on a ticket and include details that external users may not understand or care about.
{% endhighlight %}

This change allower the tickets to be categorised correctly just over 50% of the time (159/300 tickets categorised correctly).

### Examples

To increase the accuracy of the system further, I started to include some specific examples in my prompt. Some specific issues that came up in my testing included comments from the LLM such as:

{% highlight console %}
While the comment details actions taken, it's poorly formatted and includes a broken/messy URL within the text, hindering readability and making it difficult to quickly understand the key information for other support team members.
{% endhighlight %}

This seemed to happen quite often, where certain comments were getting flagged as bad due to the way the urls, tables, and images get formatted when pulling the HTML comment directly from the database. To resolve this issue, I've added a line to specify these exact example and ask the AI to disregard this in its analysis of the tickets.

{% highlight console %}
Note that this comment is parsed from an HTML output so may contain odd formatting, HTML tags, etc... Some URLs may also become malformed. Please disregard these artifacts in your analysis.
{% endhighlight %}

Some other specific examples that seemed to flag quite often were cases where an External Comment contains quite more technical information - this is because cases like ticket escalation (to other users as well as to other teams and disciplines) appears as an External comment. These specific cases will need to include more technical detail so that the higher level of support team or other internal teams will be able to understand the complexities of what has been performed. I have included some examples of these cases in the prompt so that the LLM can attempt to avoid them.

{% highlight console %}
Sometimes internal communication is posted as an External comment, this includes Triages (i.e. queueing of tickets), escalations (i.e. <comment snippet redacted>), moving tickets to other disciplines (i.e. <comment snippet redacted>) and assignments (i.e. <comment snippet redacted>). In these cases, a ticket may contain more technical detail, but would still be marked as a good external ticket.
{% endhighlight %}

Even after this, the LLM was flagging a lot of tickets as 'bad' due to minor gramatical issues. Some small mistakes like using the wrong there / their, or use of incorrect conjugations were flagging as bad tickets despite the content being very good. A large amount of these issues are due to the Polish members of our support team who speak English as a second language. I've included an additional line in the prompt to indicate that part of our support team does not speak english as a first language so some level of lenience is needed when it comes to reviewing the specific grammar.

{% highlight console %}
Some members of our support team speak English as a second language, so some amount of grammar and spelling mistakes should be forgiven, though large mistakes which may confuse readers should still be flagged.
{% endhighlight %}

## Analysis

The Bayesian classifier was able to identify ticket comments as good or bad with a 72% accuracy. With an LLM I am only able to achieve a 66% accuracy at classifying ticket comments. 

While this number is definitely lower, the system has some advantages. Firstly, while the accuracy as a whole is lower, the LLM has extremely good precision, with only 2 of the tickets flagged as 'good' being false positives. This means that using this system to provide scores may provide more of an incentive to write good comments without re-inforcing negative behaviours.
Secondly, the LLM provides a sentence long summary explaining why it has given that specific status. This means that when returning the stats to users, they are able to understand why they have received the stats they have. Tickets are given explanations such as : 

{% highlight console %}
This is a good comment because it directly provides a link to relevant documentation that will help other support team members quickly access helpful information.

The comment consists solely of an image reference and provides no textual information about the issue, steps taken, or diagnosis, making it unhelpful for internal team communication and future guide creation.
{% endhighlight %}

Since I am not a trained prompt engineer, I'm sure there are significant improvements which could be made to the prompts I have written.

# Conclusion

Between these two systems, the LLM has a slightly worse accuracy, but overall I still think it is a better approach between the two options. It is able to provide a justification of why a specific status was given and is able to adapt to phrasings better. The LLM can more easily generalise, rather than relying on the use of specific keywords like the Bayesian classifier does.

Additionally, the higher precision of the LLM system means that it will enforce good behaviours without incorrectly marking bad behaviours as good.

The remaining comments which flag incorrectly in the LLM model are primarily due to comments such as escalations, de-escalations, and assignments containing a larger amount of technical inforamtion which the LLM thinks is inappropriate for a customer-facing comment - even if that comment is not directly targetted at a customer. It may be possible to work around this, but my speciffic prompts have not been able to work around this.

## Garbage In; Garbage Out

The most important principle of AI training is that bad input data means bad output data. This principle likes plays into the performance of the Bayesian classifier, but less so when it comes to the ticket comments.

I still think this principle may affect my testing results for the LLM system. For example, an external comment containing just a table of items and costs wit a message saying "can you please accept the costs. Thank you." is marked in my testing data as a 'good' comment. The LLM flags this as bad saying it lacks context and info about what this is supposed to represent, or how the assignee is supposed to interperet the table. I would agree with the LLM in saying that this is a 'bad' comment.

As such, I think the accuracy of both of these systems may be misrepresented when tested.

## Formatting Issues

The other issue which I think will have impacted both of these systems is bad formatting. The comments extracted from the Jira Database include HTML formatting tags on tables, URLS, and visual formatting for bold, itallic, and pre-foramtted blocks. These tags interfere with both systems.

The inclusion of a table HTML tag, for example may be identified as a 'good' characteristic in the Bayesian classifier. This means that even bad comments might get flagged as good because they contain tables, even if those tables are meaningless or contain useless information and no context.

This formatting seems to confuse the LLM quite often too, identifying fairly simple tables as complex technical data due to the HTML tags, spacings, and such. This makes it flag potentially good comments as bad due to what it percieves is technical info.
