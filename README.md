# HGL runtime

The native Rust execution engine for graphs compiled from HGL. This project
owns graph instantiation, endpoint storage, evaluation and the Rust provider
for core's native declarations.

The [language specification](https://github.com/hhenson/hgl_spec) defines
HGL behaviour. [Core](https://github.com/hhenson/hgl_core) owns the runtime
contract and the shared conformance cases.
