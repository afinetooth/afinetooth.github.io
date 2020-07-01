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

For a project made up of code and tests, a test *coverage* library can be added to assess how well the project's code is being covered by its tests. In the case of our Ruby project, we'll use a library called Simplecov, which comes packaged as a Rubygem.

On each run of your project's test suite, the test coverage *library* generates a test coverage *report*, like so:

![test coverage]({{ site.url }}/assets/test_coverage.png)

Since this test coverage *report* changes each time we add code to our project, the *report* is what we send to a service like Coveralls, which compares it to previous reports and tracks how our test coverage changes over time.

# How it works in CI/CD

![test coverage in CI/CD]({{ site.url }}/assets/test_coverage_in_ci.png)

1. You push changes to your code at your SCM (GitHub).
2. Your CI service builds your project, runs your tests, and generates your test coverage report.
3. Your CI posts the report to Coveralls.
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

*If you want to follow along, now's a good time to fork the project from this repo and clone it down to your local machine. Once you've done that, you can follow these steps with your own copy.*

To add a new public repo to [CircleCI](http://circleci.com/), [Log in](https://circleci.com/vcs-authorize/) at https://circleci.com/vcs-authorize/ with your GitHub login*:

![circleci-login.png]({{ site.url }}/assets/circleci-login.png)

If you belong to multiple GitHub Organizations, select the one that applies to your project:

![circleci-choose-org.png]({{ site.url }}/assets/circleci-choose-org.png)

Then you'll see the list of GitHub projects for your organization:

![circleci-org-projects.png]({{ site.url }}/assets/circleci-org-projects.png)

Click Set Up Project next to your newly forked project:

![circleci-setup-project-coveralls-demo-ruby.png]({{ site.url }}/assets/circleci-setup-project-coveralls-demo-ruby.png)

Then you'll see the New Project Set Up page for `coveralls-demo-ruby`:

![circleci-project-ready-prompt.png]({{ site.url }}/assets/circleci-project-ready-prompt.png)

Here you have the choice to let CircleCI walk you through setting up your project, or add your own config file manually. We're going to add our config file manually because in this context it's actually simpler, and quicker, so...

Click __Add Manually__:

![circleci-start-project-options.png]({{ site.url }}/assets/circleci-start-project-options.png)

You'll receive a prompt asking if you've already added a `./circle/config.yml` file to your repo:

![circleci-start-project-add-config-manually.png]({{ site.url }}/assets/circleci-start-project-add-config-manually.png)

You haven't, so let's go do that now.

Just leave that window alone. We'll come back to it.

# Add a `.circle/config.yml` to the project repo

At the base directory of your project, create a new, empty file called `.circleci/config.yml`.

Now, paste the following configuration settings into your empty ``.circleci/config.yml`:

```ruby
version: 2.1

orbs:
  ruby: circleci/ruby@1.0
  coveralls: coveralls/coveralls@1.0.4

jobs:
  build:
    docker:
      - image: cimg/ruby:2.6.5-node
    steps:
      - checkout
      - ruby/install-deps
      - ruby/rspec-test
      - coveralls/upload:
          path_to_lcov: ./coverage/lcov/project.lcov

workflows:
  version: 2.1
  build_and_test:
    jobs:
      - build:
        filters:
          branches:
            only:
              - circle-ci
```

Save the file and commit it:

```
git add .
git commit -m "Add .circleci/config.yml."
```

And guess what?

That's it! CircleCI is building your project in its remote CI environment.

## Configure the project to use Coveralls

Now, let's tell CircleCI to start sending test coverage results to Coveralls.

# Add the project to Coveralls

To add your repo to [Coveralls](https://coveralls.io/sign-in), go to http://coveralls.io/sign-in and __Sign In__ with GitHub:

![coveralls-sign-in.png]({{ site.url }}/assets/coveralls-sign-in.png)

Upon first sign-in, you won't have any active repos, so go to __Add Repos__ and find a list of your public repos:

![coveralls-add-repo.png]({{ site.url }}/assets/coveralls-add-repo.png)

To add your repo, simply click the Toggle control next to your repo name, switching it to __ON__:

![coveralls-add-repo-turn-on.png]({{ site.url }}/assets/coveralls-add-repo-turn-on.png)

Great! Coveralls is now tracking your repo.

# Finish setup

[WIP] *Complete section*

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

## verify changes in test coverage via Coveralls

Since we understand how test coverage works in this project, let's verify those same results through the Coveralls service.

[WIP] *Change tests back and push first build to Coveralls.*

If you've already configured your project to use Coveralls & CircleCI, then CircleCI has already pushed your first build to Coveralls, and you've noted that coverage stands at 80%:

![coveralls-first-build-80-percent.png]({{ site.url }}/assets/coveralls-first-build-80-percent.png)

The badge on your repo reinforces that:

![coveralls-badge-80-percent.png]({{ site.url }}/assets/coveralls-badge-80-percent.png)

Now let's validate that Coveralls is tracking changes in test coverage on our project.

To do that, let's add a test that lifts coverage to 100%.

Open the test file, /spec/class_one_spec.rb, and uncomment the second test in the file, so that this:

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
