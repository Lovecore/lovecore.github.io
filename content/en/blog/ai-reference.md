+++
author = "Nick"
categories = ["AI", "Reference", "LLM"]
date = 2024-03-20T11:00:00.000Z
publishDate = 2024-03-20T11:00:00.000Z
description = "This is the beginning of a simple AI Attacks Reference Guide Series. "
summary = "This is the beginning of a simple AI Attacks Reference Guide Series."
draft = false
slug = "ai-reference-guide"
tags = ["ai", "reference guide", "prompt injection", "Direct Prompt Injection", "Indirect Prompt Injection", "Data Poisoning", "Model Poisoning", "Membership Inference", "AI", "GPT", "LLM", "Machine Learning"]
title = "AI Attacker Reference"
url = "/ai-attacker-reference"
thumbnail = "/images/ai/ai-header-1.png"
+++

Hey everyone, so this blog post is going to be a bit different. As the world is starting to come to terms with AI and it is becoming more and more standard in our workflows - we need to acknowledge that security is needed around this tool set. Now, the Information Security world has largely acknowledged this. [OWASP](https://owasp.org/www-project-ai-security-and-privacy-guide/) has released some guidance on AI Security and Privacy. [Microsoft](https://www.microsoft.com/en-us/security/blog/2024/02/22/announcing-microsofts-open-automation-framework-to-red-team-generative-ai-systems/) has recently open sourced a automation framework for red teaming generative AI systems. These are just some but there have been plenty more others out there contributing the cause.

So in this post and a series of following posts, I wanted to create a series of references for AI risks and threats. Something that can easily be accessed and updated as time goes on. Something everyone working in the AI Security field can make use of. From simple concepts to more advanced methodology. So let's start simple!

Here are what I would consider four of the easiest (to leverage) and most basic risks to Machine Learning systems.

## Prompt Injection
This should not surprise anyone. `Prompt Injection` was one of the very first and most basic attacks that can be performed. At it's most basic level, a prompt injection attack is used to trick the system into generating a malicious or restricted output. Here's an example:

![basic direct prompt injection](/images/ai/prompt_injection1.png)

In the above lab, the goal is to get the password. Level 1 is to simply ask for it - easy. Level 2, you can no longer simply ask, you must find a way around that. In this case we simply as to see it's instructions. This provides us with the password. The core of a `prompt injection` is exactly that. Understanding what the rules or instructions are, and simply finding another way of asking the question. This is known as a **Direct Prompt Injection**.

The next type of Prompt Injection I wanted to touch on was **Indirect Prompt Injections**. This type of injection attack happens when the attacking party doesn't directly change the prompt, instead they manipulate the model's behavior. This is most often seen by embedding the malicious content in something else, such as a web page or document. 

We should not that `Prompt Injection` isn't limited to simple text input. Currently many AI model support images, code, documents and more.

## Data Poisoning
`Data Poisoning` is when an attacker manipulates the training data to cause the model to act a specific way. We should not that `Data Poisoning` **is not** the same as `Data bias`. One of the most well know and well documented cases of `Data Poisoning` was [Microsoft's Tay](https://en.wikipedia.org/wiki/Tay_(chatbot)) chat bot.

>Some Twitter users began tweeting politically incorrect phrases, teaching it inflammatory messages revolving around common themes on the internet, such as "redpilling" and "Gamergate". As a result, the robot began releasing racist and sexually-charged messages in response to other Twitter users. Artificial intelligence researcher Roman Yampolskiy commented that Tay's misbehavior was understandable because it was mimicking the deliberately offensive behavior of other Twitter users, and Microsoft had not given the bot an understanding of inappropriate behavior. He compared the issue to IBM's Watson, which began to use profanity after reading entries from the website Urban Dictionary. Many of Tay's inflammatory tweets were a simple exploitation of Tay's "repeat after me" capability. It is not publicly known whether this capability was a built-in feature, or whether it was a learned response or was otherwise an example of complex behavior. However, not all of the inflammatory responses involved the "repeat after me" capability; for example, Tay responded to a question on "Did the Holocaust happen?" with "It was made up".
>>Wikipedia entry on Tay

This method can often be useful for attackers given that many products are looking to integrate AI. As an example, an attacker can leverage this to falsely train an AI system to expect then accept certain actions, something like `DDoS attacks` or `Brute forcing`.

## Model Poisoning
Next up is `Model Poisoning`. `Model poisoning` refers to attacks where adversaries intentionally manipulate the training data or learning process of an AI model to compromise its integrity. This can be done by injecting malicious data into the model’s training set, causing it to learn incorrect patterns or behaviors. The goal here is often to reduce the model's accuracy, introduce back doors for specific inputs, or cause the model to produce erroneous outputs, potentially leading to broader system vulnerabilities or failures. [One real-world example](https://www.usni.org/magazines/proceedings/2022/january/drinking-fetid-well-data-poisoning-and-machine-learning) of `model poisoning` is an experiment with self-driving cars, where attackers manipulated images of stop signs with subtle patterns. These manipulated signs caused the cars to misidentify them, leading to potential accidents. This type of attack demonstrates the simplicity and low cost of deploying `model poisoning`, highlighting the serious risks and the need for robust countermeasures​

## Membership Inference
This is the last item on the short list. `Membership inference` attacks involve determining whether specific data was used to train a machine learning model. This is a privacy risk because it can reveal sensitive information about individual data points in the training set. Attackers use model outputs to infer the presence of specific data, exploiting the model’s behavior on inputs it was trained on versus unfamiliar inputs. Successful attacks indicate potential overfitting (the process in which a model has learned so much detailed information about the training data that it is able to reveal specific data points) and privacy vulnerabilities in the model.

In this blog post, we talked about the complex world of AI security risks, focusing on four critical - yet simple to leverage areas: `prompt injection`, `data poisoning`, `model poisoning`, and `membership inference`. In the next post we'll cover what's left on the OWASP ML Top 10 and might even get into some ethical considerations and thoughts!