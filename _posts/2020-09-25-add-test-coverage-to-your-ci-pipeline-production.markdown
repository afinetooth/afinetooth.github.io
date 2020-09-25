---
layout: post
title:  "Add Test Coverage to your CI/CD Pipeline"
date:   2020-09-25 09:55:52 -0700
categories: CI CD Test Coverage
---

<a name="title"></a>
One of the key indicators of a healthy codebase is good test coverage. Once you've bought into the value of CI/CD, it makes sense to use a test coverage service to track changes to your project's test coverage over time. Not only can it ensure tests increase at the same rate as code, it can also help you control your development workflow with pass/fail checks and PR comments showing where coverage lacks and how to improve it.

In this tutorial we're going to put a simple codebase with test coverage into a CI pipeline on CircleCI, then configure CircleCI to send our project's test coverage results to Coveralls, a popular test coverage service used by some of the world's largest open source projects.

We're going to do this by employing CircleCI's orb technology, which makes it fast and easy to integrate with third-party tools like Coveralls.

<p>&nbsp;</p>

<a name="prerequisites"></a>
## Prerequisites

To follow along with this post, you'll need the following:

- Enough familiarity with Ruby to read some basic code and tests
- A GitHub account
- A CircleCI account

Note: <i>We'll create a free Coveralls account along the way.</i>

<p>&nbsp;</p>

<a name="basic_concepts-test_coverage"></a>
## Test *coverage*, not tests

If you're new to test *coverage*, here's how it works:

For a project made up of code and tests, a test *coverage* library can be added to assess how well the project's code is being *covered* by its tests. (*In the case of our Ruby project, we're using a test coverage library called Simplecov.*)

On each run of your project's test suite, the test coverage *library* generates a test coverage *report*.

<p>&nbsp;</p>

<a name="basic_concepts-how_it_works_in_ci"></a>
## How it works in CI/CD

1. You push changes to your code at your SCM (ie. GitHub).
2. Your CI service builds your project, runs your tests, and generates your test coverage report.
3. Your CI posts the test coverage report to Coveralls.
4. Coveralls publishes your coverage changes to a shared workspace.
5. And if you choose to do so, Coveralls sends comments and pass/fail checks to your PRs to control your development workflow.

<p>&nbsp;</p>

<a name="simple_app"></a>
## A Simple App&mdash;with Test Coverage

Here's an extremely simple Ruby project which employs both tests and test coverage:

![simple project]({{ site.url }}/assets/simple_project.png)

