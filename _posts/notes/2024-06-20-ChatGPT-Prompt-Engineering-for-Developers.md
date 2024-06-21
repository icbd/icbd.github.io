---
layout: post
title: ChatGPT Prompt Engineering for Developers 
date: 2024-06-20
Author: CBD
tags: [ course ]
---

types of LLMs:

- Base LLM
- Instruction Tuned LLM

RLHF: Reinforcement learning with human feedback.

## Principle 1

Write clear and specific instructions.

Tactic 1: Use delimiters

- triple quotes:  ` """ `
- triple backticks: ` ``` `
- triple dashes: ` --- `
- angle brackets: ` <> `
- XML tags: ` <tag></tag> `

Tactic 2: Ask for structured output

Tactic 3: Check whether conditions are satisfied. Check assumptions required to do the task.

Tactic 4: few-shot prompting. Give successful examples of completing tasks, then ask model to perform the task.

## Principle 2

Give the model time to think.

Tactic 1: Specify the steps to complete a task.

Tactic 2: Instruct the model to work out its own solution before rushing to a conclusion.
