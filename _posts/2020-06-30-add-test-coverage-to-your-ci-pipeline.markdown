---
layout: post
title:  "Add Test Coverage to your CI/CD Pipeline"
date:   2020-06-30 09:55:52 -0700
categories: CI CD Test Coverage
---

<a name="title"></a>
One of the key indicators of a healthy codebase is test coverage. Once you've bought into the value of CI/CD, it just makes sense to add a test coverage service to track changes to your project's test coverage over time. Not only can it ensure tests increase at the same rate as code, it can also help you control your development workflow with pass/fail checks and PR comments showing where coverage lacks and how to improve it.

In this tutorial we're going to put a simple codebase with test coverage into a CI pipeline at CircleCI, then configure CircleCI to send our project's test coverage results to Coveralls, a popular test coverage service used by some of the world's largest open source projects.

We're going to do this by employing CircleCI's Orb technology, which makes it fast and easy to integrate with third-party tools like Coveralls.

<p>&nbsp;</p>

<a name="prerequisites"></a>
## Prerequisites

To follow along with this post, you'll need the following:

- Enough familiarity with Ruby to read some basic code and tests
- A GitHub account
- A CircleCI account

Note: *We'll create a free Coveralls account along the way.*

<p>&nbsp;</p>

<a name="basic_concepts"></a>
## Basic Concepts

<a name="basic_concepts-test_coverage"></a>
# Test *coverage*, not tests

If you're new to test *coverage*, here's how it works:

For a project made up of code and tests, a test *coverage* library can be added to assess how well the project's code is being covered by its tests. (*In the case of our Ruby project, we'll use a library called Simplecov, which comes packaged as a Rubygem.*)

On each run of your project's test suite, the test coverage *library* generates a test coverage *report*, like so:

![test coverage]({{ site.url }}/assets/test_coverage.png)

Since this test coverage *report* changes each time we add code to our project, the *report* is what we send to a service like Coveralls, which compares it to previous reports and tracks how our test coverage changes over time.

<p>&nbsp;</p>

<a name="basic_concepts-how_it_works_in_ci"></a>
# How it works in CI/CD

![test coverage in CI/CD]({{ site.url }}/assets/test_coverage_in_ci.png)

1. You push changes to your code at your SCM (ie. GitHub).
2. Your CI service builds your project, runs your tests, and generates your test coverage report.
3. Your CI posts the test coverage report to Coveralls.
4. Coveralls publishes your coverage changes to a shared workspace.
5. (Optionally) Coveralls sends comments and pass/fail checks to your PRs to control your development workflow.

<p>&nbsp;</p>

<a name="simple_app"></a>
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

<p>&nbsp;</p>

<a name="simple_app-run_tests"></a>
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

In our case, 4/5 lines are covered, translating to 80% coverage.

<p>&nbsp;</p>

<a name="simple_app-add_tests"></a>
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

<p>&nbsp;</p>

<a name="setup_ci"></a>
## Set up the CI pipeline

Now that we understand how test coverage works in this project, we'll soon be able to verify the same results through Coveralls.

But first we'll need to set up the CI pipeline.

<p>&nbsp;</p>

<a name="setup_ci-add_to_circleci"></a>
# Add the project to CircleCI

*If you want to follow along, now's a good time to fork the project from this repo and clone it down to your local machine. Once you've done that, you can follow these steps with your own copy*

*Note: From here on we'll assume you're starting with a fresh project with no changes to the original. In other words, with test coverage starting at 80%.*

