# Stop Flaky Tests: Freeze Time in Laravel

![Stop Flaky Tests: Freeze Time in Laravel](assets/poster.jpg)

Flaky tests that pass locally but fail on CI are often caused by time-dependent assertions.

This article shows a real-world Laravel example where `Date::now()` is called twice, resulting in mismatched timestamps and random test failures. The fix is simple: **freeze time during the test** using `Date::setTestNow()` or `$this->freezeTime()`.

## What you'll learn

- Why time-based assertions cause flaky tests
- How CI environments amplify timing issues
- How to properly freeze time in Laravel tests
- When to assert exact timestamps vs. non-null values

## 📎 Read Full

[Stop Flaky Tests: Freeze Time in Laravel](https://dev.to/tegos/stop-flaky-tests-freeze-time-in-laravel-testing-1cnj)