(Find it on GitHub [here](https://github.com/coverallsapp/coveralls-demo-ruby).)

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

Note: <i>that right now, only one of the two methods in `ClassOne` is being tested.</i>

We've installed our test coverage library, Simplecov, as a gem in our `Gemfile`:

```ruby
source 'https://rubygems.org'

gem 'rspec'
gem 'simplecov'
```

And we've passed a configuration setting to Simplecov in our `spec/spec_helper.rb` telling it to ignore files in our test directory:

```ruby
require 'simplecov'

SimpleCov.start do
  add_filter "/spec/"
end
```

<p>&nbsp;</p>

<a name="simple_app-run_tests"></a>
### Running tests

Let's run the test suite for the first time and see the results:

```
bundle exec rspec
```

Results:

```
ClassOne
  covered
    returns 'covered'

Finished in 0.0028 seconds (files took 1 second to load)
1 example, 0 failures

Coverage report generated for RSpec to /Users/jameskessler/Workspace/2020/afinetooth/coveralls-demo-ruby/coverage. 4 / 5 LOC (80.0%) covered.
```

Note: <i>In additional to the test results themselves, Simplecov is telling us it generated a test coverage report for us in a new `/coverage` directory.</i>

Conveniently, it generated those results in HTML format, which we can open like this:

```
open coverage/index.html
```

Our first coverage report looks like this:

![coverage_80_percent_index]({{ site.url }}/assets/coverage_80_percent_index.png)

Where coverage stands at 80% for the entire project.

Clicking on `lib/class_one.rb` brings up results for the file:

![coverage_80_percent_file]({{ site.url }}/assets/coverage_80_percent_file.png)

Where you'll notice covered lines in green, and uncovered lines in red.

In our case, 4/5 lines are covered, translating to 80% coverage.

<p>&nbsp;</p>

<a name="simple_app-add_tests"></a>
### Adding tests to complete coverage

To "add" tests, un-comment the test of the second method in ClassOne:

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

Open the new results at `coverage/index.html`.

The new report looks like this:

![coverage_100_percent_index]({{ site.url }}/assets/coverage_100_percent_index.png)

Coverage has increased from 80% to 100% (and turned green).

And now, if we click on `lib/class_one.rb` we see:

![coverage_100_percent_file]({{ site.url }}/assets/coverage_100_percent_file.png)

Five (5) out of five (5) relevant lines are now covered, resulting in 100% coverage for the file, which means 100% total coverage for our one-file project.

<p>&nbsp;</p>

<a name="setup_ci"></a>
## Setting up the CI pipeline

Now that we understand how test coverage works in this project, we'll soon be able to verify the same results through Coveralls.

But first we'll need to set up the CI pipeline.

<p>&nbsp;</p>

<a name="setup_ci-add_to_circleci"></a>
### Adding the project to CircleCI

*If you want to follow along, now's a good time to [fork the project from this repo](https://github.com/coverallsapp/coveralls-demo-ruby) and clone it down to your local machine. Once you've done that, you can follow these steps with your own copy.*

*Note: From here on we'll assume you're starting with a fresh project with no changes to the original. In other words, with test coverage starting at 80%.*

To add a new public repo to [CircleCI](http://circleci.com/), [Log in](https://circleci.com/vcs-authorize/) with your GitHub login:

![circleci-login.png]({{ site.url }}/assets/circleci-login.png)

If you belong to multiple GitHub Organizations, select the one that applies to your project:

![circleci-choose-org.png]({{ site.url }}/assets/circleci-choose-org.png)

Then you'll see the list of GitHub projects for your organization:

![circleci-org-projects.png]({{ site.url }}/assets/circleci-org-projects.png)

Click __Set Up Project__ next to your new project:

![circleci-setup-project-coveralls-demo-ruby.png]({{ site.url }}/assets/circleci-setup-project-coveralls-demo-ruby.png)

Then you'll see the New Project Set Up page:

![circleci-project-ready-prompt.png]({{ site.url }}/assets/circleci-project-ready-prompt.png)

Here you have the choice to let CircleCI walk you through setting up your project, or add your own config file manually.

We're going to add our config file manually in order to get a closer look, so click __Add Manually__:

![circleci-start-project-options.png]({{ site.url }}/assets/circleci-start-project-options.png)

You'll receive a prompt asking if you've already added a `./circle/config.yml` file to your repo:

![circleci-start-project-add-config-manually.png]({{ site.url }}/assets/circleci-start-project-add-config-manually.png)

We haven't, so let's go do that now.

<p>&nbsp;</p>

<a name="setup_ci-add_config_yml"></a>
### Adding a configuration file to the project repo

At the base directory of your project, create a new, empty file called `.circleci/config.yml`.

```
vi .circleci/config.yml
```

Now, paste the following configuration settings into your empty `.circleci/config.yml` file:

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
### What do those config settings mean?

It's worth pointing out that we're using v2.1 of CircleCI's configuration spec for pipelines, the latest version, and this is indicated at the top of our file:

```yaml
version: 2.1
```

Two of the [core concepts](https://circleci.com/docs/2.0/concepts/#section=getting-started) of the v2.1 config spec are orbs and workflows.

[Orbs](https://circleci.com/orbs/) are reusable packages of configuration that can be used across projects for convenience and standardization. Here we're leveraging CircleCI's newly provisioned [Ruby orb](https://circleci.com/orbs/registry/orb/circleci/ruby), which makes quick work of setting up a new Ruby project.

```yaml
orbs:
  ruby: circleci/ruby@1.0
```

[Workflows](https://circleci.com/docs/2.0/workflows/) have been around since v2.0 and are simply a means of collecting and orchestrating jobs. Here we've defined a simple workflow called `build_and_test`.

```yaml
workflows:
  build_and_test:
    jobs:
      - build
```

Which invokes a job we've defined, called `build`, that checks out our code, installs our dependencies and runs our tests in the CI environment&mdash;a Docker image running Ruby 2.6.5 and Node:

```yaml
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

Note that in the final step of our job, we're using a built-in command for running RSpec tests that comes with CircleCI's new [Ruby orb](https://circleci.com/orbs/registry/orb/circleci/ruby), called [`rspec-test`](https://circleci.com/orbs/registry/orb/circleci/ruby#commands-rspec-test):

```yaml
steps:
   [...]
   - ruby/rspec-test
```

Not only does this provide a one-liner for running our RSpec tests, it also gives us some freebies, including automated parallelization and a default test results directory.

### Why automated parallelization?
It allows us to run tests from our test suite [in parallel](https://circleci.com/docs/2.0/parallelism-faster-jobs/), which improves speed and is particularly handy when running a lot of tests.

### Why a default test results directory?
As a convenience, this gives us a single place to store our test results in our CI environment, already merged from any parallel runs.

---
<p>&nbsp;</p>

Save the file, commit it, and push:

```
git add .
git commit -m "Add .circleci/config.yml."
git push -u origin master
```

And guess what?

That's it! CircleCI is building your project in its remote CI environment.

<p>&nbsp;</p>

<a name="setup_ci-confirm_first_build"></a>
## Confirming your first build

CircleCI started building your project the moment you pushed that last commit:

```
git push -u origin master
```

To prove that to yourself, just visit your project at [CircleCI](https://app.circleci.com/).

For me, that meant going here:<br />
[https://app.circleci.com/pipelines/github/coverallsapp/coveralls-demo-ruby](https://app.circleci.com/pipelines/github/coverallsapp/coveralls-demo-ruby)

Your URL will be different, but should follow this format:

```
https://app.circleci.com/pipelines/github/<your-github-username>/<your-github-repo>
```

So we're checking our first build, and&mdash;*whoops, that doesn't look right*...

Our first build has failed:

![circleci-first-build-failed.png]({{ site.url }}/assets/circleci-first-build-failed.png)

*Why?*

Note the error message:

```
bundler: failed to load command: rspec [...]
LoadError: cannot load such file -- rspec_junit_formatter
```

The CircleCI Ruby orb seems to be looking for `rspec_junit_formatter`, which, upon reviewing the [orb docs](https://circleci.com/orbs/registry/orb/circleci/ruby#commands-rspec-test), makes sense:

![circleci-ruby-orb-rspec-test-command-docs.png]({{ site.url }}/assets/circleci-ruby-orb-rspec-test-command-docs.png)

The notes on the `rspec-test` command read:

```
You have to add `gem `spec_junit_formatter`` to your Gemfile.
```

So let's do just that.

Install the `rspec_junit_formatter` gem in your `Gemfile`:

```ruby
# Gemfile
[...]
gem 'rspec_junit_formatter'
```

Run `bundle install`:

```
bundle install
```

Push the change:

```
git add .
git commit -m "Add 'rspec_junit_formatter'."
git push
```

<p>&nbsp;</p>

Then check our build again... and&mdash;*great!*

A successful build:

![circleci-first-build-success.png]({{ site.url }}/assets/circleci-first-build-success.png)

Notice those test results, which look much like those we got when [running locally](#run-tests):

```ruby
[...]

ClassOne covered returns 'covered'
  0.00042 seconds ./spec/class_one_spec.rb:7

Finished in 0.0019 seconds (files took 0.12922 seconds to load)
1 example, 0 failures

Coverage report generated for RSpec to /home/circleci/project/coverage. 4 / 5 LOC (80.0%) covered.
```

Just like in our local environment, Simplecov is generating a coverage report and storing it in the `/coverage` directory.

```ruby
Coverage report generated for RSpec to /home/circleci/project/coverage. 4 / 5 LOC (80.0%) covered.
```

We now have test coverage in CI.

<p>&nbsp;</p>

<a name="setup_coveralls"></a>
## Configuring the project to use Coveralls

Now, let's tell CircleCI to start sending those test coverage results to Coveralls.

We're in luck here, since Coveralls has published a [Coveralls orb](https://circleci.com/orbs/registry/orb/coveralls/coveralls) following the [CircleCI orb standard](https://circleci.com/docs/2.0/orb-intro/), which makes this plug-and-play.

But before we can set this up, we'll need to create a new account at [Coveralls](https://coveralls.io/), which is free for individual developers with public (open source) repos.

<p>&nbsp;</p>

<a name="setup_coveralls-add_project"></a>
### Adding the project to Coveralls

To add your repo to [Coveralls](https://coveralls.io/sign-in), go to [http://coveralls.io/sign-in](http://coveralls.io/sign-in) and __Sign In__ with GitHub:

![coveralls-sign-in.png]({{ site.url }}/assets/coveralls-sign-in.png)

Upon first sign-in, you won't have any active repos, so go to __Add Repos__ and find a list of your public repos:

![coveralls-add-repo.png]({{ site.url }}/assets/coveralls-add-repo.png)

To add your repo, simply click the Toggle control next to your repo name, switching it to __ON__:

![coveralls-add-repo-turn-on.png]({{ site.url }}/assets/coveralls-add-repo-turn-on.png)

Great! Coveralls is now tracking your repo.

<p>&nbsp;</p>

<a name="setup_coveralls-finish_setup"></a>
### Finishing the setup

Prior to the release of the Coveralls orb, the default approach to setting up a Ruby project to use Coveralls would be to install the Coveralls rubygem, which leverages Simplecov as its main dependency and takes care of uploading Simplecov's results to Coveralls.

However, to stick with the v2.1 config at CircleCI, and to leverage the benefits of CircleCI's new Ruby orb, we'll set up the Coveralls orb to work with the Ruby orb.

<p>&nbsp;</p>

#### Preparing to use the Coveralls orb

Now the first consideration, which is a little counter-intuitive, is that the Coveralls orb is written in Javascript, rather than Ruby, with a dependency of Node. This is no matter though, since, if you recall, we configured the Ruby orb to install a Docker image containing both Ruby *and* Node:

```ruby
jobs:
  build:
    docker:
      - image: cimg/ruby:2.6.5-node
    steps:
      [...]
```

However, another requirement of the Coveralls orb is that it expects test coverage reports in LCOV format. So to meet that requirement, we'll make a few more changes to our project.

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
Here we require `simplecov-lcov`, and we tell Simplecov to do two things:

First, combine multiple report files into a single file.

```ruby
SimpleCov::Formatter::LcovFormatter.config.report_with_single_file = true
```

Second, export results in LCOV format.

```ruby
SimpleCov.formatter = SimpleCov::Formatter::LcovFormatter
```

<p>&nbsp;</p>

#### Updating your `.circleci/config.yml`

Next, we'll add the Coveralls orb to the `orbs` section of our `.circelci/config.yml`:

```ruby
# .circleci/config.yml
version: 2.1

orbs:
  ruby: circleci/ruby@1.0
  coveralls: coveralls/coveralls@1.0.4

[...]
```

And in the `jobs` section, we'll add a new step to our `build` job:

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
          path_to_lcov: ./coverage/lcov/project.lcov

[...]
```

That command, `coveralls/upload`, calls the Coveralls orb's `upload` command. And below it, we'll pass the `path_to_lcov` parameter, which tells the orb where to find the coverage report it should upload to the Coveralls API.

<p>&nbsp;</p>

#### Adding a `COVERALLS_REPO_TOKEN`

Finally, if you're using a private CI service like CircleCI, the Coveralls API requires an access token to securely identify your repo. This is called your `COVERALLS_REPO_TOKEN`.

You'll encounter this if you visit the start page for your Coveralls project before you have any builds:

![coveralls-repo-token-start-page.png]({{ site.url }}/assets/coveralls-repo-token-start-page.png)

But you can also grab it at any time from your project's SETTINGS page:

![coveralls-repo-token-settings.png]({{ site.url }}/assets/coveralls-repo-token-settings.png)

<p>&nbsp;</p>

To let CircleCI POST securely to the Coveralls API on behalf of your repo, just add your `COVERALLS_REPO_TOKEN` as an environment variable in the CircleCI web interface under **Project Settings > Environment Variables** like so:

![circleci-env-var-repo-token.png]({{ site.url }}/assets/circleci-env-var-repo-token.png)

Now we're ready to send coverage results to Coveralls from CircleCI.

<p>&nbsp;</p>

So let's push all the changes we just made:

```
git add .
git commit -m "Finish coveralls setup."
git push
```

<p>&nbsp;</p>

<a name="verify_test_coverage"></a>
### Verifying test coverage via Coveralls

Since we understand [how test coverage works in this project](#simple_app), let's verify those same results through the Coveralls service.

Given we configured our project to use CircleCI and Coveralls, and pushed those changes to our repo, that last push triggered a new build at CircleCI:

![circleci-new-build-80-percent.png]({{ site.url }}/assets/circleci-new-build-80-percent.png)

Which in turn uploaded test results to the Coveralls API per the build log:

```
#!/bin/bash -eo pipefail

[...]

sudo npm install -g coveralls
if [ ! $COVERALLS_REPO_TOKEN ]; then
  export COVERALLS_REPO_TOKEN=COVERALLS_REPO_TOKEN
fi
export COVERALLS_ENDPOINT=https://coveralls.io

[...]

cat ./coverage/lcov/project.lcov | coveralls

[...]

[info] "2020-09-25T21:53:13.404Z"  'sending this to coveralls.io: ' '{"source_files":[{"name":"lib/class_one.rb","source":"class ClassOne\\n\\n  def self.covered\\n    \\"covered\\"\\n  end\\n\\n  def self.uncovered\\n    \\"uncovered\\"\\n  end\\n\\nend\\n","coverage":[1,null,1,1,null,null,1,0,null,null,null,null],"branches":[]}],"git":{"head":{"id":"c6b825b7bd7d4f7bbe4e75e530884a4b9fd9d9cd","committer_name":"James Kessler","committer_email":"afinetooth@gmail.com","message":"Configure project for CircleCI & Coveralls using the Coveralls orb.","author_name":"James Kessler","author_email":"afinetooth@gmail.com"},"branch":"circle-ci","remotes":[{"name":"origin","url":"git@github.com:coverallsapp/coveralls-demo-ruby.git"}]},"run_at":"2020-09-25T21:53:13.376Z","service_name":"circleci","service_number":"1917bc85-51f8-4646-80db-8b15cc40ad6c","service_job_number":"17","repo_token":"*********************************"}'

CircleCI received exit code 0
```

And triggered a new build at Coveralls:

![coveralls-first-build-80-percent.png]({{ site.url }}/assets/coveralls-first-build-80-percent.png)

Which shows coverage at 80%. Which is what we expected.

Now, let's validate that Coveralls is tracking *changes* in test coverage for our project. To do that, let's re-add that test that lifts coverage to 100%.

Open the test file, `/spec/class_one_spec.rb`, and uncomment the second test in the file

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

![circleci-first-build-100-percent.png]({{ site.url }}/assets/circleci-first-build-100-percent.png)

Which in turn triggers a new build at Coveralls:

![coveralls-new-build-100-percent.png]({{ site.url }}/assets/coveralls-new-build-100-percent.png)

Which now reads 100%:

![coveralls-new-build-100-percent-zoomed.png]({{ site.url }}/assets/coveralls-new-build-100-percent-zoomed.png)

Bam! Automated test coverage updates from Coveralls.

<p>&nbsp;</p>

<a name="next_steps"></a>
## Next steps

Now that your project is set up to automatically track test coverage, some things you might want to do next include:

1. __Get badged__ - Add a nifty "coverage" badge to your repo's README.
2. __Configure PR comments__ - Inform collaborators of changes to test coverage before merging.
3. __Set up pass/fail checks__ - Block merging unless coverage thresholds are met.
4. __Explore more complex scenarios__ - Leverage parallelism for larger projects.

Start with the [Coveralls docs](http://docs.coveralls.io) here.

<p>&nbsp;</p>

---

<p>&nbsp;</p>

## Conclusion

A healthy codebase is a well-tested codebase, and a healthy project is one where test coverage stays front and center throughout development.

A test coverage service, like [Coveralls](https://coveralls.io), lets you track changes to your project's test coverage over time, surface those changes for your whole team to see, and even stop merges that degrade the project's quality.

Using [CircleCI](https://circleci.com/)'s latest orb spec, this tutorial showed how easy it can be to connect your project with a test coverage service by making it part of your CI/CD pipeline, especially when the service leverages the configuration standard of your CI platform.

<p>&nbsp;</p>
