# HGL runtime

The native Rust execution engine for graphs compiled from HGL. This project
owns graph instantiation, endpoint storage, evaluation and the Rust provider
for core's native declarations.

The [language specification](https://github.com/hhenson/hgl_spec) defines
HGL behaviour. [Core](https://github.com/hhenson/hgl_core) owns the runtime
contract and the shared conformance cases.

## First design review

| Draft | What to review |
| --- | --- |
| [Graph and evaluation](docs/design/graph.md) | What the compiler hands over, who owns a run, and how a diamond settles |
| [Endpoint storage](docs/design/endpoints.md) | What a producer owns, what an input sees, and how a child expires |
| [Port sequence](docs/design/port.md) | The first vertical slice, its HGL witnesses, and the costs to measure |

These are implementation proposals. The repository is at the design stage.
The proposed first Rust library has three parts. The planner finishes typed
graphs, the evaluator drives instances, and the core provider binds HGL's
native operations to endpoint storage and run services.
