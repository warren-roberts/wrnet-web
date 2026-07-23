+++
date = '2026-07-23'
draft = false
title = 'Managing this site with Codex'
summary = 'A short note on shifting my Hugo website workflow into a Codex-assisted local editing loop.'
+++

![Codex, Hugo, and Cloudflare workflow](codex-hugo-cloudflare.png)

## A new workflow

I have been moving more of the care and feeding of this site into a local workflow with Codex. The basic shape is still familiar: Hugo builds the site, GitHub stores the source, and Cloudflare deploys it. The difference is that Codex is now helping me manage the local edits, check the build, and keep the moving parts aligned.

This has significantly cut down on the hassle of dealing with upgrading, writing, and deploying. With this decreased burden I hope it will increase my velocity. Creativity is often obstructed by the snares of technology.

## What changed

The site itself is still intentionally simple. I write Markdown, keep images next to the posts that use them, and let Hugo do what Hugo does best. Codex adds a useful layer around that process: inspecting the repository, updating dependencies, catching build failures, and preparing changes before I decide what should actually ship.

## Why it matters

For a personal site, the goal is not to create a complicated publishing system. The goal is to make writing and maintenance feel low-friction enough that I keep doing it. If Codex can handle the mechanical parts while I focus on the words, that is a pretty good trade.
