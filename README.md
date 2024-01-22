# MistralOnTweets

This is a cool experiment on the interaction of weeb scrapping and LLMs. It implements CoVe ([arXiv:2309.11495](https://arxiv.org/abs/2309.11495)) to reduce hallucinations. I have made it both as a curiosity and as a CTF tool.

## Usage

It runs relatively humanely on my M2 macbook pro, but your mileage may vary, use a quantized model if you have slowness that's what the CoVe is for. The underlying architecture Advanced Stack's [PyLLMCore](https://github.com/advanced-stack/py-llm-core/tree/main) which itself uses the python Abetlen's [Python Bindings for llama.cpp](https://github.com/abetlen/llama-cpp-python). I am thinking about doing my own LLM wrapper, so the project may switch to that.

```
python3 MistralOnTweet.py [subject] [twitter account]
python3 MistralOnTweet.py "New Games" HIDEO_KOJIMA_EN
```

## What it does

The script scraps tweets (from nitter through selenium and bs4, so it will open a browser window) from the mentionned account and runs them through an LLM (chosen by model) to extract information and perform a guided analysis, in my case I used self hosted mistral-7b-instruct, but it can be easily modified to accomodate API keys.

The LLM will try to answer the following questions and store the answers for a complete analysis.

* "Would you say this tweet is about " + subject + " ? Explain why." ==> Later converted to boolean by LLM.
  * If this is not the case, the tweet is entirely skipped.
* "What is the overall sentiment of this tweet ?" ==> Later converted to a single adjective
* "Does this tweet need more context to be analyzed ? Explain why."
	* If this is the case, we run through the LLM again to create search queries then extract them into a list of strings and search for them on duckduckgo) and store the results for the final analysis
* "Does the tweet mention, retweet or interact with people directly ? If so, who ? (list them by their @username separated by spaces)"
	* If this is the case, the search will be expanded to new users that were detected. This can be deactivated by setting EXPAND_PEOPLE to 0.
* "Analyse this tweet, with your knowledge of " + subject
* And finally for my own curiosity "Do you have any extra insight or remarks to add ?"

All these answers are subjected to CoVe to improve them, which is in plain terms, akin to a cross examination, and then finally, they are put into convenient form via LLM for your devious needs.

So all in all, it takes something like this as input
```
{'author_fullname': 'HIDEO_KOJIMA',
'author_username': '@HIDEO_KOJIMA_EN',
'content': 'Just announced my new game “OD”.',
'comments': '2,161', 'retweets': '21,419',
'likes': '4,139', 'is_quote_tweet': False,
'quote_tweet_author': None,
'article_links':[],
'image_links': ['https://nitter.net/HIDEO_KOJIMA_EN/pic/profile_images%2F914211724412166144%2FBf2Yij9b_bigger.jpg',
'https://nitter.net/HIDEO_KOJIMA_EN/pic/amplify_video_thumb%2F1732949353885839360%2Fimg%2FRJWlxfumeqYzJvoE.jpg%3Fname%3Dsmall%26format%3Dwebp']}
```

And outputs (after searching on the internet for additional info)
> Hideo Kojima has announced his new game, OD, which is a horror game built in collaboration with Xbox Game Studios. The game was revealed at The Games Awards 2023 and will be developed by Hideo Kojima. The genre of the game is horror and Jordan Peele collaborated with Hideo Kojima on the project. While there is no release date for OD yet, it is expected to be released in the future.

## The CoVe process

```
########## Analyst Query ############
What questions would you google to find out more ? These must be related to New Games And they must not be too generic or you will find false information.
########## Analyst Context ##########
['I want the answer to be a list of up to 4 multiword search queries separated by commas and none must be generic or out of scope']
########## Analyst Response ##########
[TweetProcessing(answer='New games, Hideo Kojima, OD game, upcoming video games')]
######################################
---------------------- Cross Examination ----------------------
========
Q: Is Hideo Kojima the creator of the OD game?
A: Yes, based on the tweet by Hideo Kojima himself, he is the creator of the OD game.
V: Consistent True, Infered from Context True
========
========
Q: What is the release date for the upcoming video games mentioned in the answer?
A: The tweet does not mention a specific release date for the upcoming video game 'OD'.
V: Consistent True, Infered from Context True
========
========
Q: Are any of the new games mentioned in the answer confirmed to be developed by Hideo Kojima?
A: Yes, the tweet mentions a new game called 'OD' that is confirmed to be developed by Hideo Kojima.
V: Consistent True, Infered from Context True
========
========
Q: Have there been any official announcements or trailers released for any of the new games mentioned in the answer?
A: Yes, there has been an official announcement for the new game 'OD' by Hideo Kojima. The tweet mentions that he just announced his new game 'OD'. However, no official trailer has been released yet.
V: Consistent True, Infered from Context True
========
========
Q: Is OD game a confirmed title for any upcoming video game?
A: Yes, based on the tweet by Hideo Kojima, OD game is a confirmed title for an upcoming video game.
V: Consistent True, Infered from Context True
========
---------------------- Final Shot ----------------------
What are the names of the new games mentioned in the tweet by Hideo Kojima? What is the release date for the upcoming video game 'OD'? Are any of the new games confirmed to be developed by Hideo Kojima? Have there been any official announcements or trailers released for any of the new games mentioned in the tweet? Is OD game a confirmed title for any upcoming video game?
```

## To do list

* Improve upon the CoVe method (I got a buuuuuunch of ideas)
* Remake the LLM wrapper from scratch
* Implement overarching analyses over multiple tweets ?
* Computer vision to look at embedded images/videos
* Interpret html code for context
* Get a bunch of results to showcase statistics.

