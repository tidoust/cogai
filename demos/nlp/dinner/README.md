# Dinner Demo

This is a natural language demo for a dialogue between a customer and a waiter at a restaurant. The demo is based around the following sample dialogue:

<blockquote>
C: Good evening.<br>
W: Good evening and welcome.<br>
C: A table for one please.<br>
W: Certainly. Just here?<br>
C: Could I sit by the window?<br>
W: I'm sorry. The window tables are all reserved.<br>
C: This table will be fine.<br>
W: Are you ready to order?<br>
C: Yes. I'll have tomato soup for starters.<br>
W: A tomato soup. What would you like for the main course?<br>
C: I'll have the plaice.<br>
W: I'm afraid the plaice is off.<br>
C: Oh dear. What do you recommend?<br>
W: The steak pie is very good.<br>
C: That will be fine.<br>
W: Would you like anything to drink?<br>
C: Yes, a glass of red wine.<br>
W: Is that all?<br>
C: Yes, thanks.<br>
W: Here you are.<br>
C: Thanks, that looks great!<br>
W: You're welcome!<br>
C: The bill please.<br>
W: Here you are.<br>
C: Can I pay with a credit card?<br>
W: Certainly.<br>
C: Thank you and good bye.<br>
W: Good bye, hope to see you soon.
</blockquote>

