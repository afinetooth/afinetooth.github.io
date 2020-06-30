---
layout: post
title:  "Add test coverage to your CI/CD pipeline"
date:   2020-06-30 09:55:52 -0700
categories: CI CD Test Coverage
---

One of the key indicators of a healthy codebase is test coverage. Once you've bought into the value of continuous integration, and continuous deployment, it just makes sense to add a test coverage service to track changes to your project's test coverage over time. Not only can it ensure your tests increase at the same rate as your code, it can also help you control your development workflow by providing pass/fail checks and PR comments showing where coverage lacks and how to improve it.

In this tutorial we're going to put the simplest possible codebase employing tests and test coverage into a CI pipeline at Circle CI, then make sure Circle CI is sending the project's test coverage results to Coveralls, a popular test coverage service used by some of the world's largest open source projects.

# Prerequisites

To follow along with this post, you'll need the following:

- A basic understanding of Ruby
- Some familiarity with testing, and test coverage
- A GitHub account
- A CircleCI account
