---
layout: single
title: Why companies fail with Data Science
---
{:refdef: style="text-align: center;"}
![image](/assets/img/arrow_fail.jpg){:height="300px"}
{: refdef}

From my experience over four years most companies that hire Data Scientist don't actually realize how difficult it is to actually bring Data Science solutions to production (at least from my experience in Brisbane, Australia). <br>
Some think all you need is a laptop, install Python or R, get some data, build some models and push the results to some dashboard like Power BI and Tableau. <br>
In reality, what often happens is the following: <br>
- The data sits in someones' computer and is too big to be emailed, you end up with some weird solution where you mount a Windows network drive on your Linux development cluster and get them to upload the data to that network drive. <br>
- There's some complicated process where you need to join multiple data sources together, some from databases but some from unstructured object stores or some data from APIs. You write up some Python scripts and glue them together with cron jobs and call it a day because company/management/SME thinks that the code will just work, nothing ever fails... right? <br>
- You try to train a Deep Learning model and realize how slow it is on a laptop with no GPU. You ask if you can get a PC with GPU only to be told that you would need to fill out a hardware requisition form, this would then get reviewed by an IT board. They end up then rejecting your request anyways. You end up training on CPU over the weekend only to realize some minor code error causes your training script to fail half way over the weekend. You cry yourself to sleep and start the training script over the next weekend... <br>
- You finally get your hands on the dataset only to realize it will never fit in RAM. You could do this in batches but even then it would take a long time plus the overhead and investment in actually writing up the batching code. You end up sub-sampling the dataset and hope for the best. <br>

All of the examples above are from my own experience. <br>
I think the reason this ends up happening (in Brisbane and the companies I worked for) is not only because they do not realize how hard it is to actually do Data Science and therefore think a Data Scientist should be able to just do all this where in reality what you actually need is a team comprised of Data Analyst, Data Engineer, Data Scientist and UI/UX Developers to actually create anything with value. <br>
They also don't realize that their on-premise systems are not equipped to actually do any Data Science. This is exacerbated by the fact that existing processes and culture makes it almost impossible or ridiculously slow to actually obtain the necessary hardware to even start. <br>

The companies that I've found Data Science to actually be effectively deployed are ones that have realized the necessity to have a Data Science team instead of a Data Scientist/Engineer/Analyst/Visualisation all in one. <br>
They also understand that using on-prem or laptop/desktop are simply not effective and have started instead using cloud based providers like Azure/GCP or AWS.