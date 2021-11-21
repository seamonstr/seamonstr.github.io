--
title: Terraform: first steps
published:false
--

In this step I want to figure out how to host the SPA (single page application) that I created in my last post. It's just a `Hello World` React app, so it just needs some HTML, JS and CSS hosting somewhere. Of course, I want to serve it from a decent URL, and I also want it to have a TLS certificate and `https` goodness, so it needs a reasonably capable host.

I found [this](https://medium.com/@joecrobak/production-deploy-of-a-single-page-app-using-s3-and-cloudfront-d4aa2d170aa3) article that seems to have all the bits that I need (and uses S3, which I wanted), so I'm going to work through that. The deviation will be that I'm using GitHub actions instead of CircleCI for build and deployment.

## Design of my deployed app
My app's deployment is fairly straightforward:
* An S3 bucket that the single-page application (SPA) will be written to (js, html, css, any images).
  * Correct caching header setups so that the browser won't reload the large portions of the app unless they've changed
  * Some infra-as-code that will accomplish the above - this needs to be parameterised so that it can build to either `dev` or `prod`.
* At some point, a CDN (CloudFront, most like)
* A GitHub action that will build, test and deploy the app to `dev` every time it's changed
* A GitHub action that will deploy it to `prod` when I tag.

I've decided to go with CloudFormation in this V1 rather than Terraform. THe article I'm reading is talking CloudFormation, and doing Terraform would mean pausing here to learn enough to be effective... so let's do the simple thing to start; I'll move over as the next step.

## S3 Bucket
My S3 bucket needs to have the following characteristics:
* It needs to be configured for both public read access and available as a website (ie. it needs a configuration that allows everybody to read the contents)
* The terraform that names the bucket needs to have the bucket name parameterised so that we can have one named "dev", one named "prod", etc.

