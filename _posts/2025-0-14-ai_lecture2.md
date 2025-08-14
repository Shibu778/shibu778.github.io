---
title: What Makes an AI an "Intelligent Agent"? 🤖
date: 2025-08-14
categories: [AI]
tags: [aicourse, lecturenotes]
author: Shibu
description: Second post in the AI class series.
toc: true
pin: false
comments: true
---

Welcome back to our class blog! Today, we're diving into the core concept of a **rational agent**, which is central to modern AI. We'll explore what agents are, how they interact with their environment, and what makes their behavior "good" or "intelligent."

---

### The Agent and Environment Model

At its simplest, an **agent** is anything that can perceive its **environment** through **sensors** and act upon it using **actuators**. This simple model applies to a wide range of entities:
- **Human Agent:** Our eyes and ears are sensors, while our hands and voice are actuators.
- **Robotic Agent:** A robot's cameras and range finders act as sensors, and its motors are actuators.
- **Software Agent:** A software agent's sensors could be keystrokes or network packets, and its actuators might be displaying information on a screen or sending network packets.

The agent's input at any given moment is called a **percept**. The entire history of everything the agent has perceived is its **percept sequence**. The agent's behavior is defined by its **agent function**, which is a mathematical mapping from every possible percept sequence to an action. The **agent program** is the actual, concrete implementation of this function that runs on a physical system or "architecture".

---

### Defining "Good Behavior": The Concept of Rationality

What makes an agent "intelligent" or "good"? It's all about **rationality**. A **rational agent** is one that does the "right thing" to maximize its performance. We measure this with a **performance measure**, which evaluates the desirability of the sequence of environment states the agent's actions produce.

A key point here is that the performance measure is based on the state of the _environment_, not the agent's internal state. A good design defines this measure based on what you actually want to happen in the world, not on how you think the agent should behave.

For example, a vacuum cleaner agent's performance shouldn't be measured by how much dirt it sucks up, as it could just dump dirt and clean it again. A better measure would be rewarding it for having a clean floor, perhaps with a penalty for noise or electricity use.

A rational agent's choice of action depends on four key things:

- The  **performance measure**.
- Its **prior knowledge** of the environment.
- The **actions** it can perform.
- Its complete **percept sequence** to date.

It's important to distinguish between rationality and **omniscience**. An omniscient agent knows the future, which is impossible in reality. A rational agent simply maximizes its _expected_ performance based on the information it has. Rationality also includes important behaviors like **information gathering** and **exploration** to get new percepts that can improve future decisions. A rational agent should also be **autonomous**, meaning it learns from its experience to compensate for any incorrect or partial initial knowledge from its designer.

---

### The Variety of Task Environments

The nature of the environment heavily influences how an agent should be designed. To design an agent, we first specify its **task environment** using the **PEAS** (Performance, Environment, Actuators, Sensors) framework.

For example, here's a PEAS description for an automated taxi driver:
- **Performance Measure:** Safe, fast, legal, comfortable trip, and maximizing profits.
- **Environment:** Roads, other traffic, pedestrians, and customers.
- **Actuators:** Steering wheel, accelerator, brake, signal, and horn.
- **Sensors:** Cameras, sonar, speedometer, GPS, and a keyboard for the passenger.

Environments can be categorized along several dimensions:

- **Fully vs. Partially Observable:** Can the agent's sensors see the complete state of the environment at all times? A vacuum cleaner with only a local dirt sensor is in a partially observable environment.
- **Single vs. Multi-Agent:** Is the agent operating alone or are there other agents whose actions affect its performance? A crossword puzzle is single-agent, while a game of chess is multi-agent.
- **Deterministic vs. Stochastic:** Is the next state of the environment completely predictable from the current state and the agent's action? Real-world scenarios like taxi driving are stochastic due to unpredictable factors.
- **Episodic vs. Sequential:** Is the agent's experience broken into separate episodes where its actions in one don't affect the next? In a sequential environment like chess or taxi driving, current actions have long-term consequences.
- **Static vs. Dynamic:** Does the environment change while the agent is deliberating? A crossword puzzle is static, but a taxi-driving environment is dynamic.
- **Discrete vs. Continuous:** Does the environment have a finite number of states, or can its attributes (like speed or location) take on a continuous range of values? Chess is discrete, while taxi driving is continuous.
- **Known vs. Unknown:** Does the agent or designer know the rules and outcomes of actions in the environment? It's possible to have a known environment that is partially observable (like a game of solitaire) or an unknown one that is fully observable (like a new video game).

---

### The Structure of Agent Programs

The core challenge for AI is to create a program that produces rational behavior without relying on a massive, unfeasible lookup table. There are four main types of agent programs that address this challenge:

- **Simple Reflex Agents:** These agents act based only on the current percept, ignoring any history. They use **condition-action rules** (e.g., _if car-in-front-is-braking then initiate-braking_). These agents are simple but can struggle in partially observable environments, often getting stuck in infinite loops.
- **Model-Based Reflex Agents:** To handle partial observability, these agents maintain an **internal state** of the world. They use a **model of the world** to update this state based on how the world evolves and how their actions affect it. This allows them to make more informed decisions than simple reflex agents.
- **Goal-Based Agents:** These agents go beyond just understanding the current state. They also have a **goal** that describes desirable situations (e.g., arriving at a destination). They use their model of the world to find a sequence of actions that will lead to achieving that goal. This makes them more flexible than reflex agents, as changing the goal is easier than rewriting many rules.
- **Utility-Based Agents:** When there are multiple ways to achieve a goal, or when goals conflict, a **utility-based agent** is needed. These agents have a **utility function** that measures their "happiness" or preferences for different world states. They choose the action that maximizes the **expected utility** of the outcomes, which is crucial for making decisions under uncertainty.

**Learning Agents:** All of these agent types can be enhanced with a **learning element**. This component uses feedback from a **critic** to improve the agent's **performance element** over time. A **problem generator** suggests exploratory actions that might not seem optimal in the short term but could lead to discovering better long-term strategies. Learning allows agents to operate in unknown environments and become more competent with experience.

Finally, the internal "knowledge" of these agents can be represented in different ways, from simple **atomic representations** (a state is an indivisible unit) to **factored representations** (a state is a collection of attribute-value pairs) and complex **structured representations** (describing objects and their relationships). The choice of representation affects the agent's expressiveness and the complexity of the reasoning required.

### References
1. Artificial Intelligence: A Modern Approach" by Russell and Norvig, Chapter 2
