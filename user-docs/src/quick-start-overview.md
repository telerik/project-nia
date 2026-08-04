# Quick Start

NIA is a command-line agent harness for software development life cycle (SDLC) workflows. It connects your project context and development tools to an AI coding agent so you can draft issues, create plans, review code, and run other configured workflows.

There are **two quick-start tracks** to help you understand how NIA works and what to expect when you deploy it in your own projects. They serve different purposes, and neither one depends on the other — so start with whichever fits how you like to learn.

> **Run NIA in an isolated environment.** NIA runs AI coding agents with a high degree of autonomy — they can execute commands and modify files on their own. Run NIA in a Dev Container, virtual machine, or another isolated development environment to protect your local system and other projects.

## Choose your track

### Guided Demo App

A specially configured demo application with a set of guided tutorials that showcase the value of NIA through automatic, wizard-driven `nia learn` commands. Each tutorial runs a real-world example in action — from a two-minute question to a full security review. NIA validates your environment, explains what each command does, runs the real workflow against the sample code, and tracks your progress as you go.

Lessons take roughly 5–60 minutes **each**, and actual execution time varies with the workflow and your coding agent. You don't need to complete every lesson — run as many as are useful to you.

Best if you want a guided, preconfigured experience without configuring NIA for your own project.

➡️ **[Start the Demo App track](./quick-start-sample-app.md)**

### Your Own Project

Configure NIA in your own repository and work through the base commands to understand how NIA's autonomous workflows operate under the hood. This track lets you inspect each command and step on its own — `nia issue draft`, `nia issue plan`, `nia code create`, and more — before running them together as part of a larger workflow loop.

Best if you want to evaluate NIA on your own code and understand each step before running a complete workflow.

➡️ **[Start the Your Own Project track](./quick-start.md)**
