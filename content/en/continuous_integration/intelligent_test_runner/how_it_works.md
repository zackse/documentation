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

Intelligent Test Runner is Datadog's test impact analysis solution. Test impact analysis is a technique that has been gaining popularity over the past few decades. However, it's typically hard and time-consuming to implement. Intelligent Test Runner simplifies this complexity with an easy setup process.

Test impact analysis maps each test to the set of code files in your repository that your test uses. Its goal is to skip tests that are not affected by the code changes that are being introduced, in order to reduce the time taken by the testing phase of your CI.

An extreme example is a Pull Request that only changes a typo on a README file. For that PR, running all tests doesn't provide much value. On the contrary, flaky tests might make your CI fail, forcing you to retry the pipeline, potentially multiple times, before merging. This is a waste of both developer and CI time. With Intelligent Test Runner, a PR changing a README file would skip all tests.

## What sets it apart

Some test selection solutions try to make up for the lack of code coverage information by trying to infer in a probabilistic fashion which tests are relevant through the use of Machine Learning. This results in a test selection mechanism that can skip tests that were relevant, and cause build failures in your default branch.

Other test impact analysis solutions are based off code coverage but are hard to set up. Additionally, they only look at the diff in your latest commit to evaluate which tests to run. This means that you need to guarantee that all your commits run through CI. Even if you do, this doesn't play well with GitHub's Pull Requests, which only take into account the CI status of the latest commit to allow merging the PR.

Intelligent Test Runner leverages per-test code coverage information along with data from [Test Visibility](../../tests/) to search previous test in all relevant past commits. Configuration of Intelligent Test Runner is a one-click operation on most languages, and the results are accurate and more precise than other methods.


## How test selection works

When you enable Intelligent Test Runner, per-test (or per-suite, depending on the framework) code coverage is transparently collected and sent to Datadog.

When starting a new test session in your CI, the Datadog library requests a list of skippable tests from the backend. This request includes information about the current commit, the service that is being tested, and OS and runtime configuration. The Datadog backend uses that information to search for previous test runs that can be used as proof that, if a specific test is run, we wouldn't be gaining any value. This happens whenever a previous passed test run is found that was run on a commit that has the same version of the covered and [tracked](../#tracked-files) files.

{{< img src="continuous_integration/itr_test_selection.png" alt="Intelligent Test Runner Test Selection Diagram" style="width:60%;">}}

The Datadog library then removes tests marked as unskippable in source from the skippable tests list. It then proceeds to run the tests, but directs the test framework to skip those that remain in the skippable test list.

{{< img src="continuous_integration/itr_skipped_test_run.png" alt="Intelligent Test Runner Skipped Test" style="width:80%;">}}

Let's take a look at a specific example:

{{< img src="continuous_integration/itr_example.png" alt="Intelligent Test Runner Example Diagram" style="width:80%;">}}

In the previous diagram shows a developer branch that branches out from `main` and has several commits. On each commit, the CI has been running two tests (A and B) with different results.

- **Commit 1** ran both tests. This commit contained changes that affected the tracked files, and the covered files of both A and B.
- **Commit 2** ran both tests again:
  - Test A has to be run because, although this commit did not affect test A (no changes in tracked files or covered files), there are no previous test runs that passed for Test A. Since Intelligent Test Runner cannot guarantee that the test will pass if were run, it doesn't skip it. This time, the test passes, which indicates this is a flaky test.
  - Test B was run both because there is no previous successful test run for this test, and also because commit 2 changes files that affect it.
- **Commit 3** runs all tests because a tracked file was changed.
- **Commit 4** runs all tests too:
  - Test A is run because there is no previous test run that meets all criteria: test runs from commits 1 and 3 cannot be used because they failed, and the test run from commit 2 cannot be used because tracked files have been changed since commit 2 until commit 4.
  - Test B is also run because there is no previous test run that meets all criteria: test runs from commit 1 and 2 cannot be used because they failed, and the test run from commit 3 cannot be used because covered files for test B were modified between commits 3 and 4.
- **Commit 5** was able to skip one test:
  - Test A could be skipped thanks to the test run in commit 4. This test run meets all of the necessary criteria: tracked files haven't been changed between commit 4 and 5, neither have the impacted files for test A, and test A passed in commit 4. Therefore, if the test was run it would exercise the same code paths as in commit 4, and it would not provide any new information in the CI. In this case, skipping Test A has two benefits: there's the performance/cost benefit of not running the test as well as the increased reliability of the CI since Test A is flaky.
  - Test B had to run because its covered files changed in commit 5.
- **Commit 6** was able to skip both tests:
  - Test A could be skipped thanks to the test run in commit 4.
  - Test B could be skipped thanks to the test run in commit 5.
