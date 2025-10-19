# WebAssembly JavaScript API - Compatibility Matrix

## Implementation Status

Last Updated: 2025-10-19

### Namespace & Validation
| API | Status | Notes |
|-----|--------|-------|
| WebAssembly.validate(bytes) | ✅ Implemented | Full validation via WAMR |
| WebAssembly.compile(bytes) | ✅ Implemented | Async compilation via Promise |
| WebAssembly.instantiate(bytes, imports) | ✅ Implemented | Returns Promise<{module, instance}> |
| WebAssembly.instantiate(module, imports) | ✅ Implemented | Returns Promise<Instance> |
| WebAssembly.compileStreaming(source) | ⚠️ Limited | Fallback: buffers full response, no incremental compilation |
| WebAssembly.instantiateStreaming(source, imports) | ⚠️ Limited | Fallback: buffers full response, no incremental compilation |

### WebAssembly.Module
| API | Status | Notes |
|-----|--------|-------|
| new WebAssembly.Module(bytes) | ✅ Implemented | Full support |
| Module.exports(module) | ✅ Implemented | Returns export descriptors |
| Module.imports(module) | ✅ Implemented | Returns import descriptors |
| Module.customSections(module, name) | ✅ Implemented | Direct binary parsing |

### WebAssembly.Instance
| API | Status | Notes |
|-----|--------|-------|
| new WebAssembly.Instance(module, imports) | ✅ Implemented | i32 function imports only |
| instance.exports | ✅ Implemented | Function exports (i32 only) |

### WebAssembly.Memory
| API | Status | Notes |
|-----|--------|-------|
| new WebAssembly.Memory(descriptor) | ⚠️ Limited | Works but WAMR C API limitations |
| memory.buffer | ⚠️ Limited | Returns ArrayBuffer (size may be 0) |
| memory.grow(delta) | ⚠️ Limited | WAMR blocks host-side growth |

### WebAssembly.Table
| API | Status | Notes |
|-----|--------|-------|
| new WebAssembly.Table(descriptor) | ⚠️ Limited | Works but WAMR C API limitations |
| table.length | ⚠️ Limited | May return 0 due to WAMR limitation |
| table.get(index) | ⚠️ Limited | WAMR limitation |
| table.set(index, value) | ⚠️ Limited | WAMR limitation |
| table.grow(delta, value) | ⚠️ Limited | WAMR blocks host-side growth |

### WebAssembly.Global
| API | Status | Notes |
|-----|--------|-------|
| new WebAssembly.Global(descriptor, value) | 🔴 Blocked | WAMR C API limitation - returns garbage values |
| global.value (getter) | 🔴 Blocked | Returns uninitialized memory (WAMR limitation) |
| global.value (setter) | 🔴 Blocked | Cannot set values (WAMR limitation) |
| global.valueOf() | 🔴 Blocked | Returns garbage values (WAMR limitation) |

### Error Types
| API | Status | Notes |
|-----|--------|-------|
| WebAssembly.CompileError | ✅ Implemented | Proper Error subclass |
| WebAssembly.LinkError | ✅ Implemented | Proper Error subclass |
| WebAssembly.RuntimeError | ✅ Implemented | Proper Error subclass |

## Known Limitations

### Type Support
- **Function imports/exports:** i32 only (f32/f64/i64/BigInt planned for Phase 3.2B)
- **Multi-value returns:** Not yet supported
- **Multiple module namespaces:** Only "env" supported for imports

### WAMR API Blockers
- **Memory API:** WAMR v2.4.1 C API does not support standalone Memory objects
  - Created Memory objects have no accessible data region
  - `wasm_memory_data_size()` returns 0 for host-created memories
  - `wasm_memory_grow()` explicitly blocked for host calls
  - Resolution: Memory objects work when exported from WASM instances

- **Table API:** WAMR v2.4.1 C API does not support standalone Table objects
  - Created Table objects are non-functional
  - `wasm_table_size()` returns 0 for host-created tables
  - `wasm_table_grow()` explicitly blocked for host calls
  - Resolution: Table objects work when exported from WASM instances

