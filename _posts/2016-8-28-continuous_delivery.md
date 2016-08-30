---
layout: post
title: Continuous delivery
excerpt: Releasing production changes frequently and faster using VCS.
tags: [release, engineering, startup, production, version control, git]
comments: true
---

You often hear words like "release", "pipeline" etc when you're working in a product based company.
They usually follow [Continuous Delivery](https://en.wikipedia.org/wiki/Continuous_delivery) because it helps to reduce
cost, time and risk of delivering changes by allowing for more incremental updates to applications
in production.

<figure style="display: block;">
	<a href="http://rootpy.com/images/Continuous_Delivery_process_diagram.png"><img class="image_border" src="http://rootpy.com/images/Continuous_Delivery_process_diagram.png"></a>
	<figcaption><a href="http://rootpy.com/images/Continuous_Delivery_process_diagram.png">Code passes through various stages. Source: Wikipedia</a></figcaption>
</figure>

   This release cycle looks more like a network when we define a flow for it.
It must go through all phases even if it's a simple fix or an enhancement/big feature.


  You've to design it in such way that everyone will able to follow standard practices and guidelines.
Your project should clean no matter how many releases you've.
Okay. We agree on above points. So where do we FOCUS for development workflow?


> In continuouse delivery, You've to deliver a set of fixes / enhancements / features in short period.
To achieve this, multiple developers need to work simultaneously.

### The Version control WAY

It plays an important role in any release.
ohh really?? This is the first reaction of most of the developers.
some of them can say testing or algorithm or deployment etc

Actually they simply don't know power of [version control system](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control) 
otherwise they would agree on above point.


### Why version control plays important role.

* Developers can work on simultaneously without stepping toeing each other legs.
* Ability to go back in older version quickly.
* Track metadata about changes.
* Developer can test his/her code thoroughly and share it with colleagues.


> When you've releases in every week, You have to make sure that public branch like master shouldn't have broken changes.
  Also there is no point to deploy and test code which is broken. 

We've been following codebase's [merge request flow](https://support.codebasehq.com/articles/repositories/creating-a-merge-request) at [ShopSocially](shopsocially.com). It's similar to github's [pull request flow](https://help.github.com/articles/about-pull-requests/).


### How is merge request flow helping in continuous delivery / releases?
Most of the times, developers get conflicts whenever they merge their code into target branch.
It's expected due to multiple colleagues are working simultaneously.
Production release gets delay due to conflicts at target branch like develop.

Merge request helps to take a call whether to merge your branch or rebase it

>**It shows no conflicts with target branch.**
<figure style="display: block;">
	<a href="http://rootpy.com/images/merge_request.png"><img class="image_border" src="http://rootpy.com/images/merge_request.png"></a>
	<figcaption><a href="http://rootpy.com/images/merge_request.png">No Conflicts with target branch</a></figcaption>
</figure>
You're lucky and reviewer can easily merge your code in target branch.


>**It show conflicts if your branch has with target branch.**

<figure style="display: block;">
	<a href="http://rootpy.com/images/merge_conflicts.png"><img class="image_border" src="http://rootpy.com/images/merge_conflicts.png"></a>
	<figcaption><a href="http://rootpy.com/images/merge_conflicts.png">Conflicts with target branch</a></figcaption>
</figure>

* In this case, You've to do rebase with target branch.
* Developer has to fix conflicts in his/her branch.
* It's far better than manually merging multiple branches in target branch.
  Because conflicts need to be resolved by developers in public branch like develop.

Now we come to know that how is merge request helping in continuous delivery.

#### Advantages using merge request

* It tells you when to do merge or rebase.
* Feature branches always sync up with public branches.
* Public branch like master, develop always clean. So Conflicts need to be resolved at developer's branch.
* Code review of large enhacements /features can easily done as it shows changes with respect to target branch.
* Reviewers can see the changes anytime. Also they can comment on it.

#### Disdvantages using merge request
* It's pain to resolve conflicts if your branch is far behind from target branch.


You can run existing test cases whenever features/fixes gets merge.

In short, version control system is an important key in continuous delivery.
It all depends on how you DEFINE the workflow for development.
