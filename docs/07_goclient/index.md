# Go SDK

## Overview

Go SDK attempts to follow Go language conventions. The conversion of a Go program to the fault-oblivious workflow function is expected to be pretty mechanical.

Temporal requires determinism of the workflow code. It supports deterministic execution of the multithreaded code and constructs like `select` that are non-deterministic by Go design. The Temporal solution is to provide corresponding constructs in the form of interfaces that have similar capability but support deterministic execution.

For example, instead of native Go channels, workflow code must use the `workflow.Channel` interface. Instead of `select`, the `workflow.Selector` interface must be used.

For more information, see [Creating Workflows](02_create_workflows#]).

## Links

- GitHub project: [https://github.com/temporalio/temporal-go-sdk](https://github.com/temporalio/temporal-go-sdk)
- Samples: [https://github.com/temporalio/temporal-go-samples](https://github.com/temporalio/temporal-go-samples)
- GoDoc documentation: [https://godoc.org/go.temporal.io/temporal](https://godoc.org/go.temporal.io/temporal)
