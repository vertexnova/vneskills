---
name: vne-review-bindings
description: >
  Review checklist for language bindings over the C API (Swift, C#, Kotlin, JNI):
  generated code, native ownership, error mapping, JNI rules, threading, parity
  with C++. Use when reviewing bindings/**, *.swift, *.cs, *.kt, *.kts, JNI C,
  or the Swift, C#, Kotlin samples.
---

# Review: bindings

## Generated code

- Hand edits to `*.g.cs`, `vnegfx_jni.g.c`, or other generated files.
- C API changed without regenerating every binding in the same PR.

## Ownership

- A native create without exactly one destroy path: `SafeHandle` (C#), `deinit`
  (Swift), `AutoCloseable` plus `Cleaner` (Kotlin).
- GPU or context work from a finalizer, `Cleaner`, or GC thread.
- Use after the owning context was destroyed.
- Pinned or borrowed memory released before the native call returns.

## Errors

- A C result code ignored; every result maps to the language error
  (`GfxException`, Swift `throws`, Kotlin exception) with the last-error text.

## JNI

- No `ExceptionCheck` after a call that can throw.
- Long `GetPrimitiveArrayCritical` sections or allocation inside them.
- Local refs created in a loop without a ref frame; global refs never deleted.
- A `JNIEnv` cached or used on another thread.

## Threading

- Context used from a thread other than its creator (one-thread rule).
- UI work off the main thread (iOS, Android, Avalonia).

## Numbers and parsing

- Culture-sensitive parsing (C# without `InvariantCulture`, Kotlin without
  `Locale.ROOT`).
- Implicit byte order; Java signed bytes not masked with `& 0xFF`.
- C# unchecked casts of counts read from files.

## Parity with C++

- Loader caps, orientation matrices, or defaults that differ from the C++
  reference (these are duplicated in three languages today).
- API coverage table not updated for a new or changed function.
- A sample option or knob dropped in one language.

Sources: .NET native interop best practices (SafeHandle); Android JNI tips;
JNI specification; Swift C interoperability docs.
