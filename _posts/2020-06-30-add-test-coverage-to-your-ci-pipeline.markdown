---
layout: post
title:  "Add Test Coverage to your CI/CD Pipeline"
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

For a project made up of code and tests, a test *coverage* library can be added to assess how well the project's code is being covered by its tests. (*In the case of our Ruby project, we'll use a library called Simplecov, which comes packaged as a Rubygem.*)

On each run of your project's test suite, the test coverage *library* generates a test coverage *report*, like so:

![test coverage]({{ site.url }}/assets/test_coverage.png)

Since this test coverage *report* changes each time we add code to our project, the *report* is what we send to a service like Coveralls, which compares it to previous reports and tracks how our test coverage changes over time.

# How it works in CI/CD

![test coverage in CI/CD]({{ site.url }}/assets/test_coverage_in_ci.png)

1. You push changes to your code at your SCM (GitHub).
2. Your CI service builds your project, runs your tests, and generates your test coverage report.
3. Your CI posts the test coverage report to Coveralls.
4. Coveralls publishes your coverage changes to a shared workspace.
5. (Optionally) Coveralls sends comments and pass/fail checks to your PRs to control your development workflow.

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

# Run tests

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

Notice that, in additional to the test results themselves, Simplecov is telling us it generated a test coverage report for us in a new `/coverage` directory.

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

# Add tests and see coverage change

To "add" tests, simply un-comment the test of the second method in ClassOne:

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
  describe "uncovered" do
    it "returns 'uncovered'" do
      expect(ClassOne.uncovered).to eql("uncovered")
    end
  end
end
```

Now run the test suite again:

```
bundle exec rspec
```

And open the new results at `coverage/index.html`.

Here's how things look now:

![coverage_100_percent_index]({{ site.url }}/assets/coverage_100_percent_index.png)

Notice coverage has increased from 80% to 100% (and turned green).

And now, if we click on `lib/class_one.rb` we see:

![coverage_100_percent_file]({{ site.url }}/assets/coverage_100_percent_file.png)

Five (5) out of five (5) relevant lines are now covered, resulting in 100% coverage for the file, which means 100% coverage for our one-file project.

## Set up the CI pipeline

Now that we understand how test coverage works in this project, we'll soon be able to verify the same results through Coveralls.

But first we'll need to set up the CI pipeline.

# Add the project to CircleCI

*If you want to follow along, now's a good time to fork the project from this repo and clone it down to your local machine. Once you've done that, you can follow these steps with your own copy*

*Note: From here forward we'll assume you're starting with a fresh project with no changes to the original. In other words, with test coverage starting at 80%.*

To add a new public repo to [CircleCI](http://circleci.com/), [Log in](https://circleci.com/vcs-authorize/) at [https://circleci.com/vcs-authorize/](https://circleci.com/vcs-authorize/) with your GitHub login:

![circleci-login.png]({{ site.url }}/assets/circleci-login.png)

If you belong to multiple GitHub Organizations, select the one that applies to your project:

![circleci-choose-org.png]({{ site.url }}/assets/circleci-choose-org.png)

Then you'll see the list of GitHub projects for your organization:

![circleci-org-projects.png]({{ site.url }}/assets/circleci-org-projects.png)

Click Set Up Project next to your newly forked project:

![circleci-setup-project-coveralls-demo-ruby.png]({{ site.url }}/assets/circleci-setup-project-coveralls-demo-ruby.png)

Then you'll see the New Project Set Up page for your project:

![circleci-project-ready-prompt.png]({{ site.url }}/assets/circleci-project-ready-prompt.png)

Here you have the choice to let CircleCI walk you through setting up your project, or add your own config file manually.

We're going to add our config file manually in order to get a closer look, so Click __Add Manually__:

![circleci-start-project-options.png]({{ site.url }}/assets/circleci-start-project-options.png)

You'll receive a prompt asking if you've already added a `./circle/config.yml` file to your repo:

![circleci-start-project-add-config-manually.png]({{ site.url }}/assets/circleci-start-project-add-config-manually.png)

We haven't, so let's go do that now.

# Add a `.circleci/config.yml` to the project repo

At the base directory of your project, create a new, empty file called `.circleci/config.yml`.

```
vi .circleci/config.yml
```

Now, paste the following configuration settings into your empty `.circleci/config.yml`:

```ruby
version: 2.1

