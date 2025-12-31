# Playing-With-LLM-Calls
Personal Project to Learn Agentic Framework / Lack Of

## Starting with Open-AI SDK for the Quiz Application 
I have been thinking about 3 different paths for students to choose from 

**1. The Tutor Path** : Useful to learn during the module, Students use this path when they want to excel at current learning and deep dive. 

**2. The Snippet / Code Path** : You interact with this path when you want to strengthen your coding prowess

**3. The MCQ / CCEE Path** : Only useful during CCEE / Preparing for CCEE (planning to fine tune based on a bunch of data), need to get feedback about generated content for this to workout, Or just use a vector DB for each module then randomly pick questions from each module 

---

# Implementation Logic 

For all of this to come together we need a router agent, rather there needs to be UI in place that lets students decide which path they want to choose. 

> Assuming they choose the Tutor Path
> There further needs to UI in place for students to pick from a specific module
> After picking said module there needs to be memory in place for the student and their level such that this adaptive tutor can assess student's comprehension of chosen module
> To intelligently handle student's level we can apply ELO logic
> > Or a much easier way to implement is to just have another agent which handles Judgement of Level

> After Level Handling of student, Is there a need for SME Agents for each module ?
> This then needs a router to handle which SME Agent to route to 



## Logic for the Tutor Path

## Logic for CCEE Path 

Vectorise and index specific module questions 
use RNG to pick random questions / see if there is any framework available to pick from vector DB 



