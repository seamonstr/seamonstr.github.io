--
title: Build a deploy to S3
published: false
--

# 
### Building a react app for every push to `master`

Okay, CI... CD comes later.

I've done a bit of messing about with CircleCI previously, and that was okay, but this time out I'm going to go with GitHub actions.

#### Github actions - intro
GitHub Actions allows you to associate workflows to events in GitHub for your repo. For example, whenever there's a push to your repo you can trigger a workflow to kick off.

You configure your workflows in yaml files in the `.github` directory of your repo. A full description of workflows, jobs, steps and so on is [here](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions), but put simply a `job` is a set of commands that execute one after the other in the same execution context (ie. shell sesssion).  Anything you create or set in one step of a job will be available to subsequent steps.

A workflow contains multiple jobs, and when a workflow is triggered it will run the jobs in parallel (although you can configure them to have dependencies).

Jobs are run on VMs called "runners", and each runner will run only one job at a time. The VMs are hosted by GitHub, although you can host your own runners and GitHub will call those for you via a webhook.

#### Setting up my first workflow



### Hosting a React SPA (single page application)

Now that I know enough React to understand roughly what it's doing and how one creates and builds an app, I need to figure out how I'm going to host this app, and how I'm going to get it automatically built and deployed.  

#### First path not taken: an integrated full-stack service

Naively, I would have thought that a standard way of doing this is something like an S3 bucket mounted to a URL, or perhaps a little Nginx container. Later on, of course, I'd have to integrate an authentication service, hook that up to some authorisation thing and provide login screens, secure areas, TLS certificates, etc.

These days you get services like Heroku and Amazon Amplify that do all of this stuff in a full-stack, highly integrated sort of way. They have deep knowledge of React and its build processes, and so can just pull code from your git repo and deploy it automatically - I assume they support automated testing and intermediate environments!

Amplify provides deep integration to the rest of the AWS services, including incognito, so you can manage all of your users and their permissions through that service. It also provides a bunch of React libraries that do I know not what - I assume some basic components along with some specialised stuff to do the authentication and integrate with AWS services.

I investigated Amplify reasonably deeply and was intending to go down that path, but actually... the point of this first project is to learn a bunch of the stuff that Amplify would do for me automatically (roles and deployments, authentication, encrytion and so on). So, I'm going to stick to idea number one and bolt some stuff together myself. 

#### Second path, taken!

Another short google revealed [this article](https://medium.com/@joecrobak/production-deploy-of-a-single-page-app-using-s3-and-cloudfront-d4aa2d170aa3) which lightly covers off all the bits I would have expected: hosting the SPA, having an HTTPS port for it (with certs, etc), CDN, redirects for both the `www` and root version of the site.

