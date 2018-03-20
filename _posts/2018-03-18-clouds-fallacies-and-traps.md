---
title: Clouds, fallacies and traps
layout: post
tags:
- cloud
- iaas
- paas
- saas
commentIssueId: 2
---

Working in a team that 100% utilises cloud services being IaaS, PaaS or Saas to make things rolling and the product owners happy, may look as a shopping promenade at first sight, however it requires some high-level thinking, anti-pattern recognition and careful design. Along the way, you also learn that you have to build a lot of integration glue and unfold your system architecting skills in ways you have not predicted at the beginning.
<!--more-->

The advantages of using the cloud are generally known because salesmen talk about it. Depending on the provider and the expertise of the engineers that run the service behind the scenes, you can get stuff like:
* upgraded kernels without rebooting instances or downtime
* sophisticated wrappers around complex datastore operations and APIs
* ready-to-use metrics shippers with high granularity and configurability
* notifications about security patches, required maintenance operations or serious issues with you infrastructure

Things like upgrading multi-100GB MongoDB clusters without even moving your little finger or gaining visibility into precalculated statistics over hidden PostgreSQL system tables becomes a self-service norm. 

However, the reality is not perfect at all and the quality gap between cloud providers is huge. After a few months of experience with cloud services, you come across a lot of well-disguised traps that frequently take advantage of the common fallacies of cloud advocates. For the cloud skeptics, the argument most of the time boils down to:

> "You can't believe that everything is going to work out well in the cloud, aren't you?"

Of course not, but usually the argument fails to highlight the aspects that you should take into consideration when a cloud service evaluation has to be done or an engaging operation in your current cloud infrastructure should be executed. 

This post is about outlining the main pain points I recognised while using the cloud goodies, grouped into meaningful categories, so that they can be used as a reference and reminder both for skeptics and advocates.


## Scaling ##

Cloud is all about infinite scaling, isn't it? And autoscaling is what makes this even better. Well, for a start I wish you never meet this kind of providers that have implemented only the upscaling part of autoscaling! Yes, this is true. Some providers out there scale your instances based on a resource metric over a documented (or not) evaluation window, but they do not scale them down when the party is over. If you are not cautious enough, you learn about it at the end of the month when the bill slides under your door.

The other thing that a cloud provider should be tested against, is the maximum scale that they are able to operate a specific technology. Your Solr index has grown 100X fold since you started using it or the Cassandra cluster that lies behind your log management platform should now handle 10X the log events volume since last year. You should pay attention and ask beforehand what is the maximum capacity and complexity that these cloud guys you bought your service from can handle. Sky is not the limit and you may have to plan a migration to a clustered variant with a slightly diffferent API specification or land to a completely different technology that will take time for production code to be refactored and the learning curve to be advanced.

You should finally pay attention to instances flavors and pricing. Some providers are very inflexible when they specify their flavors and the price may be very high for your future use cases.


## Monitoring and visibility ##

If you are lucky, your cloud provider will provide you with logs and metrics. Do not celebrate about it though unless first you double-check that the "read_iops" metric that pops up in your dashboard is an IOPS metric indeed and not some exotic calculation over the blkio kernel subsystem. Make sure that capacity resource metrics can also be expressed in % utilisation, because otherwise knowing that you have 5.2M bytes free memory is of little usage.

Some cloud providers may also provide an alerting framework for you but it may or may not fulfill your needs. If there is no provided integration with your alerting provider of choice, be prepared and build integration glue to ship stuff there.


## Advertised features and integrations ##

If one thing should be told, then this is that you should not take as granted the operational readiness of a feature that is advertised or documented in the service provider's  website. 

The integration, you and your Security Officer decided to enable in order to start auditing your users actions, may be one click away (check!), the documentation is there (check!) and seems up-to-date (check!). When you enable the integration though, you realise that it simply does not work and start questioning if you are the only person on Earth that tried to use it or at least paid attention.


## Deprecations, EOL and new features ##

So your cloud service provider sent an email notification 3 months ago saying that they are going to deprecate their legacy alerting framework. Or they just posted a blog post saying that they are going to drop SSLv3 from their supported protocols. If you expect that you are going to get a big flashy notification in your UI some days before the EOL or something like a last email notification that you have to switchover to the new and shiny stuff, then you are overoptimistic. And by being overoptimistic in that case, you may end up with no alerting framework during the weekend or clients connections getting rejected.

Not to speak about new cloud software releases that may change the behavior of your cloud machinery. One day your WAF rules do work as expected and the other a new feature that was just introduced creates artificial traffic and your website starts blocking random IPs. Hopefully, these incidents are very rare, however you should and have to pay attention to your most business critical cloud services blogs/newsletters/feeds.


## Documentation ##

The quality of documentation varies a lot between cloud service providers. You should take as granted that some bits may be missing and other may be not exactly accurate. One way or another you need to proof-check your procedures and do your dry-runs especially if you are making your study while preparing for a complex maintenace task.

Some providers stand out from the others and open sourced their documentation, so take the time and issue a PR if you find an inaccuracy. Your SRE colleague right next to you will not fall for it the next time she encounters the same issue.


## Support ##

There are different types of support contracts you can buy from your provider but in general these contracts are expensive. Or at least this is the first impression until you realise that the exprertise of the people that form the *gratis* support layer varies a lot and most of the time leads you to frustration or extensive waiting for an authoritative answer or escalation. Without a professional or enterprise support contract, do expect that an explanation for missing data points from your monitoring provider or unexpected ping failures to your favorite cloud load balancer may take up to days or weeks.

*Side note:*  These expensive enterprise contracts most of the time cover you around the clock for any day of the month and they will do their best to response to an incident as fast as possible. The investigation part though on what went wrong with your case may take up to 1-2 days or even more at some cases.  


## Bugs and hacks ##

Even the better and most famous cloud providers may have bugs in their pipelines. And we are talking about pipelines that you are using extensively but one day, you decide to do something slightly different, like making your master database read-only before promoting your slave. Only to realise that the failover will not succeed because the read-only flag propagated to your slave (thus any `ALTER` statements against the slave will fail) and nothing in their automation pipeline tried to check about this condition and revert it or at least notify you about it.

Do your dry-runs and plan for failure even for your most trustful provider.  


## Upstreams and security notifications ##

Do not expect your cloud provider to notify you about a data corruption bug that was resolved in the last upstream version of your production database. This is your job, fool yourself not. There are some guiding stars out there that will let you know, but again, do not expect from them to make these versions available in a couple of hours after the upstream release. Time will pass.

Truth be told, it will make you happy one day to receive a notification from your AMQP provider that there is a new security patch that should be applied to your AMQP instances. This is not the norm though, albeit it should.

## Epilogue ##

Now, let's take a break! 

The list above is not an exhaustive one. There is a lot more that can be said for performance expectations, high availability design, in-built collaborativeness to name a few and the list can grow and grow up to the sky. I hope that the topics above were appetising and challenged you enough in order to ask more in-depth questions the next time you will find youself messing up with your current cloud service or just started POC'ing one of them out there.

Stay calm, pay the bill but question everything :)