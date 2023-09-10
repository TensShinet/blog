---
title: '[SD] Summary: Chapter 5'
tags:
  - sd
date: 2023-09-05 23:38:29
---




**TL; DR**



Summarize how to do the estimation.



<!--more-->



## QA



### How many commonly used types of servers are in the data center? And what are they?



There are three commonly used types of servers in the data center. They are:



+ Web servers: Depending on the praticle situation, web servers' memory and storage resources can be small to medium. However, such servers require good computational resources.
+ Application servers: They may require extensive computational and storage resources but need more available memory resources.
+ Storage servers: They require a lot of non-volatile storage resources.



### Tell me the important rate you have to remember.



| QPS handled by MySQL           | It's approximately on the order of several thousand.         |
| ------------------------------ | ------------------------------------------------------------ |
| QPS handled by key-value store | It's approximately on the order of tens of thousands.        |
| QPS handled by cache server    | It's approximately on the order of several hundred thousand. |



### If the DAU is 500M, the requests on average are 20 per day, and the QPS of a server is 8000, please tell me the total requests per day, total average requests per second, and total servers required.



+ Five hundred million times twenty, which is 10 billion for the total daily requests.
+ Ten billion divided by eighty-six thousand four hundred is about one hundred fifteen thousand for the total average requests per second.
+ There are two cases here:
  + We assume the traffic is even. It's one hundred fifteen thousand divided by eight thousand. The result is about 15. 
  + There is only one second for the daily request. It's ten billion divided by eight thousand. The result is about one million two hundred fifty thousand.



### If the DAU is 250M, the daily tweets are 3, the storage required per tweet is 250B, the storage required per image is 200 kilobytes, and the storage per video is 3 megabytes, please tell me the total requests per day, the storage for tweets per day, the storage of images per day, the storage of videos per day, and the total storage per day.



+ Total requests per day are three times two hundred million. The result is seven hundred fifty million.
+ The storage of tweets per day is two hundred fifty bytes times seven hundred fifty million. The result is about one hundred eighty-seven point five gigabytes.
+ Let's assume only ten percent of tweets with one image. As a result, the storage of images per day is seven hundred fifty million times zero point one times two hundred kilobytes. The result is fifteen terabytes.
+ Let's assume only five percent of tweets with one video. As a result, the storage of videos per day is seven hundred fifty million times zero point zero five times three megabytes. The result is about one hundred twelve point five terabytes.
+ The total storage is zero point one eight seven five TB plus fifteen TB plus one hundred twelve point five. The result is one hundred-eight terabytes.

















