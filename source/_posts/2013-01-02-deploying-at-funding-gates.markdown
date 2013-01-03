---
layout: post
title: "Deploying at Funding Gates"
date: 2013-01-02 17:10
comments: true
categories:
---

*by [Matt Rogish](http://www.mattrogish.com/) ([@MattRogish](https://twitter.com/MattRogish))*

Deploying, it seems, is hard. Folks have written very complex mechanisms to
maintain different [releases and branches](http://nvie.com/posts/a-successful-git-branching-model/):

{% img center /images/deploying/gitflow.png %}

Wow! That seems like a lot of work. You need to keep track of releases, hotfixes,
and make sure that any last-minute work is propogated to all the relevant branches.

This feels very [un-agile](http://www.rendell.org/pebble/software/2010/02/09/1265756844985.html) and not very lean.
Moreover, it requires a gatekeeper (or more!) who decide if/when a release is to go out,
appropriate documentation (here's version 1.x release notes), and individual developers
scrambling to get feature X or bug Y into the current release.

That's a lot of headaches! Luckily, we're working on web application software
so we don't have to worry about cutting releases and delivering them to end-users.
Press a button and everyone gets the new feature!

As we're optimizing for cycle time (how long it takes for a feature to hit production
after starting) any hand-offs or waiting introduces costly delays. Code that has
been written that is not deployed to users is [wasteful undelivered inventory](http://www.joelonsoftware.com/items/2012/07/09.html).

### Process

1. Developer works on feature
1. Developer commits and pushes to master
1. [The system](http://www.circleci.com) automatically deploys to production

{% img center /images/deploying/ABC.jpg %}

There is no next step. That's too simple, right? Right. There are some pre-requisites
that make this possible:

* We require high unit and functional test coverage to ensure nothing gets broken.
* At every push to a remote branch, a Continous Integration server runs and rejects a build
if a test fails, notifying the entire team of the failure.
* Instead of working on long-running feature branches, we prefer to work with short running
local branches or directly on master (up to each developer to decide)
* All features that aren't "ready" (either not finished yet or not ready for "marketing" purposes) are
disabled via feature flags on staging/production.
* Features we're currently working on are enabled in development and QA/test
* Any server errors are immediately delivered to the team chat/email and we have other
monitoring systems in place that would identify "something bad" happening in production


### Benefits

This gives us the ability to work on master, have master always deployable,
and be confident that code that isn't ready for public consumption won't appear.
Someone identifies a bug? I can add a test to catch it, fix it, and have it deployed
to production within minutes - all without needing to get permission from anyone
or perform git gymnastics.

Deployment is now a non-issue. No one needs to be "taught" how to do it. There
isn't even a button that needs pressed.

Continuous deployment isn't crazy, and it absolutely scales with larger teams.
Plenty of big companies like [Github](https://github.com/blog/1241-deploying-at-github),
[Etsy](http://codeascraft.etsy.com/2010/05/20/quantum-of-deployment/), and the
poster-child, [IMVU](http://timothyfitz.com/2009/02/10/continuous-deployment-at-imvu-doing-the-impossible-fifty-times-a-day/)
are probably delivering as you read this.

You can add [Funding Gates](http://www.fundinggates.com) to that list!