---
layout: post
title: My experience with microservices
---

I had the opportunity some last summer to speak in one lecture of the Cloud Apps Master (URJC). The subject being discussed was microservices architecture, and some guests from the industry could expose their knowledge. Therefore, in my case, I spoke about my learnings while working in the Ericsson product called CCDM (Cloud Core Data-Storage Manager). Basically, if one performs a quick search in google "5g core cloud native", the results are that many network vendors are implementing 5g core using cloud native technologies. This may produce, although not in all cases of course, products based on microservices architecture, trying to leverage the scalability and other benefits of microservices to address the 5G requirements. The same product for 4G networks was a monolith, so we could see also here the migration.

At the end, what we have is roughly fifty different microservices. The main ones are the Apache Geode services, because the product is a database for subscriber data, and after that the other just build the whole functionality around: 4G/5G interfaces, alarms, notifications, configuration management, logging, queues... Thus, based on my experience working on this product for more than three years, this is what I could tell about working with microservices.

## Team organization
We have ten development teams approximately, owning some microservices, libraries or sidecars. At the end, each team behaves as a tiny company, since they have the ownership of the software that they maintain. Deployment, troubleshooting, testing, security, development, quality... all these areas are responsibility of the team. Hence the developer need to adoptMy  what is called a T-shaped approach, where they have strong skills in development but also need to address the other areas. Also I would point that, of course, communication is important, to fill the gaps with the other teams and produce great documentation such that the "clients" (the other teams basically) can use your APIs or your libraries correctly

## Libraries
To avoid duplicated code between some microservices some libraries were developed. This caused many issues in the past, when managing the dependencies of each microservice. Now it is automatized and has become something smoothly, but we were struggling with this at some point. Since we are producing releases, we found that different versions of the same library was used in one release product. For instance, imagine that library "la" is updated to support a new functionality, and that microservice "ma" includes this version, but microservice "mb" does not (it does not need the last functionality, and is not in a hurry for doing it). If we had to fix a bug of that release, it was a mess because of the different versions, and the fix was needed to be provided in different branches and so on. We were also considering transforming the library into another microservice.

## Languages
The main language for the microservices is Java, leveraging Spring framework for building API Rest. But also c++, python and go are used for other microservices and containers. As developer, you have to know some languages and learn fast. Of course there are some agreements and is better that the teams embrace these languages before introducing a new one, otherwise the complexity will increase. Probably in a monolith the number of languages is smaller.

## Testing
Regarding testing, since we are deploying the microservices in Kubernetes you need to test also this layer. We are developing templates for the Helm charts and also policies (tests) for this templates and also for the values used in the microservices. Also including now tests to the rendering of the templates for the different kubernetes objects. Even in my team we are developing some "subsystem test", which means that we have isolated the deployment of our microservices, used some stubs for the services that are the dependencies, and be able to run our applications without the need of installing the whole product. Of course, this besides the usual unit tests, component tests, integration tests...

## Upgrade
Finally, for upgrades, I did not work too much on this but I know the experience from other colleagues, and it is also challenging right now. We could expect to upgrade the system, update the different microservices, but what we have is that the upgrade is complex, needs and order, caution with backward compatibility and with the data that is already in the database. Probably the upgrade of a monolith seems simple indeed.


## Conclusion
I would say that most of the notes written here are negatives, because at the end you spend more time fixing issues than realizing the benefits. And I would say that the benefits is performance and scalability in our case. Could we have the same performance with a monolith? Well, this could be a good exercise to perform. But what is sure is that everything is a trade off.
