---
published: true
title: Interviewing During Covid
category: misc
layout: default
---

**I interviewed for Google's Tensorflow, Apple's MLPT (Machine Learning Platform & Technology), Bytedance's ad infrastructure, Databrick's ML team, Citadel Securities as a quantitative research analyst, Hudson River Trading(HRT) as an algorithm engineer, and Jane Street's research desk as SWE. I received offers from all of the companies except for Jane Street. Here's my experience interviewing during COVID.**

*Disclaimer: I won't be walking on the edge of leaking confidential information like an idiot(yes, I signed an NDA for all of these companies). Don't expect to get any hints for your interviews.*

The structure of this blog is inspired by my friend [Han's medium blogpost.](https://medium.com/@XiaohanZeng/i-interviewed-at-five-top-companies-in-silicon-valley-in-five-days-and-luckily-got-five-job-offers-25178cf74e0f)

![interviews]({{ site.url }}/assets/interviews.png)

# Table of Contents

* TOC
{:toc}


# Preparation

**Working on machine learning infrastructure is 99% systems engineering and 1% machine learning.** My experience on machine learning infrastructure teams has taught me this, and preparing for systems engineering topics was the right way to go. I did the following to prepare:

## Algorithms

**~50 leetcode hard questions**. Some of them are DP, some are graph based, some of them are just NP-hard problems that are a pain to code(which is the point), and some include devising some clever data structure that supports a very specific access pattern. I gave myself roughly 40 minutes to solve these problems. ~15%(7 questions) of the time I couldn't figure out the correct solution because time limit exceeded, memory limit exceeded, or I was just flat out wrong. I directly read the solutions and learned the tricks necessary to solve the type of problems moving forward. Don't bother with medium or easy questions since hard questions often contain medium/easy tasks as subroutines, and these companies probably wouldn't ask you easy leetcode questions anyways.

I wrote the solutions in either python and C++(sometimes both) and went back to polish my code for minor optimizations or readability improvements. For C++, I made sure I wasn't using raw pointers unless appropriate and I was using C++17 (`constexpr` functions, `std::array` instead of raw arrays, smart pointers, template type deduction with lambdas, etc) features. The reason I wasn't using C++20 was because the online coding platforms(like coderpad) likely use stable distributions of GCC and clang, which means some of the new features are in their experimental phase. **I didn't want to encounter a bug with concepts or `std::ranges`  in the middle of the interview.** (In fact, I found a [bug with attributes](https://stackoverflow.com/questions/62398252/why-likely-attribute-in-c20-raises-a-warning-here) recently in a new version of gcc)

I also spent a few days on problems elsewhere:

- [Codejam problems](https://codingcompetitions.withgoogle.com/codejam/archive). Round 1 and 2 are feasible, but round 3 was very difficult. I'd suggest studying round 1's if you only care about interviews.
- [Codeforce contests](https://codeforces.com/). There are 3 tiers(or Divs, as they call it), and for interviews I suggest Div 3 and Div 2. Don't bother with the D+ questions in Div 2, and definitely don't bother with Div 1.

## Systems Design

Working at Airbnb has made me pretty familiar with high level distributed systems design, but of course I worked only with a subset of them. I think Martin Kleppman's book [Designing Data Intensive Applications](https://www.google.com/books/edition/Designing_Data_Intensive_Applications/p1heDgAAQBAJ?hl=en) is a great read, but you'll have to pick and choose which sections you want to go over as it's a pretty dense book. If you don't have time, maybe just try understanding how Kubernetes works with Marko Luksa's [Kubernetes in Action](https://www.manning.com/books/kubernetes-in-action), which is a much easier read. You can then draw parallels with the distributed design for K8s against whatever systems design question the interviewer has for you.

Make sure you know some fundamental ideas about distributed systems like the **map reduce paradigm**, **sharding**, **asynchronous and synchronous follower replicas**, **CAP theorem**, etc. *What you don't want to do is read 3 sentences about each of the terms above and regurgitate it in your interviews. Interviewers have been doing this for a while, they know you don't actually understand the concepts.* Don't be that guy.

## Math Questions

**These are only asked in finance firms.** Honestly, these are just all over the place. I read this green book called [A Practical Guide to Quantitative Finance Interviews](http://quantfinanceinterviews.com/) by Xinfeng Zhou, but only doing a single problem in each section by myself. Hedge funds will quiz you on discrete math to probability theory to geometry to information theory to literally anything. My advice is if you're a software engineer interviewing for a hybrid of finance and tech places, timebox yourself in this category.

---

I have not seen an interview question this cycle that was an exact question I've seen online or in books. Your mileage may vary.

# The interview process

Interviewing and talking with all of these companies was a great experience, even with COVID in place. Obviously, as shelter-in-place continues, these companies are conducting virtual on-site interviews and trying to make this process as smooth as possible. Without getting into the specifics, I'll outline some common things I've noticed during the process in the COVID era.

- Many companies use Zoom or Google Hangouts for their on-sites.
- They give you ~15 minute breaks in between interviews for water breaks.
- Some companies give you a longer lunch break (45 mins to an hour).
- If you're interviewing for a company in another time zone, prepare to wake up in the early AM's or interview in the late afternoon (sometimes after dinner).
- Conveying an idea takes slightly longer because you're not drawing on a whiteboard. Some companies have virtual whiteboard apps and others allow the use of Zoom whiteboards.
- **Some companies added more interview rounds for virtual on-sites.** Apparently more people are getting into companies with subpar technical skills during COVID and they're making the process more selective. I think this can also be due to the increase in competition due to unemployment rates increasing.
- Feedback and communications with recruiters is generally faster.

## More interview rounds during COVID

The bolded text might scare you as a potential candidate, but don't worry too much. The added questions aren't testing you if you know how to implement a bloom filter or a fibonacci heap or something niche. They usually test on the *coding abilities of the person and how well they'd actually ramp up in a novel, collaborative environment*. This can manifest itself in multiple ways - live debugging session with a new codebase, reading documentation to work with new technology, or a collaborative brainstorming sesion for a hard(er) problem. If you're a decent software engineer you shouldn't worry about these as much.

## Dealing with time zones

*One of the biggest struggles I had during the interview process was adjusting my sleep schedule to wake up at 5-6AM to make sure I'm awake and on time for the interviews in New York/Chicago (I'm in California so this was a 3 hour gap)*. Usually, companies would fly you out the day-of or the day before the on-site. I've always felt tired after a plane flight and was able to get a good night's rest before the interviews in the past. With COVID, everything is virtual and the companies expect you to interview at their hours.

Even with slowly adjusting my sleep schedule over a week or two I still had trouble with sleep. Personally, I get pretty nervous before an on-site and I'd need to feel adequately tired to get a good night's rest instead of tossing and turning in bed. With the clock turned 3 hours back, I suddenly found myself not tired enough to sleep on time the night before the interview(even with a whole week of adjusting). This led to me consistently getting 6-7 hours of sleep instead of the 9 hours of sleep I usually get on game day, which really sucked.

Ultimately, I have no idea how much the sleep problem really affected my performance, but it was enough to shake my confidence going in.

*NOTE: +1 to Citadel for proactively breaking my on-site over multiple days so I can have a sane sleep schedule for their interviews. This might depend on the specific team you're interviewing with.*

## Which ones were the hardest?

This is subjective, and the question can be broken up into multiple components:

- **Time pressure - Jane Street**. This is probably why I failed their interviews, which were a bit longer than usual. I tend to explain my approach before coding anything to get a confirmation on the interviewer's side that I'm on the right track. I probably spent too much time explaining and didn't have enough time to finish the code for some interviews.
- **Math questions - Citadel**. They asked me some _really_ interesting math problems that aren't related to finance at all. I don't think they expect the interviewer to get 100% of the questions since whenever I solved one the interviewer was ready with another. HRT also asked some.
- **Systems design - HRT**.
- **Outside-the-box problems - Databricks**. They conduct one of the most unique interviews I've ever had.
- **Language specific questions - Citadel/HRT**. Grilled me a lot on low level C++ stuff.
- **Length of interview - HRT**. I started at 8AM PST (I requested to move it to 8AM from 7AM) and finished at ~2:30PM. **That is a whopping 6 hours and 30 minutes.** I also did a coding challenge and 2 phone screens before I moved to on-site, totalling almost 10 hours for interviews.
- **General algorithm questions - Jane Street/HRT**. I think Jane Street was a bit harder given the time pressure. The flavor of algorithm questions are also different between these firms.

Once again, this breakdown is **subjective**. I obviously have a lot of experience interviewing with Silicon Valley companies so the novelty of questions from the finance companies added to the difficulty.

# Making a decision

This was the hardest part for me. I spent two weeks suffering from analysis paralysis. I would wake up wanting to go to company X but wake up another day wanting to go to company Y. *The tug of war between different recruiters stressed me out - I couldn't sleep, I couldn't eat, I couldn't do anything during the days and I unknowingly stressed out those around me with the process **(special shoutouts to Ben, Eric, Nishanth, Mickey and Kibeom for dealing with my BS)***. I used the following criteria to make my decision:

- **Manager support** - How much support would my manager give me to learn new things and work on impactful projects? Is my manager someone I admire and want to learn from?
- **Tech debt** - How much tech debt is there and do I have to deal with it?
- **Project flexibility/impact** - Are the projects assigned to me or do I have autonomy to choose what I think is most impactful/interesting? If the project is assigned to me, is it something that I'll enjoy doing for a few years at least? Will the acquired skills associated with the project be transferrable?
- **Ability to learn** - How collaborative is the company? Are my coworkers going to be domain experts? Can I engage in discussions with critical problem solvers?
- **Stability** - How high is the attrition rate? There isn't anything bad about the idea of removing low performers to keep a company efficient, but the pressure to deliver often comes at a detriment of learning new things and keeping the infrastructure robust.
- **Location** - Between bay area, Seattle, Chicago, and New York City, which one do I want to go to the most? This was a complicated decision.
- **Compensation** - Do I need to worry about money or can I just put my head down and build things? How much risk is associated with the package?
- **Culture/work-life balance** - Is the company individualized or collaborative? What's the managerial structure? Are there lots of politics? How long do people usually work? I don't want to burn out and stop working on my blog and pursuing other hobbies, as that will likely take a toll on my mental health and ultimately cause me to leave the company anyways.

I made a weighted linear model consisting of these features and used that arbitrary numerical output to reduce my choice to between Citadel and HRT which were exactly equivalent in numerical value. In the end, I decided using my gut feeling and went with HRT.

## The culture and the "small" things count

My decision was largely driven by the vibe of the people I talked to:

- My friends at HRT, especially Ben, were big factors for my decision. 
- HRT set up a virtual dinner with me and three potential new grad coworkers, with delivery app credits. It made me feel valued as a candidate since the three coworkers were so friendly and ready to chat.
- The hiring manager understands me very well - he knew what projects I was interested and what my hesitations were when it came to making a decision and addressed them. In the morning before my final deadline, he sent me a heartfelt e-mail recapping many of the topics we've discussed and supporting me.
- The recruiter also corresponded with me every week and was receptive to my feedback.

Aside from HRT, I'd like to thank Xinan and Xing for being amazing hiring managers who spent **a lot** of time talking to me and helping me throughout the process with invaluable advice. Their experience, honesty, relatability, and transparency made the decision so much harder to make (in their favor, of course). Because of them, I felt like I wasn't just another data point in the interview pipeline, and that they were advocating for my success regardless of where I end up.

---

Although I felt like the decision making process was least impacted by COVID, if I were able to meet potential co-workers face-to-face it would've been clearer which place I would've liked to work at.

# Conclusion

Interviewing during COVID is definitely a different experience. It was a lot of stress and I'm glad to be over with it. In the future, for my sanity, I would not go through the process with so many companies in different time zones at the same time. I'd like to thank my family & friends for supporting me through the entire process and cheering me on. I couldn't have done it without you all :)

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>