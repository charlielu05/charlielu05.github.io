---
layout: single
title: Code anywhere with Github Codespaces
---
{:refdef: style="text-align: center;"}
![image](https://learn.microsoft.com/en-us/training/media/student-hub/GithubCodespaces-02.png){:height="300px"}
{: refdef}

I've recently traded up my old 2018 iPad Pro 11" to a 2021 12.9" version with the magic keyboard. Apart from using it as just my Youtube/Netflix machine, I always wanted to try out the "Digital Nomad" life style since my new job is 100% working from home. Even though I've got quite a high spec MacBook Pro 16", lugging around a 2kg machine around with me in a large backpack just didn't seem appealing to me. <br>

Now previously, when I attempted this with my 11" iPad Pro I had to resort to using a SSH app (Terminus) to connect to an external virtual machine (AWS EC2) I had setup as the coding machine. Even though this technically worked, the entire user experience was a bit clunky. You were limited to using the shell editor to modify code and I am definetely not one of those VIM/Nano wizards, best I can do is edit some code and exit VIM without having to Google how to. <br>

So after a bit of searching around on the internet, I was just about to give up when I came along a blog post talking about running a web version of VS Code on the iPad Pro. (https://dev.to/n3wt0n/codespaces-on-ipad-good-enough-for-working-24bf). <br>
The entire premise was nearly too good to be true, essentially what you get is a cloud based IDE that is customised for each of your code repo you have in Github. <br>
This meant no more virtual environments or separate docker images for different code repos. Instead, you simply move into the code repo and start the Codespace instance and you would be ready to go. <br>
For some reason, my Github account had already been activate with this feature so I didn't need to apply for access at all. <br>

I've now tested it for about two days and have found some pros and cons from my short time of use. <br>
### Pro
- Really fast to start, the initial codespace environment takes about 30 seconds, after that its about 10 seconds to start up on my iPad Pro. <br>
- You can use Github secrets to inject your secrets as environment variable. So I can use this to setup my AWS access key and secret key. <br>
- It's essentially a full VSCode IDE in a browser. <br>

### Con
- Some lag when you type in the terminal, possibly because the servers are US based and I'm connecting from Australia. <br>
- There is an annoying bug where Codespaces doesn't recognise my mouse scroll wheel. Using my hand to swipe still works to scroll the screen. <br>


Overall I would say the experience is really positive and I'm keen to try just taking my iPad Pro with me on my trips out to shared workspaces when I get sick of working from home. 