orbs:
  ruby: circleci/ruby@1.0

jobs:
  build:
    docker:
      - image: cimg/ruby:2.6.5-node
    steps:
      - checkout
      - ruby/install-deps
      - ruby/rspec-test

workflows:
  version: 2.1
  build_and_test:
    jobs:
      - build:
```

---

## [WIP]

# WHAT DO THOSE CONFIG SETTINGS MEAN?

---

*
*
*

# [DRAFT]

1. We are using CircleCI's v2.1 configuration for pipelines
2. It includes a simple workflow
3. And CircleCI's newly provisioned [Ruby Orb](https://circleci.com/orbs/registry/orb/circleci/ruby) from the CircleCI Orb Registry
4. This makes quick work of setting up a new Ruby project
5. And it even includes a built-in command for running rspec tests, called [`rspec-test`](https://circleci.com/orbs/registry/orb/circleci/ruby#commands-rspec-test)
6. Not only does this make it simple to run our rspec test suite, but it also gives us some freebies, including:
7. Freebie: Automatic Parallelization
8. Freebie: Default Test Results Directory

#### Why Automatic Parallelization?
It allows to run our test suite, in parallel, for faster speed, which is particularly handy when we're utilizing more than one test suite, such as Cucumber, MiniTest and RSpec.

#### Why a Default Test Results Directory?
As a convenience, this makes it easy to tell our uploader script where to find our test results in our CI environment, already merged from any parallel runs.*

---

*
*
*

Save the file and commit it:

```
git add .
git commit -m "Add .circleci/config.yml."
```

And guess what?

That's it! CircleCI is building your project in its remote CI environment.

# Confirm your first build

CircleCI started building your project the moment you pushed that last commit:

```
git push -u origin master
```

To prove that to yourself, just visit your project at [CircleCI](https://app.circleci.com/).

For me, that meant going here:<br />
[https://app.circleci.com/pipelines/github/afinetooth/coveralls-demo-ruby](https://app.circleci.com/pipelines/github/afinetooth/coveralls-demo-ruby)

Your URL will be different, but should follow this format:

```
https://app.circleci.com/pipelines/github/<your-github-username>/<your-github-repo>
```

Your first build should look something like this:

![circleci-first-build-success.png]({{ site.url }}/assets/circleci-first-build-success.png)

A successful build&mdash;albeit, without much going on.

Notice those test results, which look the same as on our local machine:

```ruby
bundle exec rspec

ClassOne
  covered
    returns 'covered'

Finished in 0.00127 seconds (files took 0.11459 seconds to load)
1 example, 0 failures

Coverage report generated for RSpec to /home/circleci/project/coverage. 4 / 5 LOC (80.0%) covered.
```

That means our tests passed and, therefore, our build succeeded.

## Configure the project to use Coveralls

Now, let's tell CircleCI to start sending test coverage results to Coveralls.

We're in luck here, since Coveralls has published a [Coveralls Orb](https://circleci.com/orbs/registry/orb/coveralls/coveralls) following the [CircleCI Orb standard](https://circleci.com/docs/2.0/orb-intro/), which makes this plug-and-play.

But before we can set this up, we'll need to create a new account at [Coveralls](https://coveralls.io/), which is free for individual developers with public (open source) repos.

# Add the project to Coveralls

To add your repo to [Coveralls](https://coveralls.io/sign-in), go to [http://coveralls.io/sign-in](http://coveralls.io/sign-in) and __Sign In__ with GitHub:

![coveralls-sign-in.png]({{ site.url }}/assets/coveralls-sign-in.png)

Upon first sign-in, you won't have any active repos, so go to __Add Repos__ and find a list of your public repos:

![coveralls-add-repo.png]({{ site.url }}/assets/coveralls-add-repo.png)

To add your repo, simply click the Toggle control next to your repo name, switching it to __ON__:

![coveralls-add-repo-turn-on.png]({{ site.url }}/assets/coveralls-add-repo-turn-on.png)

Great! Coveralls is now tracking your repo.

# Finish setup

---

## [WIP]

# FINISH SETUP AT COVERALLS

---

*
*
*

__[DRAFT]__

Prior to the release of CircleCI's new Ruby Orb, the normal approach to setting up a Ruby project for Coveralls would be to install the Coveralls rubygem to our project, which takes care of uploading test results to Coveralls.

However, we're going to change our approach here in order to leverage some of the new features provided by the Ruby Orb, such as automated parallelization and the default location for storing test results.

Doing this prepares us to scale up our project later&mdash;to more tests and potentially more test *suites*&mdash;without requiring further changes.

To get these benefits, we'll make just a few changes to our project.

1. First, since the CircleCI Ruby orb expects XML-formatted test results, we'll install the `rspec_junit_formatter` gem in our Gemfile:

```ruby
gem 'simplecov-lcov'
gem 'rspec_junit_formatter'
```

And change the Simplecov configuration in our `spec_helper`:

```ruby
require 'simplecov'
require 'simplecov-lcov'