- **Global API:** WAMR v2.4.1 C API does not support standalone Global objects (discovered 2025-10-19)
  - Created Global objects are non-functional - **return garbage values**
  - `wasm_global_get()` returns uninitialized memory
  - Initial values passed to `wasm_global_new()` are not accessible
  - `wasm_global_set()` cannot reliably store values
  - **Impact:** Tasks 4.6-4.8 reopened as BLOCKED, WPT Global tests 0% pass rate
  - See: `docs/plan/webassembly-plan/wasm-phase4-global-blocker.md`
  - Resolution: Global objects may work when exported from WASM instances (not yet tested)

### Validation Limitations
- **WAMR validation:** WAMR is more permissive than spec requires
  - Some invalid modules may be accepted
  - Cannot be fixed without WAMR changes or custom validation layer

### WPT Test Status
- **Tests run:** 8 test files (wasm/jsapi category)
- **Pass rate:** 0% (expected - test infrastructure issues + implementation gaps)
- **Main blockers:**
  - WasmModuleBuilder helper not loading properly
  - Memory/Table/Global APIs blocked by WAMR limitations
- **Unit tests:** 100% pass rate (212/212 tests)
- **Note:** Global WPT tests not added due to 0% functionality (would add 3 more failing tests)

## Environment

### WAMR Configuration
- Version: 2.4.1
- Mode: Interpreter (AOT disabled)
- Allocator: System allocator
- Bulk Memory: Enabled
- Reference Types: Enabled
- SIMD: Disabled
- GC: Disabled
- Threads: Disabled

### jsrt Version
- Version: 0.1.0
- Engine: QuickJS 2025-04-26
- Platform: Linux/macOS/Windows

## Implementation Phases

### Completed Phases
- ✅ **Phase 1:** Infrastructure & Error Types (100%)
- ✅ **Phase 2:** Core Module API - Partial (52% - Memory blocked)
- ✅ **Phase 3:** Instance & Exports - Partial (42% - i32 support only)
- ⏸️ **Phase 4:** Table & Global - Blocked by WAMR limitations

### In Progress
- 🔵 **Phase 7:** WPT Integration & Testing (11% complete)

### Planned
- **Phase 3.2B:** Full type support (f32/f64/i64/BigInt)
- **Phase 4:** Table & Global objects (requires WAMR resolution)
- **Phase 5:** Async compilation API
- **Phase 6:** Streaming API (optional)
- **Phase 8:** Documentation & Polish

## Practical Usage

### Working Examples

```javascript
// ✅ Module loading and validation
const bytes = new Uint8Array([0x00, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00]);
const isValid = WebAssembly.validate(bytes);  // true
const module = new WebAssembly.Module(bytes);

// ✅ Module introspection
const exports = WebAssembly.Module.exports(module);  // []
const imports = WebAssembly.Module.imports(module);  // []

// ✅ Instance creation without imports
const instance = new WebAssembly.Instance(module);

// ✅ Function imports (i32 only)
const moduleWithImport = new WebAssembly.Module(wasmBytesWithImport);
const instance = new WebAssembly.Instance(moduleWithImport, {
  env: {
    log: (x) => console.log(x)  // i32 parameter
  }
});

// ✅ Function exports (i32 only)
const result = instance.exports.add(1, 2);  // Works for i32

// ✅ Error types
try {
  new WebAssembly.Module(invalidBytes);
} catch (e) {
  console.log(e instanceof WebAssembly.CompileError);  // true
}
```

### Blocked/Limited Examples

```javascript
// ⚠️ Memory - Limited functionality
const memory = new WebAssembly.Memory({ initial: 1 });
console.log(memory.buffer.byteLength);  // May be 0!
// memory.grow(1);  // Throws: "not supported by host"

// ⚠️ Table - Limited functionality
const table = new WebAssembly.Table({ element: 'funcref', initial: 1 });
console.log(table.length);  // May be 0!
// table.grow(1);  // Throws: "not supported by host"

// ❌ Global - Not implemented
// const global = new WebAssembly.Global({ value: 'i32' }, 42);  // Error

// ❌ Async APIs - Not implemented
// await WebAssembly.compile(bytes);  // Error
// await WebAssembly.instantiate(bytes);  // Error
```

## References
- [MDN WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)
- [WebAssembly JS API Spec](https://webassembly.github.io/spec/js-api/)
- [WAMR Documentation](https://github.com/bytecodealliance/wasm-micro-runtime)
- [Implementation Plan](/repo/docs/plan/webassembly-plan.md)
- [Phase 4 Table Blocker](/repo/docs/plan/webassembly-plan/wasm-phase4-table-blocker.md)
