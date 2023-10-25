# Collaboration Activity

# ESIIL Hackathon 2023 Collaboration activity
### Instructor: Culler
Git and GitHub are great tools for collaboration. They allow multiple people to edit their own copies of the project and merge them together later.

![A basic git workflow represented as two islands, one with "local repo" and "working directory", and another with "remote repo." Bunnies move file boxes from the working directory to the staging area, then with Commit move them to the local repo. Bunnies in rowboats move changes from the local repo to the remote repo (labeled "PUSH") and from the remote repo to the working directory (labeled "PULL"). ](https://cdn.myportfolio.com/45214904-6a61-4e23-98d6-b140f8654a40/68739659-fb6f-41e8-9813-32e1de3d82c0_rw_3840.png?h=5c36d3c50c350a440567a1f8f72ac028)

> Image source: [Artwork by @allison_horst](https://twitter.com/allison_horst)

Git and GitHub can also be very frustrating to use if multiple people are trying to edit the same part of your project at the same time. If you think about this for a moment, you can see why. It is hard enough for a human who has a lot of context about a project to merge multiple peoples' contributions. It's even harder for an algorithm!

![Cartoon of the GitHub octocat mascot hugging a very sad looking little furry monster while the monster points accusingly at an open laptop with "MERGE CONFLICT" in red across the entire screen. The laptop has angry eyes and claws and a wicked smile. In text across the top reads "gitHUG" with a small heart.](https://cdn.myportfolio.com/45214904-6a61-4e23-98d6-b140f8654a40/bac2b5d6-5f71-4bb2-8904-03af45448ac2_rw_1200.png?h=d9a9aef39ce69d8d04c1f0c450980030)

> Image source: [Artwork by @allison_horst](https://twitter.com/allison_horst)

## Principles of Collaborative Coding

For this hackathon, we would like you to try out our method of writing collaborative code, even if you usually use a different method. It is important to us that **everyone at all experience levels** be able to contribute to your project, and we hope it's important to you, too! We are advocating for the following two principles of collaboration for the Hackathon, because we think they will allow your group to make the best use of everyone's expertise, including both those with lots of coding experience and those who don't have much coding experience yet. This method should also avoid merge conflicts. If our method isn't working for your group, that's ok, too -- you can work as a group to modify it as needed.

### Principle #1: Never Code Alone

When working with your hackathon team, use **Pair or Group Programming** to bring together contributions from several people. We recommend a couple of rules for *inclusive* pair programming:

1. One person, who we call the **typist** has keyboard control at a time.
2. **No one else edits the file the typist is working on** in any version of the code repository until the typist has committed and pushed their changes back to GitHub.
3. The typist **only types things other group members say to type**. The typist can, of course, suggest something to the group and contribute, but they need to explain the change they want to make get buy-in from the group before changing the code, just like everyone else. What we're trying to avoid here is the typist coding by themselves.
   > Many people who are new to pair programming want to put the person who knows the most code in the typist position. We suggest the opposite - the typist is a good position for relatively new programmers, while experienced programmers might find it more difficult to go through the pair coding process instead of writing code alone.
4. The other person or people are code **architects**. They figure out how to structure the code and make suggestions to the typist. Other good ways for architects to contribute include doing research and writing pseudocode or flow diagrams to help the typist know what to type.

We believe that pair programming, when done using our rules, is inclusive. We know it can be uncomfortable to know how to fix a problem and refrain from doing it yourself. Please do not take control away from the typist, or ignore your architects! Remember that these rules are intended to make sure that everyone's input is valued and incorporated into your project. You might need to code a little slower than you might by yourself, but trust that the code your write will be higher quality. Put another way -- by choosing to pair program, we at ESIIL are **committing to quality over quantity, and to inclusivity over individual stardom**.

### Principle #2: Write Modular Code

You might be thinking, "well, that's great for 2 or 3 people, but we have 7 on our team". If so, you'd be absolutely right - pair programming doesn't work well for more than 3 people! The following are our recommendations on how to split up tasks:
 * We recommend splitting your group into subgroups of 2 or 3
 * Each subgroup has one file in your GitHub repository. This will avoid git merge conflicts and also allows the subgroups to use whichever programming language they prefer.
 * Save data to files or databases to transfer between groups. If all groups are using the same programming language, you can serialize data (e.g. pickle or RDS). Otherwise, use an open file format like csv or sqlite.
 * **Agree in advance** about how to format your data. For example, imagine that subgroup A is working on resampling daily precipitation data to monthly, and subgroup B is plotting that data, they might agree to use a csv file with two columns named 'date' and 'precipitation', with the date in 'YYYY-MM-DD' format.

Your projects will all be different, but most Earth Data Science Workflows can be split into the following sequential modules:
  1. Downloading and wrangling data
  2. Calculate statistics or apply a model
  3. Visually and verbally convey your results

Steps 2 and 3 can use randomly generated data until they have the real data.

## Skills you need to work on code in larger groups

This activity is not about teaching you to code; we're trying to get you to practice techniques for getting the most out of your group. 

  > Note that we are skipping a number of industry best practices you may have heard of, such as agile sprint techniques like scrum and kanban, as well and git branches. If your **whole group** already uses those practices...well you probably don't need this training, and feel free to designate a project manager and get going! Otherwise, we think sticking to our one-person-per-file-at-a-time rule is the best way for everyone to have a positive Earth Data Science experience.

We'll provide examples of a few technical skills needed for effective data science collaboration:

  * Exporting data to standard open file formats AND/OR serializing objects
  * Importing data from standard open file formats AND/OR deserializing objects
  * Using conditionals to cache your work
  * Generating random data that adheres to your group's agreed-upon standard

In this activity, you will practice these skills in order to collaboratively calculate and visualize a trend line for mean annual temperatures for Denver, CO.

***

# GitHub Essentials
### Instructors: Culler

<img height="100" src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png">

**About GitHub:** _GitHub is a web-based platform that provides version control and collaboration features using Git, a distributed version control system. It enables developers to work together on projects, track changes to code, and efficiently manage different versions of the project. GitHub is widely used in the software development industry and is an essential tool for collaborative projects and maintaining code quality._

***

# The CyVerse Discovery Environment & Data Management
### Instructors: Verleye

<img height="100" src="https://cyverse.org/sites/default/files/inline-images/PoweredbyCyverse_Logo.png">

**About CyVerse:** _CyVerse provides scientists with powerful platforms to handle huge datasets and complex analyses, thus enabling data-driven discovery. Our extensible platforms provide data storage, bioinformatics tools, data visualization, interactive analyses, cloud services, APIs, and more. CyVerse was created in 2008 by the National Science Foundation and now supports over 100,000 researchers in 169 countries. CyVerse has appeared in more than 1,500 peer reviewed publications, trained over 40K researchers and instructors, and supports $255M in additional research funding by NSF. Led by the University of Arizona in partnership with the Texas Advanced Computing Center and Cold Spring Harbor Laboratory, CyVerse is a dynamic virtual organization that fulfills a broad mission to enable data-driven, collaborative research._

***

# Opening an Instance in JetStream2
### Instructors: Verleye

<img height="100" src="https://jetstream-cloud.org/images/logos/jetstream2-head-logo.svg">

**About JetStream2:** _Jetstream2 makes cutting-edge high-performance computing and software easy to use for your research regardless of your project’s scale—even if you have limited experience with supercomputing systems. Cloud-based and on-demand, the 24/7 system includes discipline-specific apps. You can even create virtual machines that look and feel like your lab workstation or home machine, with thousands of times the computing power._
