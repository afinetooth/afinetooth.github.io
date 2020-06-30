---
layout: post
title:  "Add test coverage to your CI/CD pipeline"
date:   2020-06-30 09:55:52 -0700
categories: CI CD Test Coverage
---

One of the key indicators of a healthy codebase is test coverage. Once you've bought into the value of continuous integration and deployment, it just makes sense to add a test coverage service to track changes to your project's test coverage over time. Not only can it ensure tests increase at the same rate as code, it can also help you control your development workflow with pass/fail checks and PR comments showing where coverage lacks and how to improve it.

In this tutorial we're going to put the simplest possible codebase with tests and test coverage into a CI pipeline at CircleCI, then configure CircleCI to send our project's test coverage results to Coveralls, a popular test coverage service used by some of the world's biggest open source projects. 

We're going to do this by employing CircleCI's Orb technology, which makes it fast and easy to integrate with third-party tools like Coveralls. 

# Prerequisites

To follow along with this post, you'll need the following:

- Enough familiarity with Ruby to read some basic code and tests
- A basic understanding of test coverage
- A GitHub account
- A CircleCI account

* We'll create a free Coveralls account along the way.

# Some Basic Concepts

Chances are you're familiar with testing, but if test coverage is new to you, here's how it works:

For a project made up of code and tests, a test coverage library can be added, whose job it is to assess how well the project's code is being covered by its tests. Each language has its share of code coverage libaries. In the case of our simple Ruby project, we'll be using a library called Simplecov, which comes packaged as a Rubygem. On each run of your project's test suite, the test coverage library generates a test coverage report.

The test coverage report is different each time we add code to our project. It's the test coverage report that we send to a test coverage service like Coveralls, which compares our report to previous builds to track how test coverage changes over time.

# The Simplest Possible App&mdash;with Tests and Test Coverage


