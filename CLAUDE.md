# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

k3s-ansible is an Ansible playbook collection for building Kubernetes clusters using k3s.

## Repositories

| Name | URL | Branch |
|------|-----|--------|
| k3s-io (upstream) | https://github.com/k3s-io/k3s-ansible | `main` |
| jss (fork) | https://github.com/jon-stumpf/k3s-ansible | `rebase-upstream` |

## Background

The fork contains pull requests created years ago that addressed open issues and added new functionality. The original maintainer was unresponsive during that time. A new maintainer has since significantly updated the upstream codebase, overlapping some of that work, and closed the old PRs as out-of-date — but expressed interest in the changes. The goal is to rebase the fork's changes onto the current upstream `main` and resubmit as new pull requests.

## Objectives

Rebase the fork's logical changes onto upstream `main`.

## Workflow

1. Analyze all commits in the `jss` fork relative to upstream `k3s-io/main`.
2. Group commits into logical, high-level changes with representative names.
3. Propose new commits/PRs that cleanly apply those changes onto upstream `main`.
