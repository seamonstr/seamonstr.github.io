---
title: "Terraform: first steps"
published: false
---

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
* The terraform that names the bucket needs to have the bucket name parameterised so that we can have one named "dev", one named "prod", etc.
* I'd like to have some automatic URLs that can be mapped to the name of the created bucket.
* The security settings will need to differ based on the environment:
  * All buckets need to be available as web sites.
  * The private buckets need to be configured for read access only by users with the permissions to see those environments.
  * The prod bucket needs to be configured for both public read access (ie. it needs a configuration that allows everybody to read the contents)

Next step would be to have a blue/green style deployment rather than dev/UAT/prod - I'll get to that later.  It'll be a more sophisticated version of the dev/uat/prod style (ie. an environment will be deployed as a private, test one; there'll need to be a switchover mechanism that enables it for broader access and flips access to it.)

## Side path: IAM Users, groups, roles

The first thing I started thinking about here was security. I'm going to need differing levels of security on this:
* An admin role and user that can create new users and assign them to groups on the project; shouldn't be able to assign users to other groups.
* A deployment role that can be assigned to both the CI/CD system and to devs to enable creation and deployment of new environments.  Assigning to devs would only be done temporarily for working on the CI/CD system; in general, deployment needs to be automatic.
* A testing role that gives access to the non-public environments currently running.

### Creating the admin role

I accessed the (AWS IAM console)[https://console.aws.amazon.com/iamv2/home] and started creating a new group for the admin user above. One of the policies you can attach to a group is `IAMFullAccess`, which allows that group to do anything with all users; it's a start, but too permissive. 

The policy I want to enact is that a user who is a member of the group "todo-admin" has the following rights:
* User creation
* Full user management rights for members of all groups beginning "todo-", except for members of "todo-admin" (preventing account lockout!)
* Rights to assign any user to any group beginning "todo-"

(This page)[https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_iam_users-manage-group.html] shows how to create more fine-grained user policies. This is what I have so far:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowTodoAdminToCreateUsers",
            "Effect": "Allow",
            "Action": "iam:CreateUser",
            "Resource": "*"
        },
        {
            "Sid": "AllowTodoAdminToAssignToTodoGroups",
            "Effect": "Allow",
            "Action": "iam:AddUserToGroup",
            "Condition": {
                "StringLike":{
                    "aws:groupname": "todo-*"
                }
            }
        },
        {
            "Sid": "DisallowTodoAdminFromManagingTodoAdmins",
            "Effect": "Allow",
            "Action": "iam:AddUserToGroup",
            "Condition": {
                "StringLike":{
                    "aws:groupname": "todo-*"
                }
            }
        },
    ]
}
```
I was using this as an example to get inspiration from:
```language: json
{
    "Version": "2021-11-21",
    "Statement": [
        {
            "Sid": "AllowTodoAdminToCreateUsers",
            "Effect": "Allow",
            "Action": "iam:CreateUser",
            "Resource": "*"
        },
        {
            "Sid": "AllowAllUsersToViewAndManageThisGroup",
            "Effect": "Allow",
            "Action": "iam:*Group*",
            "Resource": "arn:aws:iam::*:group/AllUsers"
        },
        {
            "Sid": "LimitGroupAssignmentToAdminUsers",
            "Effect": "Deny",
            "Action": [
                "iam:AddUserToGroup",
                "iam:CreateGroup",
                "iam:RemoveUserFromGroup",
                "iam:DeleteGroup",
                "iam:AttachGroupPolicy",
                "iam:UpdateGroup",
                "iam:DetachGroupPolicy",
                "iam:DeleteGroupPolicy",
                "iam:PutGroupPolicy"
            ],
            "Resource": "arn:aws:iam::*:group/AllUsers",
            "Condition": {
                "StringNotEquals": {
                    "aws:username": [
                        "srodriguez",
                        "mjackson",
                        "adesai"
                    ]
                }
            }
        }
    ]
}
```
