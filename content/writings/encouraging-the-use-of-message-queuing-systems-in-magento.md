---
title: "Encouraging the use of Message Queuing Systems in Magento"
description: ""
tags: ["magento", "software architecture", "rabbitmq", "message queuing systems", "performance engineering"]
categories: [ "Computer Science" ]
draft: false
date: 2018-01-30
---

## Main architectural patterns in Magento

The deployment of Magento as an ecommerce platform doesn't come without its challenges in regards to performance, scalability and optimisation. This is due to several platform characteristics, with a very limited list as follows:  

* **A** - A somewhat large codebase, with the template block rendering approach for the frontend being synchronous, rendering one template only after the previous one finishes.
* **B** - High amount of repetitive synchronous requests to storage systems (database, cache), blocking code execution in the main process.
* **C** - Synchronous API calls to external systems, blocking code execution in the main process.

Please note my focus on the server-side and backend architecture on this list. It is also fair to say that these are very common challenges in web development.

Magento feels fast in the frontend when its catalogue doesn't change much, or doesn't depend on external information to be rendered. In these situations, the HTML of the content pages (like the home page) and catalogue pages (products and categories) are retained in memory by Varnish after the first time they are rendered. On subsequent identical HTTP requests, Varnish responds with this same pre-rendered HTML in a very fast manner[^1][^2]. While this helps a lot in practice, it's not a one size fits all solution: several business require fast changing of in-stock/out-of-stock or other product page statuses, thus invalidating the Varnish aggressively. Once Varnish has to reach out to Magento for any information as its backend content provider, it becomes slow again.

Even in cases in which Varnish solves your problem, this still leaves Magento's dynamic pages (cart, my account) and controllers that receive POST requests potentially vulnerable to performance problems similar to the ones mentioned above.

For problems **A** and **B**, the ongoing effort within Magento to formalise APIs across modules using service contracts[^3] and the use of multiple database instances per entity[^4] could allow it to become more parallelization-friendly, thus helping with simultaneous/distributed processing of responsibilities (such as template rendering for a given module), also opening doors to asynchronous computation[^5] across modules of Magento. Efforts in this direction are still in very early stages.

For a subset of problem **C**, message queueing systems can be used. The rest of this post will abandon problems **A** and **B**, and advocate for a possible solution to problem **C**.


## API calls: problem detail and a possible solution

In my experience, synchronous API calls in Magento instances tend to occur in pages that serve dynamic/private content, or controllers that receive a POST request into the system. Some common examples are:

* Once a client uses the login function, an API call happen to an external system to indicate client ID X is now logged in.
* Once a client subscribes to a newsletter, an API call happen to an external system to indicate client ID X is now subscribed.
* Once a client places an order, an API call to an external system to indicate the order ID X was placed, together with the items that compose this order.

Note how these API calls are of the "fire and forget" type, in which Magento is simply propagating an event across other systems. In these cases, a subset of problem **C** originally state above, instead of doing an expensive API call that blocks the request and also holds the client in the frontend to wait for its result, one could write these messages into a queue in a very fast way instead and then just go on with process of generating the response. These messages are then to be processed later by another PHP process in an asynchronous manner.

This is where RabbitMQ[^6] comes in, as the added piece of technology in Magento2[^7]. RabbitMQ accepts a new message very quickly. A different process (running on a "worker" node, and not on the application servers) then consumes that message. In the above examples, the consumer process would remove the message from the queue and actually do the API call into the external system, marking the message as "read".

With this change, the extra waiting that happens in the frontend for the client is removed, and thus the application has a faster response time.

This approach may bring some new problems, such as that the message is guaranteed to be delivered to a consumer "at-least-once"[^8] meaning that there is an expectation that these messages are *idempotent*. For an example, in the case of pushing an order into another system, accidentally pushing that same order again (hopefully with the same ID) should not duplicate the order.


## Magento's take initially

Let's take a few steps back. After its spin off from eBay, Magento announced products beyond its highly popular e-commerce platform. Magento had an Order Management System and, later on, a Business Intelligence software added to its portfolio. The goal of these products is to work together with the e-commerce storefront piece, as to provide with the merchant a seamless experience across these products. After all, these are all different faces to the same end-to-end e-commerce process.

With this challenge at hand of integrating these systems, Magento chose to have message queueing systems to do some of the work[^9], then resulting in the addition of an AMPQ implementation and a RabbitMQ client into the Magento ecommerce platform.

The important bit: Magento decided to add this support into the *Enterprise* version of its platform, a paid for version which is normally not the first choice of usage for newcomers.

Internally, in the Magento Enterprise ecommerce platform, I could still only find one instance of the RabbitMQ system actually being used, which is inside the ScalableInventory module, in an asynchronous stock decrementing piece.


## Arguments to move the AMPQ implementation to the Community (free) version

The initial idea, that Enterprise clients would have access to the AMPQ/RabbitMQ client implementation, brought a problem very similar to the Magento1 era with Full Page Cache being an Enterprise-only feature. At that time, you would rarely get a community/ecosystem module that would work well with the Full Page Cache specification. Given it was an Enterprise feature (not available in the Community version), it was simply not taken into account by extension developers.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Magento vendor sold us an &quot;Enterprise&quot; version of an extension which seems to be identical to the Community edition one. No FPC support :|</p>&mdash; Richard Carter (@RichardCarter) <a href="https://twitter.com/RichardCarter/status/753173398507978752?ref_src=twsrc%5Etfw">July 13, 2016</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Magento ended up moving its Full Page Cache implementation (now supporting Varnish) to the Community version in Magento2, thus forcing extension developers to support it. At least indirectly doing so, as if an external module doesn't support it, clients would complain.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/magento?ref_src=twsrc%5Etfw">@Magento</a> extension developers, are you ready for Varnish Cache??  Test is live today!  <a href="https://twitter.com/hashtag/MagentoMarketplace?src=hash&amp;ref_src=twsrc%5Etfw">#MagentoMarketplace</a><a href="https://t.co/hqNJy0kaNs">https://t.co/hqNJy0kaNs</a></p>&mdash; Erika (@Tbotgento) <a href="https://twitter.com/Tbotgento/status/948274104184066049?ref_src=twsrc%5Etfw">January 2, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

