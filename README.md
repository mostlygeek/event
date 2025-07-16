# About

This fork is slower than the parent but has 0% CPU usage while idle.

The benchmark results below were run on an M1 Macbook Pro w/ 32GB of RAM.

This is only maintained for [llama-swap](https://github.com/mostlygeek/llama-swap)'s requirements.

### kelindar/event

| name   | time/op | ops/s | allocs/op | vs prev | vs ref    |
| ------ | ------- | ----- | --------- | ------- | --------- |
| 1x1    | 70.1 ns | 14.3M | 0         | -       | ✅ +76%   |
| 1x10   | 60.2 ns | 16.6M | 0         | -       | ✅ +2.3x  |
| 1x100  | 36.5 ns | 27.4M | 0         | -       | ✅ +6.3x  |
| 10x1   | 80.3 ns | 12.4M | 0         | -       | ✅ +86%   |
| 10x10  | 35.7 ns | 28.0M | 0         | -       | ✅ +7.3x  |
| 10x100 | 24.7 ns | 40.5M | 0         | -       | ✅ +10.2x |

### mostlygeek/event

| name   | time/op  | ops/s | allocs/op | vs kelindar/event | vs ref    |
| ------ | -------- | ----- | --------- | ----------------- | --------- |
| 1x1    | 79.5 ns  | 12.6M | 0         | ❌ -13%           | ✅ +53%   |
| 1x10   | 66.2 ns  | 15.1M | 0         | 🟰 similar         | ✅ +2.1x  |
| 1x100  | 63.5 ns  | 15.7M | 0         | ❌ -50%           | ✅ +3.4x  |
| 10x1   | 168.7 ns | 5.9M  | 0         | ❌ -54%           | ❌ -18%   |
| 10x10  | 166.8 ns | 6.0M  | 0         | ❌ -71%           | 🟰 similar |
| 10x100 | 55.6 ns  | 18.0M | 0         | ❌ -65%           | ✅ +3.6x  |

---

---

---

<p align="center">
<img width="300" height="100" src=".github/logo.png" border="0" alt="kelindar/event">
<br>
<img src="https://img.shields.io/github/go-mod/go-version/kelindar/event" alt="Go Version">
<a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License"></a>

## Fast, In-Process Event Dispatcher

This package offers a high-performance, **in-process event dispatcher** for Go, ideal for decoupling modules and enabling asynchronous event handling. It supports both synchronous and asynchronous processing, focusing on speed and simplicity.

- **High Performance:** Processes millions of events per second, about **4x to 10x faster** than channels.
- **Generic:** Works with any type implementing the `Event` interface.
- **Asynchronous:** Each subscriber runs in its own goroutine, ensuring non-blocking event handling.

**Use When:**

- ✅ Decoupling modules within a single Go process.
- ✅ Implementing lightweight pub/sub or event-driven patterns.
- ✅ Needing high-throughput, low-latency event dispatching.
- ✅ Preferring a simple, dependency-free solution.

**Not For:**

- ❌ Inter-process/service communication (use Kafka, NATS, etc.).
- ❌ Event persistence, durability, or advanced routing/filtering.
- ❌ Cross-language/platform scenarios.
- ❌ Event replay, dead-letter queues, or deduplication.
- ❌ Heavy subscribe/unsubscribe churn or massive dynamic subscriber counts.

## Generic In-Process Pub/Sub

This repository contains a **simple, in-process event dispatcher** to be used to decouple internal modules. It provides a generic way to define events, publish and subscribe to them.

```go
// Various event types
const EventA = 0x01

// Event type for testing purposes
type myEvent struct{
    Data string
}

// Type returns the event type
func (ev myEvent) Type() uint32 {
	return EventA
}
```

## Using Default Dispatcher

For convenience, this package provides a default global dispatcher that can be used with `On()` and `Emit()` package-level functions.

```go
// Subcribe to event A, and automatically unsubscribe at the end
defer event.On(func(e Event) {
    println("(consumer)", e.Data)
})()

// Publish few events
event.Emit(newEventA("event 1"))
event.Emit(newEventA("event 2"))
event.Emit(newEventA("event 3"))
```

## Using Specific Dispatcher

When publishing events, you can create a `Dispatcher` which is then used as a target of generic `event.Publish[T]()` and `event.Subscribe[T]()` functions to publish and subscribe to various event types respectively.

```go
bus := event.NewDispatcher()

// Subcribe to event A, and automatically unsubscribe at the end
defer event.Subscribe(bus, func(e Event) {
    println("(consumer 1)", e.Data)
})()

// Subcribe to event A, and automatically unsubscribe at the end
defer event.Subscribe(bus, func(e Event) {
    println("(consumer 2)", e.Data)
})()

// Publish few events
event.Publish(bus, newEventA("event 1"))
event.Publish(bus, newEventA("event 2"))
event.Publish(bus, newEventA("event 3"))
```

It should output something along these lines, where order is not guaranteed given that both subscribers are processing messages asynchronously.

```
(consumer 2) event 1
(consumer 2) event 2
(consumer 2) event 3
(consumer 1) event 1
(consumer 1) event 2
(consumer 1) event 3
```

## Benchmarks

Please note that the benchmarks are run on a 13th Gen Intel(R) Core(TM) i7-13700K CPU, and results may vary based on the machine and environment. This one demonstrates the publishing throughput of the event dispatcher, at different number of event types and subscribers.

```
name                 time/op      ops/s        allocs/op    vs channels
-------------------- ------------ ------------ ------------ ------------------
1x1                  38.7 ns      25.9M        0             ✅ +4.2x
1x10                 13.0 ns      77.1M        0             ✅ +12x
1x100                12.2 ns      81.7M        0             ✅ +7.7x
10x1                 26.5 ns      37.7M        0             ✅ +6.3x
10x10                12.2 ns      82.3M        0             ✅ +7.8x
10x100               12.2 ns      82.0M        0             ✅ +6.6x
```

# License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
