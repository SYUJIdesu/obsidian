---
title: Thoughts On A Month With Devin
source: https://www.answer.ai/posts/2025-01-08-devin.html#appendix-tasks-attempted-with-devin
author:
  - "[[Hamel Husain]]"
published: 
created: 2025-05-15
description: Our impressions of Devin after giving it 20+ tasks.
tags:
  - web
  - AI
---
In March 2024, a new AI company burst onto the scene with impressive backing: a $21 million Series A led by Founders Fund, with support from industry leaders including the Collison brothers, Elad Gil, and other tech luminaries. The team behind it? IOI gold medalists - the kind of people that solve programming problems most of us can’t even understand. Their product, [Devin](https://devin.ai/), promised to be a fully autonomous software engineer that could chat with you like a human colleague, capable of everything from learning new technologies and debugging mature codebases to deploying full applications and even training AI models.

The early demos were compelling. [A video](https://youtu.be/UTS2Hz96HYQ?si=Wid68ZqqibBuY34-) showed Devin independently completing an Upwork bounty, installing and running a PyTorch project without human intervention.[<sup>1</sup>](https://www.answer.ai/posts/2025-01-08-devin.html#fn1) The company claimed Devin could resolve 13.86% of real-world GitHub issues end-to-end on the SWE-bench benchmark - ~3x times better than previous systems. Only a select group of users could access it initially, leading to breathless tweets about how this would revolutionize software development.

As a team at Answer.AI that routinely experiments with AI developer tools, something about Devin felt different. If it could deliver even half of what it promised, it could transform how we work. But while Twitter was full of enthusiasm, we couldn’t find many detailed accounts of people actually using it. So we decided to put it through its paces, testing it against a wide range of real-world tasks. This is our story - a thorough, real-world attempt to work with one of the most hyped AI products of 2024.

## What is Devin?

What makes Devin unique is its infrastructure. Unlike typical AI assistants, Devin operates through Slack and spins up its own computing environment. When you chat with Devin, you’re talking to an AI that has access to a full computing environment - complete with a web browser, code editor, and shell. It can install dependencies, read documentation, and even preview web applications it creates. Below is a screenshot of one way to initiate a task for Devin to work on:

![](https://www.answer.ai/posts/images/devin_slack.png)

One way to initiate a task with Devin - through Slack

The experience is designed to feel like chatting with a colleague. You describe what you want, and Devin starts working. Through Slack, you can watch it think through problems, ask for credentials when needed, and share links to completed work. Behind the scenes, it’s running in a Docker container, which gives it the isolation it needs to safely experiment while protecting your systems. Devin also provides a web interface, which also allows you to gain access to its envirnoment and watch it work with IDEs, Web Browsers and more in real time. Here is a screenshot of the web interface:

![](https://www.answer.ai/posts/images/devin_internal.png)

## Early Wins

Our first task was straightforward but real: pull data from a Notion database into Google Sheets. Devin tackled this with surprising competence. It navigated to the Notion API documentation, understood what it needed, and guided me through setting up the necessary credentials in Google Cloud Console. Rather than just dumping API instructions, it walked me through each menu and button click needed - saving what would typically be tedious documentation sleuthing. The whole process took about an hour (but only a few minutes of human interaction). At the end, Devin shared a link to a perfectly formatted Google Sheet containing our data.

The code it produced was a bit verbose, but it worked. This felt like a glimpse into the future - an AI that could handle the “glue code” tasks that consume so much developer time. Johno had similar success using Devin to create a planet tracker for debunking claims about historical positions of Jupiter and Saturn. What made this particularly impressive was that he managed this entirely through his phone, with Devin handling all the heavy lifting of setting up the environment and writing the code.

## Scaling Up Our Testing

Building upon our early successes, we leaned into Devin’s asynchronous capabilities. We imagined having Devin write documentation during our meetings or debug issues while we focused on design work. But as we scaled up our testing, cracks appeared. Tasks that seemed straightforward often took days rather than hours, with Devin getting stuck in technical dead-ends or producing overly complex, unusable solutions.

Even more concerning was Devin’s tendency to press forward with tasks that weren’t actually possible. When asked to deploy multiple applications to a single [Railway](https://railway.com/) deployment (something that Railway doesn’t support), instead of identifying this limitation, Devin spent over a day attempting various approaches and hallucinating features that didn’t exist.

The most frustrating aspect wasn’t the failures themselves - all tools have limitations - but rather how much time we spent trying to salvage these attempts.

## A Deeper Look at What Went Wrong

At this point in our journey, we were puzzled. We had seen Devin competently handle API integrations and build functional applications, yet it was struggling with tasks that seemed simpler. Was this just bad luck? Were we using it wrong?

Over the course of a month, we systematically documented our attempts across these categories:

1. Creating new projects from scratch
2. Performing research tasks
3. Analyzing & Modifying existing projects

The results were sobering. Out of 20 tasks, we had 14 failures, 3 successes (including our 2 initial ones), and 3 inconclusive results. Even more telling was that we couldn’t discern any pattern to predict which tasks would work. Tasks that seemed similar to our early successes would fail in unexpected ways. **We’ve provided more detail about these tasks [in the appendix](https://www.answer.ai/posts/2025-01-08-devin.html#appendix-tasks-attempted-with-devin) below.** Below is a summary of our experiences in each of these categories:

### 1\. Creating New Projects From Scratch

This category should have been Devin’s sweet spot. After all, the company’s demo video showed it autonomously completing an Upwork bounty, and our own early successes suggested it could handle greenfield development. The reality proved more complex.

Take our attempt to integrate with an LLM observability platform called [Braintrust](https://braintrust.dev/). The task was clear: generate synthetic data and upload it. Instead of a focused solution, Devin produced what can only be described as code soup - layers of abstraction that made simple operations needlessly complex. We ultimately abandoned Devin’s attempt and used Cursor to build the integration step-by-step, which proved far more efficient. Similarly, when asked to create an integration between our AI notes taker and [Spiral.computer](https://spiral.computer/), Devin generated what one team member described as “spaghetti code that was way more confusing to read through than if I’d written it from scratch.” Despite having access to documentation for both systems, Devin seemed to overcomplicate every aspect of the integration.

Perhaps most telling was our attempt at web scraping. We asked Devin to follow Google Scholar links and grab the most recent 25 papers from an author - a task that should be straightforward with tools like [Playwright](https://playwright.dev/). This should have been particularly achievable given Devin’s ability to browse the web and write code. Instead, it became trapped in an endless cycle of trying to parse HTML, unable to extract itself from its own confusion.

### 2\. Research Tasks

If Devin struggled with concrete coding tasks, perhaps it would fare better with research-oriented work? The results here were mixed at best. While it could handle basic documentation lookups (as we saw in our early Notion/Google Sheets integration), more complex research tasks proved challenging.

When we asked Devin to research transcript summarization with accurate timestamps - a specific technical challenge we were facing - it merely regurgitated tangentially related information rather than engaging with the core problem. Instead of exploring potential solutions or identifying key technical challenges, it provided generic code examples that didn’t address the fundamental issues. Even when Devin appeared to be making progress, the results often weren’t what they seemed. For instance, when asked to create a minimal [DaisyUI](https://daisyui.com/) theme as an example, it produced what looked like a working solution. However, upon closer inspection, we discovered the theme wasn’t actually doing anything - the colors we were seeing were from the default theme, not our customizations.

### 3\. Analyzing and Modifying Existing Code

Perhaps Devin’s most concerning failures came when working with existing codebases. These tasks require understanding context and maintaining consistency with established patterns - skills that should be central to an AI software engineer’s capabilities.

Our attempts to have Devin work with [nbdev](https://nbdev.fast.ai/) projects were particularly revealing. When asked to migrate a Python project to nbdev, Devin couldn’t grasp even basic nbdev setup, despite us providing it access to comprehensive documentation. More puzzling was its approach to notebook manipulation - instead of directly editing notebooks, it created Python scripts to modify them, adding unnecessary complexity to simple tasks. While it occasionally provided useful notes or ideas, the actual code it produced was consistently problematic.

Security reviews showed similar issues. When we asked Devin to assess a GitHub repository (under 700 lines of code) for security vulnerabilities, it went overboard, flagging numerous false positives and hallucinating issues that didn’t exist. This kind of analysis might have been better handled by a single, focused LLM call rather than Devin’s more complex approach.

The pattern continued with debugging tasks. When investigating why SSH key forwarding wasn’t working in a setup script, Devin fixated on the script itself, never considering that the problem might lie elsewhere. This tunnel vision meant it couldn’t help us uncover the actual root cause. Similarly, when asked to add conflict checking between user input and database values, one team member spent several hours working through Devin’s attempts before giving up and writing the feature themselves in about 90 minutes.

## Reflecting As A Team

After a month of intensive testing, our team gathered to make sense of our experiences. These quotes capture our feelings best:

> Tasks it can do are those that are so small and well-defined that I may as well do them myself, faster, my way. Larger tasks where I might see time savings I think it will likely fail at. So no real niche where I’ll want to use it. *\- Johno Whitaker*

> I had initial excitement at how close it was because I felt I could tweak a few things. And then slowly got frustrated as I had to change more and more to end up at the point where I would have been better of starting from scratch and going step by step. *\- Isaac Flath*

> Devin struggled to use internal tooling that is critical at AnswerAI which, in addition to other issues, made it difficult to use. This is despite providing Devin with copious amounts of documentation and examples. I haven’t found this to be an issue with tools like Cursor, where there is more opportunity to nudge things in the right direction more incrementally. *\- Hamel Husain*

In contrast to Devin, we found workflows where developers drive more (like Cursor) avoid most issues we faced with Devin.

## Conclusion

Working with Devin showed what autonomous AI development aspires to be. The UX is polished - chatting through Slack, watching it work asynchronously, seeing it set up environments and handle dependencies. When it worked, it was impressive.

**But that’s the problem - it rarely worked.** Out of 20 tasks we attempted, we saw 14 failures, 3 inconclusive results, and just 3 successes. More concerning was our inability to predict which tasks would succeed. Even tasks similar to our early wins would fail in complex, time-consuming ways. The autonomous nature that seemed promising became a liability - Devin would spend days pursuing impossible solutions rather than recognizing fundamental blockers.

This reflects a pattern we’ve observed repeatedly in AI tooling. Social media excitement and company valuations have minimal relationship to real-world utility. We’ve found the most reliable signal comes from detailed stories of users shipping products and services. For now, we’re sticking with tools that let us drive the development process while providing AI assistance along the way.