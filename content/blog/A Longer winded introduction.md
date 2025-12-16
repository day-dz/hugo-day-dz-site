+++
title = "A Longer Winded Introduction"
date = "2025-12-13T21:57:08Z"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "Jack"
cover = ""
tags = ["Introduction", "Short read"]
keywords = ["", ""]
description = "If you want to read more about my journey to where I've ended up then click below!"
showFullContent = false
readingTime = true
hideComments = false
+++
## A longer winded introduction - 13/12/25

Hi, I'm Jack and I work as a cyber security analyst within the NHS.

Today was the first day that I decided to start something that I had wanted to do for a while.  Not neccessarily "start a blog" but more have a way of documenting what I'm doing to learn more about the field that I work and have a personal interest in, Cybersecurity.  I'll use this first post as an introduction to how I ended up in my role and the stops that I made along the way. 

My cybersecurity interest was a bit of a late bloomer as even though I studied Ethical Hacking at university, I think I completed it at the wrong point in my life as whilst the topic interested me, I just did the bare minimum and didn't throw myself in to the culture fully.  I got my degree as Covid started to force everyone into lockdown and job opportunities took a weird pause as the whole world was uncertain about what to do next. During the first 6 months of covid I didn't continue to do anything in my personal life to retain the knowledge I had gained from my course. I stayed isolated with my flatmates and applied for all kinds of jobs to be able to afford rent, not particularly focusing on the field that my degree was in but more general IT where I thought I'd be more capable of handling the daily work.

After a handful of interviews and a lot more handfuls of rejection emails I finally got a foot in the door in January 2025, working on an IT service desk for the NHS. I worked in this post for almost exactly a year, learning different tools, business IT architecture and the silos that came with it before applying for a role as a system support technician within the Identity and Access management team.  This was the first step into a security related field and began to re-light the interest I had initially.

The job started with a lot of learning as there were hundreds of systems to learn about and my first hands on experience with managing an Active Directory environment. I started with the GUI management tools like Active Directory Users & Computers, getting to grips with user management, security groups and group policies. I realised quickly that I could make my life a lot easier by learning how to automate tasks and audit more efficiently if I made use of powershell for these tasks.  The team had not made much use of powershell in the past so I was building processes from scratch that the whole team used to speed up tasks.

After 7-8 months in this post there was a expansion plan that was brought forward for the team to expand into more automation and scripting to aid the team.  With this expansion came 2 new jobs as automation techician and senior automation technician.  I managed to interview and obtain the automation technician role and this is where my knowledge of powershell, graph and automation as a whole ballooned.  Working in this team I was making use of multiple API integrations, automation tools, powershell and more to improve the work flows that my previous colleagues use. An example of the type of work we did was to create an inactive user process from scratch.  This ran monthly, starting with a PS script to look at lastlogon attributes for all of our active user object within AD combined with entra reports for exchange Online logons.  Any that fell out with our 60 day period were highlighted for disablement, at which point we integrated with our Service desk tools api to raise a ticket highlighting to the user that if there is no action on the account that it will be terminated.  A scheduled check was then made in the service desk platform to see when a week has passed and then raise the termination and start another shell script to terminate the account within our local AD.

After a year and a half in this post there was a refresh of our internal Cybersecurity team and I felt that it was the right time for me to try and move into the field that I had been interested in from the start. I applied and initially was the reserve candidate but following the other person declining the job offer it was then offered to me. It's been just over a year since that point in which time I've been learning as much as I can on the job as well as going back to university to complete a masters in Cybersecurity at the same time. Next April I will hopefully graduate with my masters and then go on to getting more job related certifications to progress my knowledge further.  This blog/posts collection will be used to document the bits of work that I find interesting or to showcase elements that I have built during learning of topics.