SimpleCov::Formatter::LcovFormatter.config.report_with_single_file = true
SimpleCov.formatter = SimpleCov::Formatter::LcovFormatter
SimpleCov.start do
  add_filter "/spec/"
end
```
Now we're exporting test results in XML format.

2. Next, we'll tell Simplecov to save out test results to the default test results directory provided by the CircleCI Ruby Orb, `/tmp/test-results/rspec`:

```
Do stuff
```

Next, we'll install the Coveralls Orb for CircleCI and tell it where to find our test results:

1. We'll install the Coveralls Orb by adding it to our `.circelci/config.yml`, like so:

```
Show changes to config.
```

2. Then we'll tell coveralls where to find the test results it will upload to the Coveralls API:

```
Config
```

---

*
*
*


# Get badged

Let's add a Coveralls badge to our repo for quick status on our project's test coverage.

At the bottom of your Coveralls start page:

![coveralls-first-coverage-report.png]({{ site.url }}/assets/coveralls-first-coverage-report.png)

You'll see a box like this instructing you to badge your repo:

![coveralls-badge-your-repo.png]({{ site.url }}/assets/coveralls-badge-your-repo.png)

Click the __Embed__ button and choose the version of markup that applies for you:

![coveralls-badge-your-repo-choose-embed-markup.png]({{ site.url }}/assets/coveralls-badge-your-repo-choose-embed-markup.png)

(For a GitHub README, that's the Markdown version.)

Then paste the markup into the top of your README, and...

Voilà:

![coveralls-badge-80-percent.png]({{ site.url }}/assets/coveralls-badge-80-percent.png)

Your repo is badged!

## Verify test coverage via Coveralls

Since we understand how test coverage works in this project, let's verify those same results through the Coveralls service.

If you've already configured your project to use Coveralls & CircleCI, then CircleCI has already pushed your first build to Coveralls, and you've noted that coverage stands at 80%:

![coveralls-first-build-80-percent.png]({{ site.url }}/assets/coveralls-first-build-80-percent.png)

The badge on your repo reinforces that:

![coveralls-badge-80-percent.png]({{ site.url }}/assets/coveralls-badge-80-percent.png)

Now let's validate that Coveralls is tracking changes in test coverage on our project.

To do that, let's re-add that test that lifts coverage to 100%.

Open the test file, `/spec/class_one_spec.rb`, and uncomment the second test in the file, so that this:

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

Becomes this:

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
  describe "uncovered" do
    it "returns 'uncovered'" do
      expect(ClassOne.uncovered).to eql("uncovered")
    end
  end
end
```

Now, save the file, commit the change and push it to GitHub:

```
git commit -m "Add tests to make coverage 100%."
git push
```

That push will trigger a new build at CircleCI:

[IMAGE] *New build at CircleCI*

Which in turn triggers a new build at Coveralls:

![coveralls-new-build-100-percent.png]({{ site.url }}/assets/coveralls-new-build-100-percent.png)

Which now reads 100%:

![coveralls-new-build-100-percent-zoomed.png]({{ site.url }}/assets/coveralls-new-build-100-percent-zoomed.png)

Which is reinforced by your updated badge:

![coveralls-badge-100-percent.png]({{ site.url }}/assets/coveralls-badge-100-percent.png)

Bam! Automated test coverage updates—from Coveralls.

# Next steps

Now that your project's set up to track test coverage in CI, some of the next things you might want to do include:

1. Configure PR comments
2. Set up pass/fail checks
3. Explore more complex scenarios, like multiple test suites and parallel builds

Start with the Coveralls docs here.
