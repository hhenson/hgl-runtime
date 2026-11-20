# Port sequence

**Proposed.** Port one path from a compiled HGL graph to an observable
result. Use the same HGL source on the Rust and hgraph targets. Inspect
the native implementation when traces differ. Core owns the expectation.

## First slice

```text
core replay → scalar producer → direct input → stateful node → core record
                  │                                               │
                  └── same output → second binding                └── run store
```

Start with a simulated run of `replay_preserves_equal_ticks`. The input
`[7, _, 7, 7, _]` must give the counter trace `[1, _, 2, 3]`. This needs
timed source resumption, one stateful node, direct bindings, and a recorder
that retains each timed delta. Compile core's HGL replay and record bodies.
They exercise the clock, list append and store as part of the first slice.
[Scalar eval][scalar] · [Core replay and record][run]

| Extend the same path | Observation |
| --- | --- |
| Fan-in and activity | Both branches settle before the join runs. Two bindings to one output can have different activity. An independent wake sees a passive input's current publication. [Graph cycles][cycles] · [Input activity][activity] |
| Nullable scalars and teardown | Published null has a timestamp. Clearing wakes an active input once. Starting the same graph twice gives fresh state. A stop failure escapes even after earlier ticks. [Transitions][transitions] · [Lifecycle][life] · [Failure cases][failure] |
| Structural values and dynamic graphs | Exercise per-child validity and retained deltas. Then update, remove and reinsert a dictionary key while another key survives. Observe child starts, stops and surviving state in the compiler's keyed-loop fixture. Check designation changes separately. [Collections][collection] · [Dictionary][dict] · [Keyed loop][keyed] · [References][refs] |
| Timers and windows | Distinguish a scheduler wake coincident with input from an alarm on that same tick. Probe window readiness before reading. Core's window cases cover count windows. A duration-window idle interval needs its own HGL case. [Wakes][wake] · [Windows][window] |

Run agreed cases with the spec and core revisions recorded. Report passes,
failures and cases not run. Keep proposed cases labelled. Differential
traces from a second target can locate a discrepancy. A missing expectation
belongs in core.

## Cost of the first layout

The rank walk visits every node on a busy cycle, even if only one is ready.
It visits none during an unscheduled interval in simulation. Start with
this predictable scan and reserve storage at instantiation. Target zero
heap allocations per steady-state scalar tick with a fixed graph and a
preallocated sink.

| Workload | Measure on Rust and native hgraph with matching graphs and hooks |
| --- | --- |
| Scalar chain and diamond | Nanoseconds per evaluated node, allocations per tick, bytes per node and direct binding |
| One active chain in a large idle graph, then all chains active | Time per cycle as graph size and active fraction vary. Use the crossover to judge a sparse ready queue. |
| Keyed churn | Creation/removal time per child, peak bytes, live slots and outstanding timers after removal |

Measure release builds. Check identical timed results first. Report graph
construction, evaluation and trace extraction separately so recording cost
is visible. These are planned measurements.

[scalar]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/tests/conformance/scalar_eval.hgl
[run]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/spec/execution.md
[cycles]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/tests/conformance/graph_cycles.hgl
[activity]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/tests/conformance/input_activity.hgl
[transitions]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/tests/conformance/scalar_transitions.hgl
[life]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/tests/conformance/run_lifecycle.hgl
[failure]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/tests/conformance/captured_failures.hgl
[collection]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/tests/conformance/collection_ticks.hgl
[dict]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/tests/conformance/dictionary_ticks.hgl
[keyed]: https://github.com/hhenson/hgl-compiler/blob/ce056c917666f2d7f5049e13672b6c6fcdff495b/crates/hgl_hgraph/tests/fixtures/demo.graph_for.hgl
[refs]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/tests/conformance/reference_ticks.hgl
[wake]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/tests/conformance/scheduled_wakes.hgl
[window]: https://github.com/hhenson/hgl_core/blob/40c3e6ff8391c4cfd2821e9fc5f2cffc3450ab68/tests/conformance/window_ticks.hgl
