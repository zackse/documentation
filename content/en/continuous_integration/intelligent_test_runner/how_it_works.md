---
title: How It Works
kind: documentation
further_reading:
  - link: "https://www.datadoghq.com/blog/streamline-ci-testing-with-datadog-intelligent-test-runner/"
    tag: "Blog"
    text: "Streamline CI testing with Datadog Intelligent Test Runner"
  - link: "https://www.datadoghq.com/blog/monitor-ci-pipelines/"
    tag: "Blog"
    text: "Monitor all your CI pipelines with Datadog"
---

## What is Intelligent Test Runner?

Intelligent Test Runner is Datadog's test impact analysis solution. Test impact analysis is a technique that has been getting popular in the past couple of decades, but is usually hard and time-consuming to implement. Intelligent Test Runner cuts through that complexity with a simple setup process.

Test impact analysis maps each test to the set of code files in your repository that your test exercises, with the goal of reducing the time taken by the testing phase of your CI. This is achieved by analyzing the code changes you are trying to test, and skipping the tests that do not impact your code changes.

An extreme example is a Pull Request that only changes a typo on a README file. For that PR, running all tests doesn't provide much value. On the contrary, flaky tests might make your CI fail, forcing you to retry the pipeline, potentially multiple times, before merging. This is a waste of both developer and CI time. With Intelligent Test Runner, a PR changing a README file would skip all tests.

## What sets it apart

Some test selection solutions try to make up for the lack of code coverage information by trying to infer in a probabilistic fashion which tests are relevant through the use of Machine Learning. This results in a test selection mechanism that can skip tests that were relevant, and cause build failures in your default branch.

Other test impact analysis solutions are based off code coverage but are hard to set up. Additionally, they only look at the diff in your latest commit to evaluate which tests to run. This means that you need to guarantee that all your commits run through CI. Even if you do, this doesn't play well with GitHub's Pull Requests, which only take into account the CI status of the latest commit to allow merging the PR.

Intelligent Test Runner leverages per-test code coverage information along with data from [Test Visibility](../../tests/) to search previous test in all relevant past commits. Configuration of Intelligent Test Runner is a one-click operation on most languages, and the results are accurate and more precise than other methods.


## How test selection works

When you enable Intelligent Test Runner, per-test (or per-suite, depending on the framework) code coverage is transparently collected and sent to Datadog.

When starting a new test session in your CI, the Datadog instrumentation requests a list of skippable tests from the backend. It provides information about the current commit, the service that is being tested, and OS and runtime configuration. The Datadog backend uses that information to search for previous test runs that can be used as proof that, if a specific test is run, we wouldn't be gaining any value. This happens whenever a previous passed test run is found that was run on the same set of impacted and [tracked](../#tracked-files) files.

{{< img src="continuous_integration/itr_test_selection.png" alt="Intelligent Test Runner Test Selection Diagram" style="width:80%;">}}

The Datadog library then removes tests marked as unskippable in source from the skippable tests list. It then proceeds to run the tests, but directs the test framework to skip those that remain in the skippable test list.

{{< img src="continuous_integration/itr_skipped_test_run.png" alt="Intelligent Test Runner Test Selection Diagram" style="width:80%;">}}
