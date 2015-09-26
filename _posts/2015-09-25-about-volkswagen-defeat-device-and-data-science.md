---
layout: post
title: Data Science and the Volkswagen Defeat Device
tags:
- data science
- volkswagen

---


> As a car owner,
>	I would like to reduce NOx emissions when I am standing still/ driving slowly in semi closed spaces (e.g. an underground parking),
>	so there is less NOx when I and others walk around this space (e.g. when leaving the car).  

Sounds like a great feature for a car. Right?

This might have been a User Story in the Data Science / Engine Software Engineering Team Sprint Planning at Volkswagen a few years ago. Let's imagine we're in that Sprint planning session. Being a data scientist myself, the steps below would be my take on building this feature. 

*Just to be 100% clear: I was not there, this blogpost is just a thought exercise*

* Step 1. Build a logistic regression model to classify the car state. It's a simple equation where you fill in values like acceleration, steering wheel angle, air pressure (maybe a light sensor?) and the model calculates a value between 0 and 1. Closer to 0 would mean that the car is more likely to be still. Closer to 1 would mean that the car is more likely to be moving. 

* Step 2. If the model 'predicts' that the car is still, we make the car recycle the engines gasses. This reduces the NOx, at the cost of less engine power. But hey, the car is standing still so the decreased engine power is not an issue. If the car is moving, it does not recycle engine gasses such that the engine is more efficient and powerful, less CO2 at the cost of more NOx. 

* Step 3. Code, test and validate the logistic regression model in R/Python/SAS. The end result is one equation; one single line of code that needs to be added to the car's software. Another couple of lines to enable/disable engine gas recycling. 

All of this would take me a couple of hours, and only a handful of people would need to be involved. 

Now I don't know anything about car manufacturing software, but I assume that some pretty exhaustive standards/processes need to be followed to add code the car's software. Nevertheless, software documentation quality suffers from time pressure and other imperfections. Plus the legal department knowing all about ISO standards not necessarily understands the effect of this 'logistic regression model' on a car when it is located in a test center.

VW seems to be eating a lot of dust at the moment. I have been trying hard to find engineering statements/explanations but instead I have only found  **legaleze apologies**. It would be great if this software 'defeat device' could be **open sourced** for public scrutiny. Or is there really a hardcoded List with lat/lon values of US car test centers in there?



*Disclaimer: I happily own a VW Passat that sleeps in an underground parking.*
