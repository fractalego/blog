---
layout: post
title:  "Representing dialogues"
date:   2017-05-10 00:18:39 +0100
categories: chatbot
---
Representing dialogues
============================================================

For decades there has been a lot of interest in building
chatbots. Natural language dialog generation is nonetheless in its
infancy.

Traditional approaches use a rule-based engine to react to a user's
input. These methods allow the programmer to design the behavior of a
chatbot according to specific patterns. For example, a formalization
of this techique can be found in the script language
[AIML](http://www.alicebot.org/aiml.html). The main problem with
rule-based systems is that they are about as good as the time invested
into creating the rules.

A more recent approach has emerged where the reaction to a user's
input is understood as a translation problem: the query is
'translated' into a reply. This approach leaves room for learning the
proper replies from a corpus of conversations.

An example of this approach has been pioneered by Lee and Vinalis in
[this article](https://arxiv.org/abs/1506.05869), where a sequence to
sequence model is employed for the translation mechanism. The sequence
to sequence model employes an 'encoder' that transforms a sentence
into a vector. This vector is then fed into a 'decoder' which
transforms it into a proper natural language reply.

In order for this approach to be successful, the vector sandwiched
between the encoder and the decoder should contain information about

1. The Semantics of the sentence (in particular entities and relations)
2. The pragmatic content
3. The general topic of the conversation
4. The intention of the user, which can be a long term goal beyond the
current sentence

Encoding all this information in a low-dimensional vector with a
single RNN is a tall task, possibly unfeasible. For this reason
[hierarchical structures](https://arxiv.org/abs/1701.07149) of
encoders-decoders have been proposed as an improvement to the original
article

One of these approaches is going to be successfull eventually. In this
page we want to play with the high level structure of a conversation.



The space of all sentences
--------------------------

In the following we use the movie conversations available in the
[Cornell Movie
database](https://www.cs.cornell.edu/~cristian/Cornell_Movie-Dialogs_Corpus.html).
All the movie lines that appear here are taken from that dataset and
are used under the [terms of the
dataset](https://www.cs.cornell.edu/%7Ecristian/memorability_files/README.v1.0.txt).

All the sentences of the corpus can be translated into vectors, for
example using the excellent doc2Vec model in
[gensim](https://radimrehurek.com/gensim/).  More refined methods
would not add much to the coarse-level description used here.

Let us represent these vectors using t-SNE with low perplexity (here
we used 5).

![Image with all the vectors](/images//vectors.png)

A granular structure appears (effectively by construction upon using
low perplexity). These 'grains' are clusters of similar sentences,
like 'Goodbye!', 'Goodbye, my friend!', ' Goodbye, my love, a thousand
times goodbye.'

Let us take a look at the first ten lines of the corpus:

```txt

A: Colonel Durnford... William Vereker. I hear you 've been seeking
 Officers?
B: Good ones, yes, Mr Vereker. Gentlemen who can ride and shoot


A: Your orders, Mr Vereker?
B: I'm to take the Sikali with the main column to the river


A: Lord Chelmsford seems to want me to stay back with my Basutos.
B: I think Chelmsford wants a good man on the border Why he fears a
flanking attack and requires a steady Commander in reserve.


A: Well I assure you, Sir, I have no desire to create difficulties. 45
B: And I assure you, you do not In fact I'd be obliged for your best
advice. What have your scouts seen?


A: So far only their scouts. But we have had reports of a small Impi farther north, over there. 

```

This dialogue describes a path in the abstract space of sentences.

![Image with a path](/images/path.png)


Clustering sentences
--------------------

Every dialogue appears as a path in the space of sentences. All these
paths would together constitute a _network_. In order to generalize the
paths, we can cluster together all the 'grains' in the previous
picture.

![Image with the clusters](/images/clusters.png)

Each of these red balls is the centroid of a cluster of ~300
sentences. After clustering, one can build the network of all
sentences by drawing an edge between nodes that have a sentence that
follows another one. The result is the rather horrible picture

![Image with the network](/images/network.png)

This network is a connected graph, i.e. there are no isolated
nodes. Note that this clustering needs to be done to create a
connected graph in the sentence space. Without the clustering of
sentences every node would be a sentence unique to a specific
conversation.



Generating dialogues of two people in a hurry (computational comedy)
---------------------------------------------------------

Given a starting point and an end point, two people will naturally
speak trying to end the conversation as soon as possible.

According to the network representation, this means that the two
people will try to reach the end of the conversation using the
shortest path. One example is the following (generated) dialogue.


```txt

Alice:  Hello! How are you?
Bob:  I should have listened to my wife. She said you were up to no good. Gossip.

Alice:   Yeh, that'd be real nice of ya, Mr. Lonneman.
Bob:  I can explain about the portal, darling.

Alice:   Thanks
Bob: Goodbye!

```

The corresponding path is

![Image with a path](/images/chat_path.png)

The sentence number 1 is 'Hello! How are you?', whereas number 5 is
'Goodbye!'. It is interesting to see that these two sentences are
nicely projected at opposite sides of the map.

All the lines in the dialogue are taken from the Cornell movie
dataset. No single sentence is original. What is presentented here is
simply a re-arrangement of the sentences according to the following
algorith:


```txt
1. Read an input line and a goal sentence ('Hello!', 'Goodbye!').
2. Find the node in the sentence network that is closest to the input line.
3. Find the shortest path between this line and the goal.
4. Find the next vertex in this path
5. Choose randomly a movie line belonging to this next vertex (each vertex
represent a cluster of ~300 lines)
4. Repeat with this last line as an input upon keeping the same goal
sentence.
```

There is an additional random component: Associating a node to an
input line is a noisy process. Given the same initial and ending
sentences there can be also variations in the number of nodes, as the
following examples show:


```txt

Alice:  Hello! How are you?
Bob:  How come nobody knows about him? And you do?

Alice:   You kids are making too much damn noise.
Bob:  I dont recollect mentioning a friend.

Alice:   Leave him alone, he's got swine fever.
Bob:  Indisposed, sir.

Alice: Goodbye!


---


Alice:  Hello! How are you?
Bob:  I'm serious, man. Don't mention it. Ever.

Alice:   Gotta be one of his disciples or something.
Bob:  Leave me alone! He's going to kill me!!

Alice:   Well, I held it like an egg.
Bob: Goodbye!


--


Alice:  Hello! How are you?
Bob:  Take a look for yourself.

Alice: I'm dying, Claudia.  I have cancer. I have cancer and I'm
dying, soon. It's metastasized in my bones and I --
Bob:  You're not contagious are you?

Alice:   They guys here aren't going to be able to hold out until battalion shows up.
Bob:  Right.  Bombardier, to me please.

Alice: Goodbye!


---


Alice:  Hello! How are you?
Bob:  I'm Jill. I really like your show. I think you're great.

Alice:   I had to come.  To be sure you were okay.
Bob: Just everything.  You.  California.  The beach.  This spot right
here.  I feel like I belong here, you know?  It just feels right.

Alice:   The friend of John's that was staying at your house.
Bob:  Leave him alone. She's his mother, not yours.

Alice:   Gail, please.
Bob: Goodbye!


---


Alice:  Hello! How are you?
Bob:  You remind me of me when I was...I guess I was never like you. So cute. So questioning.

Alice:   When can you leave?
Bob:  You don't look angry.

Alice:   It's not our mistake!
Bob:  Why, you bully. I believe you would.

Alice:   Well, goodbye.
Bob: Goodbye!
```


Conclusions
-----------

These previous examples do not quite make sense. They are made of
lines taken from different movies, and they refer to different
entities. However there are glimpes of logic in the conversation: When
Alice speaks about being ill Bob is worried about contagion. Some of
the sentences before 'Goodbye' can be seen as a conversation stopper
(especially in the last example). If you squeeze your eyes and just
consider the _sentiment_ of the conversation you might see a meaning
in those lines.

Sentiment is of course not enough for a reasonable conversation. This
structure of dialogues should be refined upon using more sofisticated
instruments.


Code
----

The code to reproduce these results is on my [github
page](https://github.com/fractalego/chatbots-dialogues-test).