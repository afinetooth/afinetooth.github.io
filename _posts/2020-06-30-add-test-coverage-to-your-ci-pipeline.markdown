---
layout: post
title:  "Add test coverage to your CI/CD pipeline"
date:   2020-06-30 09:55:52 -0700
categories: CI CD Test Coverage
---

One of the key indicators of a healthy codebase is test coverage. Once you've bought into the value of CI/CD, it just makes sense to add a test coverage service to track changes to your project's test coverage over time. Not only can it ensure tests increase at the same rate as code, it can also help you control your development workflow with pass/fail checks and PR comments showing where coverage lacks and how to improve it.

In this tutorial we're going to put a simple codebase with test coverage into a CI pipeline at CircleCI, then configure CircleCI to send our project's test coverage results to Coveralls, a popular test coverage service used by some of the world's biggest open source projects.

We're going to do this by employing CircleCI's Orb technology, which makes it fast and easy to integrate with third-party tools like Coveralls.

## Prerequisites

To follow along with this post, you'll need the following:

- Enough familiarity with Ruby to read some basic code and tests
- A GitHub account
- A CircleCI account

Note: *We'll create a free Coveralls account along the way.*

## Basic Concepts

# Test *coverage*, not tests

If you're new to test *coverage*, here's how it works:

For a project made up of code and tests, a test *coverage* library can be added to assess how well the project's code is being covered by its tests. In the case of our Ruby project, we'll use a library called Simplecov, which comes packaged as a Rubygem.

On each run of your project's test suite, the test coverage *library* generates a test coverage *report*, like so:

![test coverage]({{ site.url }}/assets/test_coverage.png)

The test coverage *report* changes each time we add code to our project, so this *report* is what we send to a service like Coveralls, which compares it to previous reports and tracks how our test coverage changes over time.

# How it works in CI/CD

![test coverage in CI/CD]({{ site.url }}/assets/test_coverage_in_ci.png)

1. You push changes to your code at your SCM (GitHub).
2. Your CI service builds your project, runs your tests, and generates your test coverage report.
3. Your CI posts the report to Coveralls.
4. Coveralls publishes your coverage changes to a shared workspace.
5. (Optionally) Coveralls posts PR comments and pass/fail checks to control your development workflow.

## A Simple App&mdash;with Test Coverage

Here's an extremely simple Ruby project employing both tests and test coverage:

![simple project]({{ site.url }}/assets/simple_project.png)

(Find it on GitHub here.)

This is the totality of the code in this project:

```ruby
class ClassOne

  def self.covered
    "covered"
  end

  def self.uncovered
    "uncovered"
  end

end
```

And these are the tests:

```ruby
require 'spec_helper'
require 'class_one'

describe ClassOne do

  describe "covered" do
    it "returns 'covered'" do
      expect(ClassOne.covered).to eql("covered")
    end
  end

  # Uncomment below to achieve 100% coverage
  # describe "uncovered" do
  #   it "returns 'uncovered'" do
  #     expect(ClassOne.uncovered).to eql("uncovered")
  #   end
  # end
end
```

*Notice that right now, only one of the two methods in `ClassOne` is being tested.*

We've installed our test coverage library, Simplecov, as a gem in our `Gemfile`:

```ruby
source 'https://rubygems.org'

gem 'rspec'
gem 'simplecov'
```

And we've passed some configuration settings to Simplecov in our `spec/spec_helper.rb`:

```ruby
require 'simplecov'

SimpleCov.start do
  add_filter "/spec/"
end
```

## Run tests

Let's run the test suite for the first time and see what the results are:

```
bundle exec rspec
```

Our test results look like this:

```
ClassOne
  covered
    returns 'covered'

Finished in 0.0028 seconds (files took 1 second to load)
1 example, 0 failures

Coverage report generated for RSpec to /Users/jameskessler/Workspace/2020/afinetooth/coveralls-demo-ruby/coverage. 4 / 5 LOC (80.0%) covered.
```

In additional to the test results themselves, Simplecov tells us it generated a test coverage report for us in the new `/coverage` directory.

Conveniently, it generated those results in HTML format, which we can open like this:

```
open coverage/index.html
```

Our first coverage results look like this:

![coverage_80_percent_index]({{ site.url }}/assets/coverage_80_percent_index.png)

Where coverage stands at 80% for the entire project.

Clicking on `lib/class_one.rb` brings up results for the file:

![coverage_80_percent_file]({{ site.url }}/assets/coverage_80_percent_file.png)

Where you'll notice covered lines in green, and uncovered lines in red.

In our case, 4/5 lines are covered, indicating 80% coverage.