To add a new public repo to [CircleCI](http://circleci.com/), [Log in](https://circleci.com/vcs-authorize/) with your GitHub login:

![circleci-login.png]({{ site.url }}/assets/circleci-login.png)

If you belong to multiple GitHub Organizations, select the one that applies to your project:

![circleci-choose-org.png]({{ site.url }}/assets/circleci-choose-org.png)

Then you'll see the list of GitHub projects for your organization:

![circleci-org-projects.png]({{ site.url }}/assets/circleci-org-projects.png)

Click __Set Up Project__ next to your newly forked project:

![circleci-setup-project-coveralls-demo-ruby.png]({{ site.url }}/assets/circleci-setup-project-coveralls-demo-ruby.png)

Then you'll see the New Project Set Up page for your project:

![circleci-project-ready-prompt.png]({{ site.url }}/assets/circleci-project-ready-prompt.png)

Here you have the choice to let CircleCI walk you through setting up your project, or add your own config file manually.

We're going to add our config file manually in order to get a closer look, so Click __Add Manually__:

![circleci-start-project-options.png]({{ site.url }}/assets/circleci-start-project-options.png)

You'll receive a prompt asking if you've already added a `./circle/config.yml` file to your repo:

![circleci-start-project-add-config-manually.png]({{ site.url }}/assets/circleci-start-project-add-config-manually.png)

We haven't, so let's go do that now.

<p>&nbsp;</p>

<a name="setup_ci-add_config_yml"></a>
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
  build_and_test:
    jobs:
      - build
```

<p>&nbsp;</p>
---

<a name="setup_ci-what_do_settings_mean"></a>
#### What do those config settings mean?

It's worth pointing out that we're using v2.1 of CircleCI's configuration spec for pipelines, the latest version, and this is indicated at the top of our file:

```ruby
version: 2.1
```

Two of the [core concepts](https://circleci.com/docs/2.0/concepts/#section=getting-started) of the v2.1 config spec are Orbs and Workflows.

[Orbs](https://circleci.com/docs/2.0/using-orbs/) are reusable packages of configuration that can be used across projects for convenience and standardization. Here we're leveraging CircleCI's newly provisioned [Ruby Orb](https://circleci.com/orbs/registry/orb/circleci/ruby), which makes quick work of setting up a new Ruby project:

```ruby
orbs:
  ruby: circleci/ruby@1.0
```

[Workflows](https://circleci.com/docs/2.0/workflows/) have been around since v2.0 and are simply a means of collecting and orchestrating jobs. Here we've defined a simple workflow called `build_and_test`:

```ruby
workflows:
  build_and_test:
    jobs:
      - build
```

Which invokes a job we've defined, called `build`, that checks out our code, installs our dependencies and runs our tests in the CI environment&mdash;a docker image running Ruby 2.6.5 and Node:

```ruby
jobs:
  build:
    docker:
      - image: cimg/ruby:2.6.5-node
    steps:
      - checkout
      - ruby/install-deps
      - ruby/rspec-test
```

[Jobs](https://circleci.com/docs/2.0/jobs-steps/#section=getting-started), of course, are the main building blocks of your pipeline, which are comprised of [Steps](https://circleci.com/docs/2.0/concepts/#steps) and the commands that do the work of your pipeline.

Note that in the final step of our job, we're using a built-in command for running rspec tests that comes with CircleCI's new [Ruby Orb](https://circleci.com/orbs/registry/orb/circleci/ruby), called [`rspec-test`](https://circleci.com/orbs/registry/orb/circleci/ruby#commands-rspec-test):

```ruby
steps:
   [...]
   - ruby/rspec-test
```

Not only does this provide a one-liner for running our Rspec tests, it also gives us some freebies, including: automated parallelization; and a default test results directory.

#### Why automated parallelization?
It allows us to run tests from our test suite [in parallel](https://circleci.com/docs/2.0/parallelism-faster-jobs/), without any additional configuration, which improves speed and is particularly handy when we're running a lot of tests.

#### Why a default test results directory?
As a convenience, this gives us a single place to store our test results in our CI environment, already merged from any parallel runs.

---
<p>&nbsp;</p>

Save the file, commit it and push:

```
git add .
git commit -m "Add .circleci/config.yml."
git push -u origin master
```

And guess what?

That's it! CircleCI is building your project in its remote CI environment.

<p>&nbsp;</p>

<a name="setup_ci-confirm_first_build"></a>
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

So we're checking our first build, and&mdash;*Whoops!* That doesn't look right&mdash;

Our first build has failed:

![circleci-first-build-failed.png]({{ site.url }}/assets/circleci-first-build-failed.png)

*Why?*

Note the error message:

```
bundler: failed to load command: rspec [...]
LoadError: cannot load such file -- rspec_junit_formatter
```

The CircleCI Ruby Orb seems to be looking for `rspec_junit_formatter`, which, upon reviewing the [orb docs](https://circleci.com/orbs/registry/orb/circleci/ruby#commands-rspec-test), makes complete sense:

![circleci-ruby-orb-rspec-test-command-docs.png]({{ site.url }}/assets/circleci-ruby-orb-rspec-test-command-docs.png)

The notes on the `rspec-test` command read:

```
You have to add `gem `spec_junit_formatter`` to your Gemfile.
```

<mark>And:</mark>

```
Enable parallelism on CircleCI for faster testing.
```

<mark>Great, let's do those things. We want those benefits (*and we want it to work!*):</mark>

First, let's install the `rspec_junit_formatter` gem in our `Gemfile`:

```ruby
# Gemfile
[...]
gem 'rspec_junit_formatter'
```

And run `bundle install`:

```
bundle install
```

<mark>Then, let's add a new key and value to our `.circleci/config.yml`:</mark>

```ruby
#./circleci/config.yml

[...]

jobs:
  build:
    docker:
      - image: cimg/ruby:2.6.5-node
    steps:
      - checkout
      - ruby/install-deps
      - ruby/rspec-test
    parallelism: 4

[...]
```

<mark>To "enable parallelism,"" we simply add the `parallelism:` key to our job called `build` and pass it a value higher than one (1). Four (4) is a common value used in CircleCI docs, so we'll start there.</mark>

<mark>Now let's push *those* changes.</mark>

```
git add .
git commit -m "Add 'rspec_junit_formatter'. Enable parallelism."
git push
```

<p>&nbsp;</p>

Let's check our build again... and&mdash;*Great!*

A successful build:

![circleci-first-build-success.png]({{ site.url }}/assets/circleci-first-build-success.png)

Notice those test results, which look much like the one we got from [running on our local machine](#run-tests):

```ruby
bundle exec rspec $TESTFILES --profile 10 --format RspecJunitFormatter --out /tmp/test-results/rspec/results.xml --format progress
[...]

  ClassOne covered returns 'covered'
    0.00042 seconds ./spec/class_one_spec.rb:7

Finished in 0.0019 seconds (files took 0.12922 seconds to load)
1 example, 0 failures

Coverage report generated for RSpec to /home/circleci/project/coverage. 4 / 5 LOC (80.0%) covered.
```

The only difference being that, before that output, the Ruby Orb's `rspec-test` command is calling `bundle exec rspec` with a number of additional parameters, telling RSpec how to format its results and where to store them:

```ruby
bundle exec rspec $TESTFILES --profile 10 --format RspecJunitFormatter --out /tmp/test-results/rspec/results.xml --format progress
```

<mark>And check out that nifty parallelism report:</mark>

![circleci-first-build-success-parallelism.png]({{ site.url }}/assets/circleci-first-build-success-parallelism.png)

<mark>Four (4) out of four (4) parallel runs, with tests split by timing, each with its own results.</mark><sup>*</sup>

<sup>*</sup> *<mark>That's probably overkill for a one file project! And I'm curious how our one test got split across four (4) jobs. We could certainly change `parallelism:` back to one (1), but for now it's not hurting anything so we'll leave it that way.</mark>*

<mark>Finally:</mark>

```ruby
Coverage report generated for RSpec to /home/circleci/project/coverage. 4 / 5 LOC (80.0%) covered.
```

Simplecov is generating a coverage report and storing it in the `/coverage` directory.

We now have test coverage in CI.

<p>&nbsp;</p>

<a name="setup_coveralls"></a>
## Configure the project to use Coveralls

Now, let's tell CircleCI to start sending those test coverage results to Coveralls.

We're in luck here, since Coveralls has published a [Coveralls Orb](https://circleci.com/orbs/registry/orb/coveralls/coveralls) following the [CircleCI Orb standard](https://circleci.com/docs/2.0/orb-intro/), which makes this plug-and-play.

But before we can set this up, we'll need to create a new account at [Coveralls](https://coveralls.io/), which is free for individual developers with public (open source) repos.

<p>&nbsp;</p>

<a name="setup_coveralls-add_project"></a>
# Add the project to Coveralls

To add your repo to [Coveralls](https://coveralls.io/sign-in), go to [http://coveralls.io/sign-in](http://coveralls.io/sign-in) and __Sign In__ with GitHub:

![coveralls-sign-in.png]({{ site.url }}/assets/coveralls-sign-in.png)

Upon first sign-in, you won't have any active repos, so go to __Add Repos__ and find a list of your public repos:

![coveralls-add-repo.png]({{ site.url }}/assets/coveralls-add-repo.png)

To add your repo, simply click the Toggle control next to your repo name, switching it to __ON__:

![coveralls-add-repo-turn-on.png]({{ site.url }}/assets/coveralls-add-repo-turn-on.png)

Great! Coveralls is now tracking your repo.

<p>&nbsp;</p>

<a name="setup_coveralls-finish_setup"></a>
# Finish setup

Prior to the release of the Coveralls Orb, the default approach to setting up a Ruby project to use Coveralls would be to install the Coveralls rubygem, which leverages Simplecov as its main dependency and takes care of uploading Simplecov's results to Coveralls.

However, to stick with the v2.1 config at CircleCI, and to leverage the benefits of CircleCI's new Ruby Orb, we'll set up the Coveralls Orb to work with the Ruby Orb.

Now the first consideration, which is a little counter-intuitive, is that the Coveralls Orb is written in Javascript, rather than Ruby, with a dependency of Node; but this is no matter since, if you recall, we configured the Ruby Orb to install a docker image containing both Ruby *and* Node:

```ruby
jobs:
  build:
    docker:
      - image: cimg/ruby:2.6.5-node
    steps:
      [...]
```

So no changes to make there.

However, another requirement of the Coveralls Orb is that it expects test coverage reports in LCOV format. So to meet that requirement, we'll make a few more changes to our project.

First, we'll add the `simplecov-lcov` gem to our `Gemfile`:

```ruby
# Gemfile
[...]
gem 'rspec_junit_formatter'
gem 'simplecov-lcov'
```

Second, we'll change some Simplecov-related configuration in our `spec_helper`:

```ruby
# spec_helper.rb
require 'simplecov'
require 'simplecov-lcov'

SimpleCov::Formatter::LcovFormatter.config.report_with_single_file = true
SimpleCov.formatter = SimpleCov::Formatter::LcovFormatter
SimpleCov.start do
  add_filter "/spec/"
end
```
Here we require `simplecov-lcov` and tell Simplecov to do two things:

<mark>Combine multiple report files into a single file:</mark>

```ruby
SimpleCov::Formatter::LcovFormatter.config.report_with_single_file = true
```

And export results in LCOV format:

```ruby
SimpleCov.formatter = SimpleCov::Formatter::LcovFormatter
```

<mark>Next, we'll tell Simplecov to save its report file to the default test results directory provided by the CircleCI Ruby Orb, `/tmp/test-results/rspec`:</mark>

```
Do stuff
```

<p>&nbsp;</p>

As our final step, we'll add the Coveralls Orb to the `orbs` section of our `.circelci/config.yml`:

```ruby
# .circleci/config.yml
version: 2.1

orbs:
  ruby: circleci/ruby@1.0
  coveralls: coveralls/coveralls@1.0.4

[...]
```

And in the `jobs` section, we'll add a step to our `build` job to call the Coveralls Orb's `upload` command:

```ruby
# /circleci/config.yml
[...]
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

[...]
```

<mark>Then we'll add the `path_to_lcov` parameter provided by the Coveralls Orb, to tell the `coveralls/upload` command where to find the coverage report it should upload to Coveralls:</mark>

```ruby
# /circleci/config.yml
[...]
      - ruby/rspec-test
      - coveralls/upload:
          path_to_lcov: ./coverage/lcov/project.lcov

[...]
```

Now push those changes:

```
git add .
git commit -m "Finish coveralls setup."
git push
```

<p>&nbsp;</p>

<a name="verify_test_coverage"></a>
## Verify test coverage via Coveralls

Since we understand [how test coverage works in this project](#simple_app), let's verify those same results through the Coveralls service.

Given that we've configured our project to use CircleCI and Coveralls, and pushed those changes to our repo, then that last push triggered a new build at CircleCI:

<mark>[IMAGE] New build at CircleCI</mark>

Which in turn pushed test results to the Coveralls API:

```
CI output showing results pushed to Coveralls API
```

And triggered a new build at Coveralls:

![coveralls-first-build-80-percent.png]({{ site.url }}/assets/coveralls-first-build-80-percent.png)

Which shows coverage at 80%.

Which is what we expected.

Now, let's validate that Coveralls is tracking *changes* in test coverage for our project.

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

<mark>[IMAGE] *New build at CircleCI*</mark>

Which in turn triggers a new build at Coveralls:

![coveralls-new-build-100-percent.png]({{ site.url }}/assets/coveralls-new-build-100-percent.png)

Which now reads 100%:

![coveralls-new-build-100-percent-zoomed.png]({{ site.url }}/assets/coveralls-new-build-100-percent-zoomed.png)

Bam! Automated test coverage updates—from Coveralls.

<p>&nbsp;</p>

<a name="next_steps"></a>
# Next steps

Now that your project's set up to track test coverage in CI, some of the next things you might want to do include:

1. __Get badged__ - Add a nifty badge to your repo's README.
2. __Configure PR comments__ - Inform collaborators of changes to test coverage to consider before merging.
2. __Set up pass/fail checks__ - Block merging unless coverage thresholds you define are met.

Start with the Coveralls docs here.
