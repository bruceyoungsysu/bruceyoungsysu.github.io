---
title: Twitter Bot Based on RNN
Date: 2019-01-20 16:34:09
categories: 
- RNN
tags: NLP, RNN
---

Diaglogue system is one of the key areas in deep learning NLP as a good dialogue system will provide a good portal for human-computer interaction, such as online chatting robot that replying consults of a customer. For companies, this will certainly cut the running cost. There are also many other interesting topics, like imitating someone tone to compose a line, even write a book, or analyzing a paragraph of text to  extract the key information.

# How to Model Language

The most unique feature of modeling language is that the relation between text components is temporal, which means the current input of word is certainly related to previous words you have said and of course also the later words you are going to say. This relation also exits between setences and paragraphs, even in larger scope which we call context. 

Thus in this way a conventional neural network cannot model language, and a new network structure came up: Recursive Neural Network (RNN).  A neuron in RNN is different as it passes the state of neuron at current time step to the next time step. If we unfold the neuron in time dimension, it will look like:

![](https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/trump_twitter/RNN_neurons.png)

(Graph by Nature)

The expansion of neurons is pretty much alike we write down a sentence (which is temporal) on a page of paper(which is spatial). Thus we can obtain a sequence of outputs, which is the respond our network. According to different types of inputs, we can build models on different levels, say character level, word level and sentence level. We can image the input is a vector representing whatever user is interested in like a word：

![](https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/trump_twitter/RNN_structure.png)

(Source: https://docs.google.com/presentation/d/1QydMhsGFeUzDYZr7dV0tt9WJryleOoKUyJth4ftRDbA/edit#slide=id.g1ce324a1fd_0_142)

To longer interaction length between neurons, there are some other transformation forms of RNN developed sunch as LSTM and GRU.

# RNN Configuration

To define the problem, there are three main steps:

1. Define the scope of problem and get the data.
2. Transform the your input from language level to vector level.
3. Construct the layers of RNN and descide how to use the output.

RNN problem can be as complex as the most complicate language problems, thus shrinking the problem scope is initial due to the resources and time constrain. Imitating someone's style is relatively simple as the corpus bank will be small and it is easier to catch the style. And meanwhile the dataset will be much easier to get.  For instance, if we only want to tweet in someone's tone, we only need to care about the tweets he/she has composed. But if we want to make an agent with general idea like Siri, it will be much more complex.

After decided the problem you are interested in, the next thing is to collect a the corresponding dataset, and prepare the set to vectorize your input. In our case we are going to build a character level model, thus the vocabulary is set as 

```python
self.vocab = ("$%'()+,-./0123456789:;=?ABCDEFGHIJKLMNOPQRSTUVWXYZ"
                    " '\"_abcdefghijklmnopqrstuvwxyz{|}@#")
```

But if we are going to build a word level model, things will go pretty complex as we need to figure out how to deal with words with no actual meaning but apears everywhere in our speech like 'a' and 'the'. We can vectorize the input by one-hot encoding according to the vocabulary. When infering the prediction, the output also needs to be decoded to characters in order to form words and sentences.

The next step we are going to build a GRU RNN network. For hidden layers, there two dimensional parameters to specify. We need to know how many neurons in each hidden layer and how many hidden layers in the network. Another different thing in RNN is that besides placeholders for input, you will also need a placeholder for the state in hidden layers and initialize them properly. In our case of twitter robot, we can set all initial state as zero.

```python
def create_rnn(self, seq):
    layers = [tf.nn.rnn_cell.GRUCell(size) for size in self.hidden_size]
    cells = tf.nn.rnn_cell.MultiRNNCell(layers)
    batch = tf.shape(seq)[0]
    zero_state = cells.zero_state(batch, tf.float32)  # first param : batch size
    self.in_state = tuple(tf.placeholder_with_default(state, [None, state.shape[1]]) for state in zero_state)  # initial state
    length = tf.reduce_sum(tf.reduce_max(tf.sign(seq),2),1)
    self.out, self.out_state = tf.nn.dynamic_rnn(cells, seq, length, self.in_state)
```

Another thing needs to keep in mind is the loss function. Here we use `tf.nn.softmax_cross_entropy_with_logits` between the input sequence and the sequence a time step further than the input as loss function. This means what we want the network to learn most is to predict the words in the next state in time dimension.

In the setting of RNN, there is another hyperparameter called `self.num_steps = 50` which defines basically the window size of input, i.e. the a character is only affected by the 50 ones come before and after it. `self.gen_length` is the desired output length of composed text, which is used in inference to descide when to terminate the decoding inference process.

The full code is here: https://github.com/bruceyoungsysu/twitter_bot

Some interesting sample output:



