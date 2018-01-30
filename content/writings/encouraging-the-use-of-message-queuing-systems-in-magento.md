---
title: "Encouraging the use of Message Queuing Systems in Magento"
description: "A needed change of heart in architectural approaches in the Magento world for integrations and events streams."
tags: ["magento", "php", software engineering", "rabbitmq", "message queuing systems", "performance engineering"]
categories: [ "Computer Science" ]
draft: false
---

## General view of Magento performance

The deployment of Magento as an ecommerce platform doesn't come without its challenges in regards to performance, scalability and optimisation. This is due to several platform characteristics:  

* **A** - A monolithic codebase, with the template block rendering approach for the frontend being synchronous, rendering one template only after the previous one finishes.
* **B** - Repetitive synchronous requests to storage systems, blocking code execution in the main process.
* **C** - Synchronous API calls to external systems, blocking code execution in the main process.

Magento feels fast when its catalog doesn't change much, or doesn't depend on external information to be rendered. In these situations, the HTML of the CMS and catalog pages (products and categories) are retained cached in Varnish after the first time they are rendered. On subsequent identical requests, Varnish responds with HTML very fast[^1][^2]. While this helps a lot in practice, several business require fast changing in-stock/out-of-stock page statuses to be modified, thus invalidating the cache aggressively.

In cases in which Varnish solves your problem, this still leaves Magento's dynamic pages (cart, my account) and controllers requests vulnerable to similar performance problems mentioned above.

For problems **A** and **B**, the ongoing effort within Magento to formalise APIs across modules using service contracts[^3] could allow it to become microservice-friendly, thus helping with parallelization of responsibilities (such as template rendering for a given module) by opening doors to asynchronous computation[^4] across modules of Magento.

The rest of this post will abandon problems **A** and **B**, and advocate for a possible solution to problem **C**.

## A possible solution

In my experience, synchronous API calls tend to occur in pages that are normally dynamic, or controllers that receive a POST request into the system, such as:

* Once a client uses the login function, an API call happen to an external system to indicate you logged in.
* Once a client subscribes to a newsletter, an API call happen to an external system to indicate you now are subscribed.
* Once a client places an order, an API call to an external system to indicate the an order was placed.

## Magento's initiative

Magento themselves faced these challenges [^5]

The idea to move RabbitMQ/AMPQ/MysqlMQ implementation/modules from Enterprise to Community.

## Arguments to move the EE feature to the CE version

If you are new to the Magento world, please note ...

The move of these features into community is to standardize and force extension developers and integration developers to behave better when developing integrations based on APIs, in a similar fashion that the Varnish move of FPC from Enterprise to Community forced extension developers to make their extensions Varnish compatible out of the box. Main issues I see currently are:
- Extensions end up building their own not so good queuing system (e.g.: integration extensions - normally in email marketing extensions, but there are other examples of not so popular extensions).
- So, as you go on installing multiple extensions that provide integration capabilities within Magento, each will create their on "synchronization" tables, and you end up with several approaches to queuing within your own application. This scenario is much harder to debug/remember/maintain/integrate.
- This also makes it very expensive for extension developers to do an integration "right". This means that they can very quickly fallback to not so good practices, such as end up doing live/direct API calls hooked on controller actions (e.g.: pushing an order event to another system during the "place order" action, whilst it could have been done asynchronously a few moments later and saved time on the request). This is actually the most common symptom: extensions that just hook expensive API calls (that could have been done asynchronously) in requests, such as "customer login", "place order", impacting frontend response time and building up requests on the server.
- This means that, on a big project, as a SI you end up spending a lot of the time of the project modifying extensions, as to "queue" their API calls yourself. This effort is then repeated over and over across implementations, and I suppose partners around the world.
- Suppose all extensions could just use the existing Magento queuing system, all you had to do as a SI is make sure the workers were running.
- Finally, the argument is that as extension developers normally target both Community and Enterprise versions, that's the challenge. They will just pick something that "works on both", thus ignoring the existing RabbitMQ/AMPQ/MysqlMQ possibilities that are Enterprise-only. This deteriorates the value of using extensions for Enterprise customers, and this is why I believe this issue is similar to the decision of bringing Varnish support to community.
- I see that Magento as an ecosystem would improve, as integrations are a *big* point of Magento projects. Plus, in such standardised world, it could mean that Community projects are more easily migrated into Enterprise once the extra Enterprise features are needed, given that the integration would likely be able to be kept the same.

## Example

## Status follow up

#### Footnotes

[^1]: https://alankent.me/2014/12/09/magento-2-caching-overview/
[^2]: http://devdocs.magento.com/guides/v2.2/extension-dev-guide/cache/page-caching.html
[^3]: https://alankent.me/2016/01/15/microservices-and-magento-quick-note/
[^4]: https://reactphp.org/ is one of the possible frameworks that could be used for assynchronous PHP.
[^5]: https://alankent.me/2016/04/06/magento-shared-services/

http://devdocs.magento.com/guides/v2.2/extension-dev-guide/message-queues/message-queues.html
https://www.balanceinternet.com.au/we-present-rabbitmq-integration-between-akeneo-and-magento-product-suite-at-magento-imagine-2016/