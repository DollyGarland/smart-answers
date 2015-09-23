# Smart Answers

## Introduction

> Smart answers are a great tool for content designers to present complex information in a quick and simple way. Defining what they are – decision trees? calculators? tools? is immaterial – what they do is provide a reusable technical framework to build a quick and simple answer to a complex question.

Read more in [a blog post](https://gds.blog.gov.uk/2012/02/16/smart-answers-are-smart/).

Have a look at
[`test/unit/flow_test.rb`](test/unit/flow_test.rb) for example usage.

This application supports two styles of writing and executing smart answers:

### Ruby smart answer flows

For more information, please go to the [Ruby SmartAnswer README](doc/smart-answer-flows.md)

### Smartdown-based smart answer flows

For more information, please go to the [Smartdown SmartAnswer README](doc/smartdown-flows.md)

## Developing

### Installing and running

NB: this assumes you are running on the GOV.UK virtual machine, not your host.

```bash
  ./install # git fetch from each dependency dir and bundle install
```

Run using bowler on VM from cd /var/govuk/development:
```
bowl smartanswers
```

### Viewing a Smart Answer

To view a smart answer locally if running using bowler http://smartanswers.dev.gov.uk/register-a-birth

### Debugging current state

If you have a URL of a Smart answer and want to debug the state of it i.e. to see PhraseList keys, saved inputs, the outcome name, append `debug=1` query parameter to the URL in development mode. This will render debug information on the Smart answer page.

### Viewing a Smart Answer as Govspeak

Seeing [Govspeak](https://github.com/alphagov/govspeak) markup of Smart Answer pages can be useful to content designers when preparing content change requests or developers inspecting generated Govspeak that later gets translated to HTML. This feature can be enabled by setting `EXPOSE_GOVSPEAK` to a non-empty value. It can be accessed by appending `.txt` to URLs (currently govspeak is available for landing and outcome pages, but not question pages).

### Visualising a flow

To see an interactive visualisation of a smart answer flow, append `/visualise` to the root of a smartanswer URL e.g. `http://smartanswers.dev.gov.uk/<my-flow>/visualise/`

To see a static visualisation of a smart answer flow, using Graphviz:

    # Download graphviz representation
    $ curl https://www.gov.uk/marriage-abroad/visualise.gv --silent > /tmp/marriage-abroad.gv

    # Use Graphviz to generate a PNG
    $ dot /tmp/marriage-abroad.gv -Tpng > /tmp/marriage-abroad.png

    # Open the PNG
    $ open /tmp/marriage-abroad.png

__NOTE.__ This assumes you already have Graphviz installed. You can install it using Homebrew on a Mac (`brew install graphviz`).

## Testing

Run all tests by executing the following:

    bundle exec rake

### Fixtures

If you need to update the world locations fixture, run the following command:

    $ rails r script/update-world-locations.rb

If you need to add/update a worldwide organisations fixture, run the following command:

    $ rails r script/update-worldwide-location-organisations.rb <location-slug>

## Making bigger changes

When making bigger changes that need to be tested or fact-checked before they are deployed to GOV.UK it is best to deploy the branch with changes to Heroku.

If you open a PR to review those changes, make sure to mention if it's being fact-checked and should not be merged to master until that's done.

### Deploying to Heroku

The 'startup_heroku.sh' shell script will create and configure an app on Heroku, push the __current branch__ and open the marriage-abroad Smart Answer in the browser.

Once deployed you'll need to use the standard `git push` mechanism to deploy your changes.

    ./startup_heroku.sh

## Merging a pull request from the Content Team

### Introduction

Members of the Content Team do not have permission to contribute directly to the canonical repository, so when they want to make a change, they create a pull request using a fork of the repository. Also since they don't usually have a Ruby environment setup on their local machine, they will not be able to update
files relating to the regression tests e.g. file checksums, Govspeak artefacts, etc. See documentation about [adding regression tests](#adding-regression-tests-to-smart-answers) for more information.

## Instructions

1. Check out the branch from the forked repo onto your local machine. Note that `<github-username>` refers to the owner

        $ git remote add <owner-of-forked-repo> git@github.com:<owner-of-forked-repo>/smart-answers.git
        $ git fetch <owner-of-forked-repo>
        $ git co -b <branch-on-local-repo> <owner-of-forked-repo>/<branch-on-forked-repo>

2. Review the changes in the commit(s)
3. Remove any trailing whitespace
4. Run the following command to re-generate the Govspeak artefacts (in `test/artefacts/<smart-answer-flow-name>`) for the regression tests:

        $ RUN_REGRESSION_TESTS=<smart-answer-flow-name> ruby test/regression/smart_answers_regression_test.rb

5. Review the changes to the Govspeak artefacts to check they are as expected
6. Run the following command to update the checksums for the smart answer:

        $ rails r script/generate-checksums-for-smart-answer.rb <smart-answer-flow-name>

7. Run the main test suite

        $ rake

8. Stage the changed files & add a new commit or amend the commit

        $ git add .
        $ git commit # ok to amend commit if only one commit in PR

9. Run the regression test for the smart answer (now that Govspeak artefacts & file checksums have been updated)

        $ RUN_REGRESSION_TESTS=<smart-answer-flow-name> ruby test/regression/smart_answers_regression_test.rb

10. Push the branch to GitHub and submit a new pull request so that people have a chance to review the changes and a Continuous Integration build is triggered. Close the original pull request.

        $ git push origin <branch-on-local-repo>

## Archiving a Smart Answer

- [How to archive a Smart Answer](doc/archiving.md)

## Content IDs

Smart answers need content-ids. You can generate one by running:

    $ be rails r "puts SecureRandom.uuid"

## Issues/todos

Please see the [github issues](https://github.com/alphagov/smart-answers/issues) page.

## Rubocop

We're using the govuk-lint Gem to include Rubocop and the GOV.UK styleguide rules in the project.

### Jenkins

We run Rubocop as part of the test suite executed on Jenkins (see jenkins.sh). Rubocop only tests for violations in files introduced in the branch being tested. This should prevent us from introducing new violations without us having to first fix all existing violations.

### Running locally

Testing for violations in the entire codebase:

    $ govuk-lint-ruby

Testing for violations in code committed locally that's not present in origin/master (useful to check code committed in a local branch):

    $ govuk-lint-ruby --diff

Testing for violations in code staged and committed locally that's not present in origin/master:

    $ govuk-lint-ruby --diff --cached

NOTE. This is mostly useful for Jenkins as we first merge, but don't commit, changes from master before running the test suite.
