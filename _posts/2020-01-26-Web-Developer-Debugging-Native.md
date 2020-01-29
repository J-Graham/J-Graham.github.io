---
layout: post
title: A web developer in a native situation.
---

## How I solved a native problem using my web brain.

There is nothing like being totally humbled.  I've been a web developer for 6 years now and I find that I can reason about and solve many of the problems I face day to day in the web world.  Sure I still get stuck on simple things and sometimes more advanced topics, but normally it is a small piece of the puzzle and I can crank out the rest.  But I found myself in a scenario last Friday 01/25 where I was humbled by the challenge and my lack of knowledge.

I work on a team at my company that is seen as the escalation point for issues that other development teams run into.  **I love this!**  Sure sometimes it is distracting and disrupting, but we are in a position to not only help people that really need it, but to also work on complicated and new issues.  Normally these fall into our *wheelhouse*, which I would define as a web based product, but this case was different.  We were presented with a challenge to understand why a UWP application update process worked fine in a 32 bit windows environment, but when trying to run it on a 64 bit environment it failed.

### 😅Act Natural😅

I like to think the reputation of our team is that of confidence, and we haven't been faced with a challenge we couldn't solve yet, but this one had me a bit worried.  I quickly realized that although I spent 6 years diving deep into web technologies, I was in a foreign land when it came to hardware level issues.  It was a bit shocking to realize that with everything I learned, there is just so much more you don't and will never know.  This seems obvious while writing this, but as someone who spends a lot of time learning new things, it was hard to accept it when presented with the reality of the situation.

Pushing aside this feeling of doubt I decided to focus on the problem.  I started the same way I start solving any challenging problem in the web world, **narrow it down.**  What part of the process is failing?  In this case manually installing the app (via a powershell script) is fine, but running an installer program (c# console app) it was failing.  That's an important clue.

### The Plan

We started by adding logging to the process.  Now the process has logging, but not like I wanted.  I want every single possible branch to be logged.  My plan was to run the program in a failing state, reset the log, then run it again in a passing state (manual running of the script).  I know that if I can narrow down the problem I can compensate better for my inexperience in the platform.  After adding in about 600 logging points we ran the test and ..... **BAM,** there was a diversion in the logs.  The successful branch changed at a single point from the unsuccessful one.

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

The successful install fell into both `if` conditions while the unsuccessful one only fell into the first.  Now that we have the problem narrowed down, step 2 is to, no surprise, narrow the problem down.  By looking at this code there are 2 conditions that could cause this.  The `$Env:Processor_Architecture` or the `$DependencyPackagesDir`.  Being I'm uncomfortable in this environment I don't trust my instincts and decide to just add to the logging the 2 variables in order to see their values as the scripts run.

We run the 2 processes again and find that the `$Env:Processor_Architecture` is x86 in the failing state and x64 in the successful state.  The second variable is matching the conditions to fall into both of the `if` blocks so I now have my target.  One thing I know from the web world is google is your friend so I google "c# console app $Env:Processor_Architecture" and I find [this](https://stackoverflow.com/a/4152544/5997923) as the top result.  I know that the installer is a C# console app and that is giving us the issue.  After reviewing the answer I decided to look at the project's target architecture and find it to be "Any Cpu".

Here is a screen shot of the actual project settings:

![settings-image](/assets/project-settings.png)

I was a bit confused because the Stack Overflow post states that "Any CPU" should default to what's most compatible and to me that meant x64.  Later we learned that there is another setting in the image that is impacting our scenario, and that's the "Prefer 32-bit" checkbox.  That was forcing this to run as an x86 on a 64 bit system.  We unchecked that setting and now updates were working on both 32 bit systems and 64 bit systems.

### Takeaways

I learned a lot from this experience.  I learned that there is a lot more to learn, and there will be even more that I never learn.  I need to be comfortable with the fact that there will be areas of development that I will never know about.  **And that's okay!**  I learn about what I like and what I need for my passion and my career, the rest just isn't important.  I'm not going to try to learn everything, I'm going to learn about things I love.

I also learned that even though you don't know anything about a particular vertical in development, almost always skills transfer.  In this case the ability to narrow down the problem, and then narrow it some more, isolate the scenario to recreate the issue and trust in google!  I also took away that not all problems are created equal and you should pay attention to the details.  Missing the check box setting could have cost us a lot of time in solving this problem.  The combination of all of these skills helped me get to the bottom of the issue quickly and efficiently.  It's ok to not know something, but it's not an excuse to not try and solve a problem.

Finally I want to make sure I state that this was not solved alone.  I had help from my team and other teams to do most of the leg work on implementing the logging and running the different scenarios.  You shouldn't go at a problem like this alone, the ability to just discuss the problem with others was immensely helpful.
