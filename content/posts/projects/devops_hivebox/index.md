---
title: "Building the Hivebox: End-to-End DevOps"
date: 2026-01-22 # YYYY-MM-DD
lastMod: 2026-06-24
summary: "The introduction post for my Hivebox project, a DevOps project that focuses on building an API end-to-end while maintaining scalability and ease of maintenance."
draft: true
# series: ["DevOps Project: Hivebox"]
# seriesOrder: 1
categories: ["DevOps", "Projects"]
tags: ["devops", "kubernetes", "go", "docker"]
---

# Introduction

# Phase 01

Deciding to add sizing estimates and priority labels to issues
t shirt sizing: https://activecollab.com/blog/project-management/t-shirt-sizing

# Phase 02

First big troubleshooting point of the project - I accidentally cloned the repo via HTTPS instead of SSH, so I was getting popups asking to sign in when it should've been using my GH SSH key.

Starting off with a main.go:
```golang
package main

import "fmt"

func main() {
	fmt.Println("Hello world!")
}
```

From here, we need to add a function for printing the version:
```go
package main

import "fmt"

func version() string {
	return "v0.0.1"
}

func main() {
	ver := version()
	fmt.Println(ver)
}
```

And a base Dockerfile that does nothing:
```
FROM alpine
CMD ["echo", "hi"]
```

Building: `docker build -t hivebox-mvp-local .`

Docker Hub: https://hub.docker.com/_/golang

Now to get the basic functions in:
```
```

Wanted to do the full PR review experience, but I can't since I'm my own PR author and reviewer. Too lazy to go create a second account to act as a PR dummy

Forgot to update the docs! Will have to do that before I can close the PR.

Time to learn something about Github! UI caught me by surprise, accidentally created a pull request against the Hivebox HQ's repo - whoops!
Leave the fork network for things that you don't want synchro with

# Phase 03

# Phase 04

# Phase 05

# Phase 06

# Phase 07