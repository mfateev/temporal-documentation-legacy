---
codecontent: workerscaffoldgo02
weight: 25
categories: [tour]
---

# Add Dependencies to Go Project

Start by creating the *Gopkg.toml* file that lists the dependencies.

The top-level dependencies are all Uber open source libraries:

1. [Tally](https://github.com/uber-go/tally) records metrics
2. [Zap](https://github.com/uber-go/zap) for logging
3. [YARPC](https://github.com/yarpc/yarpc-go) is a messaging platform
4. [Temporal](https://github.com/temporalio/temporal-go-sdk) is the Temporal Go client
