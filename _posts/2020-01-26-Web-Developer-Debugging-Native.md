---
layout: post
title: Native Problem, web brain. How to Fix a Problem When You Don’t Understand It's Core
---

# A philosophical dilemma on the limits of learning, and why you don’t need to know it all to start problem-solving.

There is nothing quite like being totally humbled.

I've been a web developer for 6 years now and I find that I can reason about and solve many of the problems I face day to day in the web world. Sure, I still get stuck on some advanced topics, and occasionally even the simple things, but normally those are just a small piece of the puzzle and I can crank out the rest.

Last Friday, however, I found myself in a scenario that humbled me to the core.

I was staring down an unfamiliar challenge that not-so-gently reminded me of my lack of knowledge.

## The Problem

I work on a team that serves as the escalation point for issues other development teams run into. I love this! I’ll admit, it can be distracting and disruptive at times, but we are in a position not only to help people when they need it most but also to work on complicated and new issues all the time.

Normally these fall into our wheelhouse, which I would define as a web based product, but this case was different.

We were presented with a challenge to understand why a UWP application update process worked fine in a 32 bit windows environment but failed when trying to run it in a 64 bit environment.

## 😅Act Natural😅

I like to think our team has a reputation for confidence, and, so far, we’ve never faced a challenge we couldn't solve. Still, this one had me a bit worried.

I quickly realized that, although I’d spent 6 years diving deep into web technologies, I was in foreign waters when it came to hardware-level issues.

It was a bit shocking to realize that, even with everything I had learned up to this point, there is always so much more to learn.

You don't know it all and you never will. 

Now that I’m writing it, that fact seems obvious, but I’m someone who spends a lot of time learning new things and enjoys learning them. When faced with the reality of the situation, it was hard to accept my lack of knowledge in this arena.
 
Nevertheless, this is what my team does. We solve problems. Other teams of people rely on us to solve these problems, and they were relying on us to solve this one. I pushed aside my doubt. It was time to focus.

We were going to solve this.

I started the same way I start solving any challenge in the web world: **narrow it down**.

“What part of the process is failing?”

In this case, manually installing the app (via a PowerShell script) was fine, but running an installer program (c# console app) was failing. That's an important clue.

## The Plan

We started by adding logging to the process. Now, the process did have logging, but not to the degree that I’d like.

I want every single possible branch to be logged.

Our plan was to run the program in a failing state, reset the log, then run it again in a passing state (manual running of the script). I know that if I narrow down the problem, I may be able to compensate for my inexperience in the platform.

After adding about 600 logging points we ran the test and ... **BAM!** There was a diversion in the logs.

The successful branch changed at a single point from the unsuccessful one.

It was on this area of code:

```powershell
 # Get architecture-specific dependencies
        if (($Env:Processor_Architecture -eq "x86" -or $Env:Processor_Architecture -eq "amd64") -and (Test-Path (Join-Path $DependencyPackagesDir "x86")))
        {
            $DependencyPackages += Get-ChildItem (Join-Path $DependencyPackagesDir "x86\*.appx") | Where-Object { $_.Mode -NotMatch "d" }
        }
        if (($Env:Processor_Architecture -eq "amd64") -and (Test-Path (Join-Path $DependencyPackagesDir "x64")))
        {
            $DependencyPackages += Get-ChildItem (Join-Path $DependencyPackagesDir "x64\*.appx") | Where-Object { $_.Mode -NotMatch "d" }
        }
```

The successful install fell into both if conditions while the unsuccessful install only fell into the first. Now that we had the problem narrowed down, Step 2 is to ... (surprise!) narrow the problem down.

By looking at this code, we could see there are 2 conditions that could cause the problem:

`$Env:Processor_Architecture` or the `$DependencyPackagesDir`

Being that, as discussed, I'm unfamiliar with this environment, I don't trust my instincts and decide to keep adding to the logging. I add the 2 variables in order to see their values as the scripts run.

We run the 2 processes again and find that the `$Env:Processor_Architecture` is x86 in the failing state and x64 in the successful state.

The second variable is matching the conditions to fall into both of the if blocks, so now we have our target. One thing I know from the web world is that Google is your friend, so I google "c# console app $Env:Processor_Architecture" and I find [this](https://stackoverflow.com/a/4152544/5997923) as the top result.

I know that the installer is a C# console app and that is giving us the issue. After reviewing the answer, I decided to look at the project's target architecture and find it to be "Any Cpu".

Here is a screenshot of the actual project settings:
![settings-image](/assets/project-settings.png)

I’ll admit I was a bit confused. The Stack Overflow post states that "Any CPU" should default to what's most compatible, which, in my mind, meant x64.

Later, we learned that there is another setting in the image that was impacting our scenario, which is the "Prefer 32-bit" checkbox. That was forcing this to run as an x86 on a 64 bit system.

Once we unchecked that setting, updates started working on both 32 bit systems and 64 bit systems.

## The Takeaways

I learned quite a bit from this experience. I learned that there is a lot more for me to learn, and there will be even more that I never learn.

I learned that I need to be comfortable with the fact that there will always be areas of development that I will never know about.

And that's okay!

I’ll continue to learn about what I like, what I’m passionate about, and what helps me in my career. The rest? Well, for me, it’s not important. Not everything will be important for every developer.

I'm not going to learn everything, but I'm going to learn the things I love.

I also learned that even when you feel like you don't know anything about a particular vertical in development, there will almost always be some sort of skills transfer. In this case, it was the ability to narrow down the problem, narrow it down more, isolate the scenario to recreate the issue, and trust in Google!

Another takeaway was that not all problems are created equal and you should always pay attention to details. Missing that checkbox setting could have cost us a lot of time in solving this problem. The combination of all of these skills helped us get to the bottom of the issue quickly and efficiently.

It's ok to not know something, but that’s not an excuse to not try and solve a problem.

### Teamwork is key
Finally, I want to stress the importance of not trying to tackle these problems alone. I couldn’t have solved this alone. Or, at the very least, it would have taken a lot longer and caused a lot more headaches.

Between the legwork done before the problem even reached us and the way everyone on my team worked together, we were able to solve this quickly and efficiently.

Don’t go at problems like this alone!

Even the ability to brainstorm and discuss with others can be immensely helpful.
