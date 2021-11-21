I added a comments section to my blog!

My blog is a Jekyll blog, served by GitHub pages (on github.io). This means that it's static - I add a post in markdown format, check it locally to make sure it works okay, then check it in to my github repo.  GitHub then regenerates all the pages and puts the new version of the blog site up at seamonstr.github.io.

Because it's static, I don't have any code that I can hook to implement a comments module (and neither would I want to!). I need a solution that I can just plug in.  I considered a few, and settled on TalkYard, a startuppy open source solution written buy a Swedish chap named Magnus Lindberg.

You can build and install TalkYard on our own server if you like, but Magnus offers a hosted service that I've gone for instead.

## Installation was a doddle

Apparently the hosting costs about €3 a month, which is fine. I [signed up for the service](https://www.talkyard.net/-/create-site/embedded-comments) (which just mean providing my name, blog URL, contact email address and a uname/passwd), and then it asked me what type of blog I have. I selected Jekyll, and then it gave me a batch of instructions on how to hook TalkYard in.

It means adding a bit of config to my Jekyll file (just some variables pointing to the TalkYard service endpoints), a new HTML file (with a `div` to render Talkyard) that gets included into each of my posts, and then a line to the post template to include the new HTML file.  That's it - done.

The Talkyard site then give me a nice in-app tour around itself, showing me how to moderate and review.  All very easy.

The default styling looks okay, as you can see below, and it works really well.

Next...