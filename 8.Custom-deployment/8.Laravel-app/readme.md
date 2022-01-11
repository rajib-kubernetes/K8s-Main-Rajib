## In production, you will most likely require more than just one web server and database.And you probably don't have a single app, but multiple apps with different concerns such as an API, a front-end, workers to process batch jobs, etc.

# How do you deploy your apps and make sure that they can scale efficiently with your users?

In this article, you will learn how to set up a Laravel application in Kubernetes.
## Kubernetes, why and what?
```
Who has lots of application deployed in production?
Google, of course.
Kubernetes is an open-source tool that was initially born from Google to facilitate a large number of deployments across their infrastructure.
```

### It is good at three things:
```
Running any type of app (not just PHP).
Scheduling deployments across several servers.
Being programmable.
Let's have a look at how you can leverage Kubernetes to deploy a Laravel 
```
