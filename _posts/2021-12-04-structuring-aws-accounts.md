---
title: "Structuring a set of Amazon accounts"
published: false
---

# Amazon accounts best practise

AWS best practise is that you use the Account as a way of partitioning permissions within Amazon. You can create various IAM users and roles, and try to use policies to seal users off from doing things with resources that they shouldn't be allowed to, but in practise the restrictions available to policies are just too limited.

The best way to do things is to use accounts to create secure boundaries between the various parts of the AWS estate. You organise the accounts into a sensible structure by using the AWS Organisations service to create a tree of Organisational Units (OUs), with accounts placed into OUs in the tree. 

Important element that I had to grok is that accounts != users. The accounts are containers for the actual AWS infrastructure, and they in turn _contain_ user logins.  Each account has its "root" user that you specifiy when you create the account, but it's just a user within that account - albeit a user with god-mode, so you guard its credential crazy carefully! In fact, by default Amazon generates a random password for it that it won't share with you, so you generally don't have root user credentials for most accounts.

There is a best practise structure for organisations and accounts that you can lift wholesale out of (Amazon's documentation)[https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.pdf#organizing-your-aws-environment] that makes a lot of sense and isn't too onerous.

The recommended top level set of OUs is:
 * **Sandbox**: contains cost limited, unrestricted accounts for people to experiment with
 * **Security**: contains the org-wide log archive, security tools, etc.
 * **Workloads**: contains all the actual biz code. There are a couple of ways of organising this: I'll be going for a `/Workloads/<App>/<environment>`, where `app` is the system, and `environment` is one of `dev`, `test`, `prod` and so on.
 