*Adapted from the [Just Good English](https://justgoodenglish.com/eating-out/) website.*

The demo involves separate cognitive agents for the customer and for the waiter, that run within a web page, where the dialogue is shown as a text chat. The demo combines declarative and procedural knowledge about typical behaviour for having dinner at a restaurant. The natural language is handled word by word, avoiding backtracking, and using concurrent syntactic and semantic processing. The dialogue text is preprocessed to replace "I'll" with "I will" and "I'm" with "I am", etc., before being coerced to lower case, stripping out punctuation and splitting into an array of words. This mimics hearing as compared to reading, along with common abbreviations.

Each agent takes its turn to speak, mapping a model of its communication intent into text, and when listening, mapping the text it hears back into an internal model of the meaning. The communication intent is determined by the current state of execution of the dinner plan. This corresponds to:

1. Greetings and welcome
2. Finding a table
3. Reviewing the menu
4. Placing an order
5. Thanking the waiter
6. Asking for the bill
7. Paying the bill
8. Farewells and please come again

Each step supports sub-plans with variations, e.g. when a window table or a specific dish is unavailable. In principle, a random number generator could be used to select between the variations when executing the dinner plan. The plan is based upon a causal model, and future work could explore how an agent can revise the plan to suit changing circumstances.

One way to represent plans is as a sequence of goals:

```
# plan with info on when it is applicable
plan p1 {purpose dinner; at restaurant, start s1}
dinner-plan s1 {goal greetings; next s2}
dinner-plan s2 {goal find-table; next s3}
dinner-plan s3 {goal review-menu; next s4}
dinner-plan s4 {goal place-order; next s5}
dinner-plan s5 {goal thank-waiter; next s6}
dinner-plan s6 {goal ask-for-bill; next s7}
dinner-plan s7 {goal pay-bill; next s8}
dinner-plan s8 {goal farewells; finished p1}
```
Each goal may have subgoals, e.g. to find a table, indicating that you have a reservation and giving your name, or stating the number of places you need and your preferences for the table location. This can be handled via either a sub-plan or a set of rules that can adapt to the variations. The act of having dinner is a situation associated with a particular context in episodic memory. The reservation, number of people and the table preferences will depend on that context, e.g.

```
# 7pm reservation under the name "Smith" for a party of 2 at the Rose and Crown
dinner {@context c1; where rose-and-crown; party-size 2; reservation smith; time 7pm}
```

### Speech acts

Apart from greetings and farewells, the dialogue consists of requests and responses that deal with each stage of the dinner plan. Negative responses trigger the search for an alternative, e.g. to accept the table the waiter originally suggested, or to pick a different dish if your first choice isn't available. The menu lists different dishes for each course. The demo could involve the customer reading through the menu, or that could be made implicit with the choices already placed into declarative memory. The customer could mentally review the choices to rank them based upon some preferences, and make a random selection if the top choices are equally ranked. This process updates the facts in the declarative memory.

The literature uses terms such as locutionary, illocutionary and perlocutionary, which are rather inscruitable. I prefer the following terms:

* *Assertion* - that something is the case, e.g. apples are a kind of fruit
* *Question* - a request for information, e.g. are you ready to order?
* *Answer* - a response to a question, e.g. yes, I would like tomato soup for starters
* *Command* - an instruction to do something, e.g. to a child: don't chew with your mouth open

Questions can either ask if something is true or not, e.g. can I pay with a credit card, or to ask for further information, e.g. what would you like for starters? Yes/no questions are often answered with additional information on the assumption that for a positive response, that is what the questioner is seeking, or to provide some alternatives when giving a negative response.

A dialogue can be represented as a sequence of speech acts where each utterance refers to the previous one, as well as to the dialogue plan. This allows semantic processing to access the dialogue history and its context in respect to the plan. Each utterance can be modelled in terms of its overt meaning, and other aspects relating to the context and social etiquette.

### Social etiquette and pragmatics

Social etiquette needs to be adhered to during the dialogue. The customer and the waiter are peers in social standing. This should be acknowledged at the start and end of the dialogue, e.g. by adding a "please", through the use of conditional verbs that signify that you realise that a request may be declined, and when answering a yes/no question with "yes please" or "no thanks". In the middle of the dialogue you can be more direct, e.g. "I'll have" rather than "I would like". When declining a request, the waiter apologises and offers an explanation, e.g. "the tables are all reserved" or "the plaice is off" (as in off the menu). The customer should acknowledge the discomfort this brings to the waiter when having to give bad news, e.g. "oh dear" or "that's a pity". The example dialogue goes further, as the customer asks the waiter for advice, which is a way of making the waiter feel to be of greater value in the dialogue. However that isn't needed if the customer already has a second choice in mind.

## Knowledge representation

The customer will have preferences, e.g. for a window table, and for specific dishes on the menu. Ideally, this would be expressed declaratively and used to make requests, and to make a choice from the options on the menu. That suggests the demo also needs a way to model the menu and a means for the customer to review the options and make a choice. Each utterance in the dialogue will be chosen based upon the current stage in the dinner plan and the information in the dialogue history. This involves mini-plans to cater for variations, e.g. whether not a window table is available, and whether a given dish is off the menu.  I need to wok on how to represent such mini-plans in terms of declarative and procedural knowledge.

What is a good enough way to represent statements like *The window tables are all reserved*?  From a logical point of view this is equivalent to:

```
for all x such that table(x) and by-window(x) then reserved(x)
```

One way to do that would be to devise a means for using chunks for expressions in first order logic, but not every English utterance has a natural interpretation in first order logic. Another option is to use a chunk representation of the natural language statement, for instance:

```
vp v1 {verb be; tense present; subject np1; object np2}
np np1 {det the; noun window, table; number plural}
np np2 {det all; adj reserved}
```

If we have a restaurant with a set of tables, some of which are at the window, we can then identify which subset is being referred to from the subject noun phrase. However, we don't need to do that in the dialogue!  The dialogue implies that if you want a window table in future, then you will need to reserve one in advance, but that goes well beyond what this particular demo needs to deal with.

A choice of dish for a given course, or a choice of drink could be represented as follows:

```
course {@context c1; starter tomato-soup}
course {@context c1; main plaice}
course {@context c1; main steak-pie}
drink {@context c1; wine red; quantity glass}
```

where @context is used to declare the context in which these particular facts are true, i.e. a particular dinner at a particular restaurant on a particular day.

The next challenge is to identify a way to represent speech acts, their semantics and pragmatics.

Here is a sketch of the chunks used to represent the sequence of utterances. Each utterance identifies who spoke it, gives the syntactic structure used, the previous utterance, and the associated stage in the dinner plan. The verbs are left as is temporarily. A following step will be to use the infinitive form of the verb along with properties for the tense and related parameters, and similarly for the number and gender properties for nouns.

<details>
  <summary>Click to expand and contract!</summary>
  
```
# c: good evening
greeting u1 {who customer; syntax np1; plan s1}
np np1 {adj good; noun evening}

# w: good evening and welcome
greeting u2 {who waiter; syntax c1; prev u1; plan s1}
conj c1 {word and; left np2; right ex1}
np np2 {adj good; noun evening}
excl ex1 {word welcome}

# c: a table for one please
question u3 {who customer; syntax np2; prev u2; plan s2}
np np2 {det a; noun table; for np3}
np np3 {number one; adv please}

# w: certainly (positive response to request)
ack u4 {who waiter; syntax a1; prev u3; plan s2}
ap a1 {word certainly}

# just here (points to nearby table)
answer u5 {who waiter; syntax ap1; prev u4; plan s2}
ap ap1 {adv just, here}

# c: could I sit by the window
question u6 {who customer; syntax vp1; prev u5; plan s2}
vp vp1 {verb could, sit; subject np4; by np5}
np np4 {pron i}
np np5 {det the; noun window}

# w: I'm sorry (negative response to request)
nak u7 {who waiter; syntax vp2; prev u6; plan s2}
vp vp2 {verb am; subject np6; object np7}
np np6 {pron i}
np np7 {adj sorry}

# w: the window tables are all reserved (explanation)
answer u8 {who waiter; syntax vp3; prev u7; plan s2}
vp vp3 {verb are; subject np8; object np9}
np np8 {det the; noun window, tables}
np np9 {det all; adj reserved}

# c: this table will be fine (the table the waiter pointed to)
answer u9 {who customer; syntax vp4; prev u8; plan s2}
vp vp4 {verb will, be; subject np10; object np11}
np np10 {det this; noun table}
np np11 {adj fine}

# w: are you ready to order
question u10 {who waiter; syntax vp5; prev u9; plan s4}
vp vp5 {verb are; mode question; subject np12; to order}
np np12 {pron you; adj ready}

# c: yes (positive response to request)
ack u11 {who customer; syntax np13; prev u10; plan s4}
np np13 {excl yes}

# c: I'll have tomato soup for starters
answer u12 {who customer; syntax vp6; prev u11; plan s4}
vp vp6 {verb will, have; subject np14; object np15; for np16}
np np14 {pron i}
np np15 {noun tomato, soup}
np np16 {noun starters}

# w: one tomato soup
assertion u13 {who waiter; syntax np17; prev u12; plan s4}
np np17 {number one; noun tomato, soup}

# w: what would you like for the main course
question u14 {who waiter; syntax vp7; prev u13; plan s5}
vp vp7 {verb would, like; mode question; subject np18; object np19; for np20}
np np18 {pron what}
np np19 {pron you}
np np20 {det the; adj main; noun course}

# c: I'll have the plaice
answer u15 {who customer; syntax vp8; prev u14; plan s5}
vp vp8 {verb will, have; subject np21; object np22}
np np21 {pron i}
np np22 {det the; noun plaice}

# w: I'm afraid the place is off
nak u16 {who waiter; syntax vp9; prev u15; plan s5}
vp vp9 {verb am, afraid; subject np23; that vp10}
np np23 {pron i}
vp vp10 {verb is; subject np24; object np25}
np np24 {det the; noun plaice}
np np25 {adv off}

# c: oh dear
answer u17 {who customer; syntax np26; prev u16; plan s5}
np np26 {excl oh; adj dear}

# c: what do you recommend
question u18 {who customer; syntax vp11; prev u17; plan s5}
vp vp11 {verb do, recommend; mode question; subject np27; object np28}
np np27 {pron what}
np np28 {pron you}

# w: the steak pie is very good
answer u19 {who waiter; syntax vp12; prev u18; plan s5}
vp vp12 {verb is; subject np29; object np30}
np np29 {det the; noun steak, pie}
np np30 {adj very; adj good}

# c: that will be fine
assertion u20 {who customer; syntax vp13; prev u19; plan s5}
vp vp13 {verb will, be; subject np31; object np32}
np np31 {pron than}
np np32 {adj fine}

# w: would you like anything to drink
question u21 {who waiter; syntax vp14; prev u20; plan s5}
vp vp14 {verb would, like; mode question, subject np33; to np34}
np np33 {pron you}
np np34 {noun drink}

# c: yes
ack u22 {who customer; syntax np35; prev u21; plan s5}
np np35 {excl yes}

# c: a glass of red wine
answer u23 {who customer; syntax np36; prev u22; plan s5}
np np36 {det a; noun glass; of np37}
np np37 {adj red; noun wine}

# w: is that all
question u24 {who waiter; syntax vp15; prev u23; plan s5}
vp vp15 {verb is; mode question; subject np38; object np39}
np np38 {pron that}
np np39 {det all}

# c: yes thanks
ack u25 {who customer; syntax np40; prev u24; plan s5}
np np40 {excl yes; noun thanks}

# w: here's your order (putting down the food and drink)
assertion u26 {who waiter; syntax vp16; prev u25; plan s6}
vp vp16 {verb is; subject np41; object np42}
np np41 {adv here}
np np42 {det your; noun order}

# c: thanks that looks great
answer u27 {who customer; syntax np43; prev u26; plan s6}
np np43 {excl thanks; that vp17}
vp vp17 {verb looks; object np44}
np np44 {adv great}

# w: you're welcome
assertion u28 {who waiter; syntax vp18; prev u27; plan s6}
vp vp18 {verb are; subject np45; object np46}
np np45 {pron you}
np np46 {adj welcome}

# c: the bill please
question u29 {who customer; syntax np47; prev u28; plan s7}
np np47 {det the; noun bill, thanks}

# w: here it is (handing the bill over)
answer u30 {who waiter; syntax vp19; prev u29; plan s7}
vp vp19 {verb is; subject np48; object np49}
np np48 {adv here}
np np49 {pron it}

# c: can I pay with a credit card
question u31 {who customer; syntax vp20; prev u30; plan s8}
vp vp20 {verb can, pay; subject np50; with np51}
np np50 {pron i}
np np51 {det a; noun credit, card}

# w: certainly
ack u32 {who waiter; syntax np52; prev u31; plam s8}
np np52 {excl certainly}

# c: thank you and goodbye
farewell u33 {who customer; syntax c2; prev u31; plan s9}
conj c2 {word and; left np53; right ex2}
np np53 {noun thank; pron you}
excl ex2 {word goodbye} 

# w: goodbye and hope to see you soon
farewell u34 {who waiter; syntax c3; prev u32; plan s9}
conj c3 {word and; left ex3; right vp21}
excl ex3 {word goodbye}
vp vp21 {verb hope; to vp22}
vp vp22 {verb see; object np54}
np np54 {pron you; adj soon}
```
Notes:

* The British Council provide a [nice summary of English grammar](https://learnenglish.britishcouncil.org/english-grammar-reference/). 
* Verbs can be annotated with *mode* to signal that they are being used for a question rather than an assertion, and hence have the subject in a different position
* Verbs need a way to indicate the tense, e.g. the future tense as in "I will bring flowers" vs the future conditional tense as in "I would bring flowers". However this is implicit given the presence of modal verbs such as "would".
* I am not quite sure whether to handle exclamations separately or as part of noun phrases. It depends on whether the exclamation is considered to be a separate sentence (e.g. "Yes. I would like tomato soup") or part of a following phrase (e.g. "Yes thanks").
* The next step will be to include syntactic parameters such as tense, number and gender
* The examples are missing the semantics and the pragmatics. I will explore those as a further exercise. An open question is whether reasoning operates off the natural language directly or whether we need a rich way to represent natural language semantics including quantifiers. This probably depends on the context. 
</details>

## Syntactic Processing

Each word is mapped to a word sense and part of speech. In most cases in the demo, this is unambiguous, where the lexicon provides multiple possibilities, a graph algorithm is invoked to find the one that best fits the context.

A shift-reduce parser is used to process the syntactic structure of an utterance. This uses a queue as a working memory, together with functions that deal with common structures, e.g. noun phrases consisting of a determiner, adjectives and nouns. Queue entries are reduced when building the chunks that describe the word dependencies in the utterance. Syntactic Processing goes hand in hand with semantic processing, e.g. resolving the meaning of a noun phrase when it is completed. Verbs may have multiple slots as well as the subject and object. These are filled by prepositional phrases. 

Prepositions can in principle attach to the subject, the verb, the object or another preposition. The syntactic processor identifies potential attachment points for evaluation by the semantic processor via queueing goals that trigger rules to invoke graph algorithms. Each choice is given a score, and the best choice is found asynchronously as the thread for each choice completes and reports back to the syntactic processor.

The semantic interpretation is built incrementally as the utterance is processed, culiminating in queuing a goal to decide what action to take.

## Semantic Processing

This can take two forms: the first is the direct invocation of graph algorithms, e.g. to select the best word sense in the current context. The second is where the rule engine is invoked by queuing a goal. The rules this triggers can access and update declarative knowledge, as well as invoking graph algorithms that are designed for natural language understanding or natural language generation.

## Natural Language Generation

Each cognitive agent will have a communication intent based upon the current state of the dinner plan according to that agent. This intent is mapped progressively into a syntactic structure and then into a sequence of words to be spoken.

## Declarative Knowledge

To model the knowledge some mindmaps were prepared as incomplete illustrations of what needs to be represented.

The following diagram describes some of the concepts needed to express a plan for a restaurant dinner.

![dinner plan](https://www.w3.org/Data/demos/chunks/nlp/dinner/images/dinner-plan.png)

The following diagram describes some of the concepts used in the dialogue between the customer and the waiter,

![dialogies](https://www.w3.org/Data/demos/chunks/nlp/dinner/images/dialogues.png)

The following diagram describes some of the concepts needed to describe choices of drinks:

![drinks](https://www.w3.org/Data/demos/chunks/nlp/dinner/images/drinks.png)

The following diagram describes some of the concepts needed to describe choices of food:

![food](https://www.w3.org/Data/demos/chunks/nlp/dinner/images/food.png)
