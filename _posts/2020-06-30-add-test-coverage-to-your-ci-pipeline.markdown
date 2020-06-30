---
layout: post
title:  "Add test coverage to your CI/CD pipeline"
date:   2020-06-30 09:55:52 -0700
categories: CI CD Test Coverage
---

One of the key indicators of a healthy codebase is test coverage. Once you've bought into the value of continuous integration, and continuous deployment, it just makes sense to add a test coverage service to track changes to your project's test coverage over time. Not only can it ensure your tests increase at the same rate as your code increases, it can also help you control your development workflow with pass/fail checks and PR comments showing whereand how coverage lacks.

# Prerequisites

To follow along with this post, you'll need the following:

- A basic understanding of Ruby
- Some familiarity with testing, and test coverage
- A GitHub account
- A CircleCI account