A possible move of the AMPQ/RabbitMQ implementation in Magento2 from the Enterprise into the Community version could standardize and force extension developers and integration developers to behave better when developing integrations based on APIs, in a similar fashion that the Varnish move of Full Page Cache from Enterprise to Community in Magento forced extension developers to make their extensions Varnish-compatible out of the box.

Note how the current situation still yields several symptoms.

Extensions end up building their own not so robust queuing system (e.g.: integration extensions - normally in email marketing extensions, but there are other examples of not so popular extensions). So, as you go on installing multiple extensions that provide integration capabilities within Magento, each will create their on "synchronization" tables, and you end up with several approaches to queueing within your own application. This is a much harder scenario to debug/remember/maintain/integrate, and a side effect of the current Enterprise-only approach.

Note the following examples from the community. This is not an attempt to name and shame, given I believe the partner/extension makers ecosystem is still trying to figure out the "Magento way" of doing this:

* The Mailchimp extension, and its custom "batch" system[^10].
* The Dotmailer extension, and its custom complex database-based cron-based queueing system[^11].

This also makes it very expensive for extension developers to do an integration "right". This means that they can very quickly fallback to not so good practices, such as end up doing live/direct API calls hooked on controller actions, e.g.: pushing an order event to another system during the "place order" action, whilst it could have been done asynchronously a few moments later and saved time on the request. This is actually one of the most common symptoms for slowness: extensions that just hook expensive API calls in requests, such as "customer login", "place order", impacting frontend response time and building up requests on the server.

Further on this point, as a Service Integrator company (a Magento implementation partner company) is on a big project with several modules, they end up spending a large chunk of the time of the project modifying extensions, as to really "queue" their API calls themselves. This effort is then repeated over and over across implementations, and I suppose this is repeated by partners around the world.

Now suppose all extensions could just use a common existing Magento queuing system: *all you had to do as a SI is to make sure the workers processes were running*.

Finally, the argument is that as extension developers normally target both Community and Enterprise versions with one module, that's the challenge. They will just pick something that "works on both", thus ignoring the existing RabbitMQ/AMPQ/MysqlMQ possibilities that are Enterprise-only. This deteriorates the value of using extensions for Enterprise customers.

If this practice starts to become commonplace, it is possible to see that Magento as an ecosystem would mature, as integrations are a *big* part of Magento projects. Plus, in such future/ideal standardised world, it could mean that Community projects are more easily migrated into Enterprise once the extra Enterprise features are needed, given that the integration would likely be able to be kept the same.

Other examples come from within Magento, specially from it's scheduled tasks (or cronjobs). A lot of these tasks are really, in my view, more suited to be events/messages to be queued and processed asynchronously. Some examples are:

* Email sending, currently done as a cron task.
* Product indexing, currently done as a cron task.
* Cache invalidation for a given product, currently done as an event.


## Current status

I brought a much less polished version of the above arguments to the Magento Software Architect Eugene Tulika[^12] - a very nice guy - via Slack. He kindly took this on-board within the Magento product team, and a decision was made to allow a move of the AMPQ implementation using MySQL as a storage into the community version[^13]. The interfaces and behaviour would be the same as if RabbitMQ was being used. That was a "good enough" decision, that solved a large amount of the above points. When migrating into the Enterprise version, one would simply point the event stream form MySQL to RabbitMQ via configuration, after draining all remaining events from MySQL.

So, this post went all over the place a bit, but I believe it comes to show how Magento is an continuously evolving platform, how welcoming the Magento Community Engineering group is, and how easy it is to engage with them. It's not by accident that Magento was the most contributed to PHP repository from last year[^14].

This initiative also gained its own channel in the Magento Slack[^15][^16], the #message-queue-to-ce channel. It's currently up for grabs.


[^1]: https://alankent.me/2014/12/09/magento-2-caching-overview/
[^2]: http://devdocs.magento.com/guides/v2.2/extension-dev-guide/cache/page-caching.html
[^3]: https://alankent.me/2016/01/15/microservices-and-magento-quick-note/
[^4]: http://devdocs.magento.com/guides/v2.2/config-guide/multi-master/multi-master.html
[^5]: https://reactphp.org/ is one of the possible frameworks that could be used for assynchronous PHP.
[^6]: https://www.rabbitmq.com/
[^7]: http://devdocs.magento.com/guides/v2.2/extension-dev-guide/message-queues/message-queues.html
[^8]: https://www.rabbitmq.com/reliability.html
[^9]: https://alankent.me/2016/04/06/magento-shared-services/
[^10]: https://github.com/mailchimp/mc-magento2/blob/develop/Setup/InstallSchema.php#L46
[^11]: https://github.com/dotmailer/dotmailer-magento2-extension/blob/76e52974c093257e9766a6b985c354f7d2bad9a7/Model/Sync/Order.php#L158
[^12]: https://twitter.com/vrann
[^13]: https://github.com/magento-partners/magento2ee/issues/40
[^14]: http://marcelpociot.de/blog/2017-12-21-a-php-year-in-review
[^15]: https://magentocommeng.slack.com/
[^16]: https://community.magento.com/t5/Magento-DevBlog/Community-Engineering-Update-August-2017/ba-p/74220
