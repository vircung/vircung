---
layout: post
title:  "Slightly advanced deployment workflow"
date:   2015-04-13 23:00:00
---

Over 5 years ago [Vincent Driessen][nvie-twitter] shared with us his well established [git workflow][git-branching-model]. It's very robust and simple (when you wrap your head around it). It almost perfectly fits into agile methodologies. In one of my past projects it was easy to drop in along with migrating from svn to git, and include it to existing [Continuous Integration][ci] system. And everyone was happy. How about more recent one? Happily not so easy. That's good i like challenges.

<!-- more -->

### What's wrong?

Over last 6 months i wanted to adopt it to one of projects in my job (and migrate from filthy svn to git). Of course it's not so easy because of internal political reasons, policies, produces and tons of documents that has to be signed by millions of people. Hopefully i received information that this can be possible in Q3 or Q4 of 2015's. So let's roll up sleeves and prepare some flow charts and graphs.

## Current repository situation

If repository has not received any thought beforehand and was managed in free-style way there is no other possibility that it is/was mess. When i heard the story i was amazed. Long story short. At the beginning everything was kept in ```root``` folder. Then ```src``` folder appeared. After some time some real structure came into being with folders like ```trunk``` or ```branches``` and other. Each release is a new branch which is left alone even if new release is published. Each production bugfix is implemented in ```trunk``` and current release ```branch```.

![I was amazed](/assets/im-not-even-mad.jpg)

## Deployment flow

At the end of the day internal policies (which i always rant about) became an ally. Hopefully deployment process is well managed and streamlined. I like that there are separate stages for both [SAT][sat] and [UAT][uat]. That gives us, developers, after completing scheduled User Stories time to go through Technical Chances. Sadly the client is more interested in new features that technical and internal improvements but who can blame hit for this? So here is release stages breakdown and deployment flow. As you can see each stage has own isolated environment.

![Deployemnt workflow](/assets/current-deployment.svg)

## Adopting git-flow

![Release flow](/assets/release-workflow.svg)

Adjusting standard release workflow was not that big issue. As meeting went along i had to be agile and make some slight changes to well known nvie's workflow. As you can see on current workflow i had to split ```release-*``` branch in two.

When new release cycle is started development code is freezed and no new features are added and SAT cycle begins. New branch ```sat-*``` is created with ```develop``` as a base and deployed to it's own env. When all found bugs are fixed and verified on ```development``` env new SAT iteration begins along with deployment and retesting on SAT env.

When no new bugs were found UAT cycle begins. Old ```sat-*``` branch is changed to ```uat-*``` and deployed on it's own env. If any bug is found it have to be fixed and deployed as soon as possible. If all bugs are fixed and all fine and dandy ```sat-*``` branch is merged to ```develop``` to include all bugfixes and also it's merged ```master``` and deployed to production.

![Hotfix flow](/assets/hotfix-workflow.svg)

If critical bug is found on production new branch ```hotfix-*``` is created with ```master``` as a base and deployed on it's own env. If it's fixed and ```hotix-*``` is changed to ```uat-*``` and UAT cycle begins. If it's all ok ```hotfix-*``` is merged back to ```master``` and deployed to production. It's also merged to ```develop```.

## Summary

Changes made are minor and right now obvious. But back then i gave it fair amount of thought to do it in cleaver way. Benefits from migration to this approach and git are significant to me. For example better changes isolation, repository performance boost, ability to manipulate commit history before push them to server, less hassle for CI configuration for each release.

I hope this approach will be accepted and possibly spread among other projects. Also i found that solving this problem and preparing myself to explain changes proposed by me gave me much satisfaction. Maybe this is the point to think again about my career path as a software developer :)

## Fun facts

I did [PoC][poc] for myself that repository can be migrated. The answer is **yes**. But.

It took about **4 hours** to convert svn into git repository. I did wrong the configuration because folder with branches is named ```branches``` instead of ```branch``` so i migrated only half of repo.

When i fixed it (i don't know exactly how much time it took to fix it along with test fetches) i managed to migrate whole repo in about **9 hours** (over 8k commits). I wonder if it's a network configuration issue or just it takes so much time for every larger svn repo or this particular svn server.

Sadly i didn't stored any statistics about previous migration mentioned at the very beginning. It would be interesting to compare

[nvie-twitter]:https://twitter.com/nvie
[git-branching-model]:http://nvie.com/posts/a-successful-git-branching-model/
[uat]:http://en.wikipedia.org/wiki/Acceptance_testing#User_acceptance_testing
[sat]:http://en.wikipedia.org/wiki/System_testing
[ci]:http://en.wikipedia.org/wiki/Continuous_integration
[poc]:http://en.wikipedia.org/wiki/Proof_of_concept