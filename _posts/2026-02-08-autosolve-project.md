---
layout: post
title: "gha-debug: Stop Waiting for CI to Tell You About Typos"
date: 2026-02-08
---

If you've worked with GitHub Actions for more than a week, you know the pain. You write a workflow file, push it, wait for CI to run, then discover a typo in line 23. Fix it, push again, wait again. Rinse and repeat until you've cluttered your commit history with "fix workflow syntax" messages.

The problem isn't just the wasted time—it's the feedback loop. GitHub Actions logs are buried in a web interface that's hard to navigate. Re-running failed jobs takes minutes. And there's no simple way to test locally without spinning up heavy Docker containers that bring their own compatibility issues.

## Enter gha-debug

I built **gha-debug** to solve this exact problem. It's a lightweight CLI tool that lets you test GitHub Actions workflows locally with step-by-step execution visibility.

Here's how it works:

```bash
pip install gha-debug
gha-debug run .github/workflows/test.yml
```

You get immediate feedback with colorized output showing each step, timing information, and clear error messages:

```
Running workflow: test.yml
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Job: test
  ✓ Checkout code (1.2s)
  ✓ Setup Python 3.11 (0.8s)
  ✓ Install dependencies (4.5s)
  ✓ Run tests (12.3s)

✓ Workflow completed successfully in 18.8s
```

## What Makes It Different

Unlike tools that try to perfectly replicate GitHub's environment (and fail due to compatibility issues), gha-debug focuses on what developers actually need: fast validation and clear error messages.

It parses your workflow YAML, validates syntax, and simulates the GitHub Actions environment locally. No Docker required. No complicated setup. Just install and run.

## Key Features

**Syntax Validation**: Catch common errors before you push.

```bash
gha-debug validate .github/workflows/*.yml
```

This alone saves countless "fix typo" commits.

**Environment Debugging**: See exactly what environment variables and contexts are available to your workflow.

```bash
gha-debug env .github/workflows/deploy.yml --job deploy
```

**Workflow Exploration**: List all workflows, jobs, and steps in your repository.

```bash
gha-debug list
```

**Filtered Execution**: Run a specific job from a workflow with verbose output.

```bash
gha-debug run .github/workflows/build.yml --job build --verbose
```

## The Workflow That Actually Works

Instead of the old workflow:
1. Write workflow
2. Push to GitHub
3. Wait for CI
4. Check logs in web UI
5. Find error
6. Fix locally
7. Repeat

You get this:
1. Write workflow
2. Run `gha-debug validate` and `gha-debug run` locally
3. See errors immediately
4. Fix them
5. Push once when everything works

The feedback loop goes from minutes to seconds.

## Why It Matters

Developer time is expensive. Every minute spent waiting for CI is a minute not spent building features or fixing bugs. More importantly, slow feedback loops kill momentum. When you have to wait five minutes to see if your syntax fix worked, you context-switch. You check email, browse HN, lose your train of thought.

gha-debug keeps you in flow state by giving instant feedback.

## Try It Yourself

The tool is open source and MIT licensed. Install it in one line:

```bash
pip install gha-debug
```

Then try it on your own workflows:

```bash
gha-debug run .github/workflows/test.yml
```

If you find it useful, star the repository on GitHub and contribute improvements. The goal is to make GitHub Actions debugging as painless as possible.

**GitHub**: [intellirim/gha-debug](https://github.com/intellirim/gha-debug)

Stop wasting time waiting for CI. Test locally, see clear errors, ship faster.