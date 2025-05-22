# Outfit Styling with Transformers

## Project Motivation 

Outfit styling, although a creative process, has many different rules and patterns that can be automated using a neural network. Generating a compatible outfit can be boiled down to a next-token prediction task, where given a set of some clothing items, we can ask, "What's the next clothing item that will maximize this outfit's compatability?"

<img src="https://github.com/hpmavani/hpmavani.github.io/blob/main/images/next-token-flow.png?raw=true" style = "width:75%"></img>

The ideal vision for this project is that the model is able to pick up on the subtleties of color, pattern, and texture combinations as well as other implicit features that determine outfit compatability and use this to predict the next "token." 

To achieve this, the model will be fit with a training dataset of about 18,000 compatible outfits with over 100,000 clothing items. Each clothing item will have a textual description, a category ID, and price. However, for simplicity, we will only consider the textual description. Additionally, category ID refers to the general category of items the clothing item belongs to ,e.g., shoes, handbags, t-shirts. This information will only be used for analyzing the distribution of items in the dataset and primary data visualization.

## Tokens? 

With the diverse array of clothing item descriptions, we can make a massive vocabulary for our model with tokenized clothing items. However, if we truly want token -> clothing description, the tokenizing and embedding must be done from scratch and we can't take advantage of in-built embedding models within pre-trained models such as GPT-2. This approach would be like training a transformer model from scratch which provides a lot of flexibility but is also tasking in terms of compute. 

Because of these drawbacks, I will initially use GPT-2 as a base model for this task. GPT-2 is a natural language processing model freely available through HuggingFace's Transformers library. GPT-2 has already been fine-tuned for next-token prediction tasks and has its own built-in tokenization and embedding algorithm. To fit GPT-2 to the corpus of outfits, one glaring issue is that GPT-2's definition of a token is drastically different than our definition of a token, which is actually a group of tokens according to GPT-2. 

For example, when we have a description like "topshop moto joni high rise skinny jeans", GPT-2 sees this as:
> "topshop \<sep> moto \<sep> joni \<sep> high \<sep> rise \<sep> skinny \<sep> jeans."

If we asked GPT-2 to predict the next clothing item, it will process 7 different tokens and give an embedding to each without capturing the semantic information of these words being part of a phrase. Additionally, it won't learn that its outputs must be sequences of tokens that resemble a clothing item. This is not what we want!

Essentially, we want to perform a next-phrase prediction, so how can we accomplish this with a model that is made for next-token prediction?

Simply, we will have a preprocessing layer before we even fit the training data to GPT-2, where we will place a delimiter between clothing item descriptions. If we have "topshop moto joni high rise skinny jeans" and "joy denim jacket", this will be fed into the tokenizer as: 

> "topshop moto joni high rise skinny jeans \<sep> joy denim jacket"

At first, the model will treat \<sep> as just another token, but with more repetitions, it will learn that \<sep> has a special meaning as a boundary token.

<img src = "https://github.com/hpmavani/hpmavani.github.io/blob/main/images/bert-tokenization.png?raw=true" style = "width: 75%"></img>

The idea of having the model learn phrase boundaries using a separation token is used within the paper "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding"; Figure 2 of this paper is shown above. 

> "Sentence pairs are packed together into a single sequence. We differentiate the sentences in two ways. First, we separate them with a special token ([SEP]). Second, we add a learned embedding to every token indicating whether it belongs to sentence A or sentence B."

Instead of sentence pairs, this specific task will include clothing items represented as phrases or a sequence of tokens. GPT-2's tokenizer doesn't use segment embeddings so we can omit the second part involving embeddings that distinguish different clothing items.

## Course of Action

This project will cover many different aspects of the ML pipeline such as: 

1. Exploratory Data Analysis (EDA)
    * Using techniques like co-occurence matrices and PCA to identify patterns in the data as well as visualizing these patterns using scatter plots and word clouds.
2. Data Preprocessing 
    * Cleaning textual data by removing punctuation and stop words (irrelevant words)
    * Removing null values
    * Creating test and validation sets that fit this task. 
    * Additional preprocessing of clothing descriptions for GPT-2 tokenization.
3. Model training
    * Fitting the training data to a base model
    * Evaluating base-line performance
4. Hypertuning & Feature Engineering 
    * Revisiting the training data if there is "bad data"
    * Engineering new features to improve the model's performance
    * Tuning parameters such as learning rates, batch size, and training epochs
    * Applying regularization techniques such as weight decay and dropout
5. Final Evaluation on Test Set
    * Analysis of error and results through accuracy metrics and qualitative means
    * Future improvements -- would image inputs improve the model?
