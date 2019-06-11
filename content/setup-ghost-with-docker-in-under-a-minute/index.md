---
title: "Setup Ghost with Docker in Under a Minute"
description: "After deciding to give Ghost a shot, I spent an evening trying to make it easily configurable, as simplicity is always best, amirite? Ghost is somewhat complex as you can use it in many different…"
date: "2018-11-07T01:45:11.246Z"
categories: 
  - Medium
  - Docker
  - Zeit
  - Ghost Blog
  - Blogging

published: true
canonical_link: https://medium.com/joelachance/setup-ghost-with-docker-in-under-a-minute-202199db49c1
redirect_from:
  - /setup-ghost-with-docker-in-under-a-minute-202199db49c1
---

![](./asset-1.jpeg)

### Introduction

After deciding to give [Ghost](https://ghost.org/) a shot, I spent an evening trying to make it easily configurable, as simplicity is always best, amirite? Ghost is somewhat complex as you can use it in many different ways, from a full site deployment to an API. It’s obviously easiest to use Ghost Pro, but as I didn’t want to pay $29 a month, here we are.

For our production deployment, we’re using Zeit’s [Now](https://zeit.co/now), which works amazingly well. All you need to do to deploy is run `now` in your project’s directory and it spins up your Docker image. Ghost Docker doesn’t publish a Dockerfile in their README, however, it’s very simple — check it out [here](https://github.com/fiveinfinity/ghost-docker). (In case you peak at the Dockerfile: yes, it’s that simple!)

Another mention is we’ll be using SQLite — there are other options like MySQL & Postgres, however, in the spirit of not spending money, we’re using SQLite.

### Walkthrough

A quick prerequisite for anyone using a custom domain: we need to make sure our custom domain’s DNS is pointing to Zeit. Set this up now so you don’t have to wait for the DNS to populate later. Zeit has an excellent walkthrough for all domain situations — follow along [here](https://zeit.co/docs/features/aliases).

-   Clone the [repo](https://github.com/fiveinfinity/ghost-docker).
-   In the project directory, run `now -e url=https://custom-domain.com` . This will give us a `now` deployment URL in our `stdout` that will look something like this: `[https://ghostdocker-xyzabc.now.sh](https://ghostdocker-xyzabc.now.sh)` .
-   Once our domain is pointing to Zeit, alias your latest deployment:`now alias [https://ghostdocker-xyzabc.now.sh](https://ghostdocker-xyzabc.now.sh) custom-domain.com` .
-   Last, `now` spins down any deployments that aren’t explicitly scaled. Run the following: `now scale domain-name.com all 1 10`. This tells `now` to scale our deployment in all regions with a minimum of 1 instance and a maximum of 10.

---

Visit `custom-domain.com` to see your newly created blog. To begin modifying your new site, visit `custom-domain.com/ghost` and create an account.
