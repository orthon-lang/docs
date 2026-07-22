# Singleton Class (Eigenclass)

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

Every Ruby object carries a hidden, per-instance class — informally called
the "singleton class," also known as the "eigenclass" — that sits between the
object and its nominal class in the method-lookup chain. Defining a method
directly on one instance (for example `def obj.greet; "hi"; end`) inserts
that method into the object's own singleton class, leaving the object's class
and every other instance of that class untouched.

Ruby uses this exact same mechanism internally to implement per-class
("static"/class-level) methods: a class is itself an object, so a class's own
singleton class holds its class-level methods.

The singleton class is also the foundation for `method_missing`, a catch-all
hook invoked when no matching method is found anywhere in the lookup chain
(including the singleton class), enabling proxies, dynamic DSLs, and "ghost
methods." `method_missing` is covered as part of this same concept rather
than split into a separate file, since it shares the same lookup-chain
mechanism and the same open questions about non-local dispatch.

This is a distinct question from
[`OBJECTS_AND_SINGLETONS.md`](OBJECTS_AND_SINGLETONS.md). That document is
about a TYPE having exactly one shared global instance (Kotlin's `object`
keyword) — "one instance shared by everyone" — while this concept is the
opposite direction: one SPECIFIC instance receives its own private method
table, invisible to every other instance of the same class.

The core problem: should Orthon permit per-instance method or behaviour
attachment independent of the instance's declared type, and should any
fallback-dispatch hook (a `method_missing` equivalent) exist for calls that
match no declared method?

## Examples

| Language | Per-instance method definition | Mechanism name | Fallback dispatch hook | Typical use |
|---|---|---|---|---|
| **Ruby** | `def obj.greet; ...; end` or `class << obj; def greet; ...; end; end` | Singleton class / eigenclass | `method_missing` | DSLs, mocking/stubbing individual objects, implementing class methods internally |
| **Python** | Monkey-patching a bound method onto one instance via `types.MethodType(func, obj)` | No dedicated per-object metaclass concept — each object shares its class's `__dict__` | `__getattr__` for missing attributes | Dynamic proxies, mocking libraries |
| **JavaScript** | Assigning a function directly onto one object instance, natural in JS since objects use a prototype chain rather than classical per-class dispatch | "Own properties" on the prototype chain | `Proxy` with a `get` trap (modern equivalent) | Flexible object composition, proxies |
| **Smalltalk** | Every class already has its own singleton metaclass holding class methods; Ruby generalised this to every object, not just classes | Metaclass | `doesNotUnderstand:` — the original fallback hook Ruby's `method_missing` was modelled on | Historical origin of both mechanisms |
| **Java/C#/Kotlin** | Not supported directly — requires wrapper/decorator objects or delegation patterns | No dedicated mechanism name | No equivalent fallback hook; Java's reflection-based `InvocationHandler` for dynamic proxies is the closest analogue | No typical per-instance-method use case in idiomatic code |

Ruby, defining a method on one instance and sketching a `method_missing`
fallback:

```ruby
obj = Object.new
def obj.greet
  "hi"
end
obj.greet   # => "hi" — no other Object instance gains this method

class Proxy
  def method_missing(name, *args)
    puts "called #{name}"
    super
  end
end
```

JavaScript, assigning a function directly onto one object instance:

```javascript
const obj = {};
obj.greet = () => "hi";
obj.greet();   // "hi" — no other object gains this method
```

`Proxy`'s `get` trap is the closest modern fallback-dispatch analogue to
`method_missing` in JavaScript.

## Implications for Orthon

1. **Presumes an identity-bearing object model** — Per-instance method
   attachment presumes an identity-bearing object model where one instance
   can diverge behaviourally from every sibling instance of the same type;
   this conflicts with [`DATA_MODEL.md`](DATA_MODEL.md)'s "Data First"
   principle (data has no imposed semantic meaning until a Data Modifier
   transforms it) and its "Value semantics by default" principle (data is
   compared by structure, not identity) — the entire premise needs
   re-evaluation once Orthon's actual object/instance model is settled,
   since it may not apply, or may apply only to reference-representation
   values rather than value-semantics data.

2. **Composition, not a hidden metaclass** — If any per-instance
   customisation need is genuinely required, Orthon's idiomatic answer is
   composition — wrap the instance in a new value carrying the additional
   behaviour ([`COMPOSITION_OVER_INHERITANCE.md`](COMPOSITION_OVER_INHERITANCE.md))
   — rather than a hidden per-object metaclass that mutates behaviour
   without changing the value's type.

3. **`method_missing` conflicts with Explicit Semantics** — A
   `method_missing`-style fallback conflicts with Orthon's Explicit
   Semantics pillar and the Manifesto's "Semantics over syntax" principle,
   since a reader (human or LLM) cannot tell from a call site whether the
   receiver has a real declared method or an intercepted fallback — this is
   the same non-local, context-dependent behaviour
   [`REFLECTION_ALTERNATIVES.md`](REFLECTION_ALTERNATIVES.md) already
   rejects for field access ("No `getField(\"name\")` pattern... string-based
   access must always be statically verified"), applied here to method
   dispatch instead of field access.

4. **Class-method use case already covered** — The class-method use case
   (Ruby's internal reliance on the eigenclass to implement `self.method`)
   is already served in Orthon's research by
   [`OBJECTS_AND_SINGLETONS.md`](OBJECTS_AND_SINGLETONS.md)'s
   companion-object/static-member discussion, so no separate per-object
   mechanism is needed purely to explain class-level methods.

5. **Route legitimate proxy/DSL needs through compile-time metaprogramming**
   — Any legitimate proxy/DSL/mocking need that `method_missing` currently
   serves in Ruby should be routed through Orthon's existing compile-time
   metaprogramming research — [`METAOBJECTS.md`](METAOBJECTS.md)'s
   traits/annotations/macros and
   [`REFLECTION_ALTERNATIVES.md`](REFLECTION_ALTERNATIVES.md)'s
   sealed-types-plus-pattern-matching — rather than an open-ended runtime
   fallback hook.

## Open Questions

1. Does Orthon's eventual object model (once
   [`DATA_MODEL.md`](DATA_MODEL.md)'s data/behaviour separation is
   finalised) even have a notion of per-instance identity that
   per-instance method attachment could attach to, or is this concept moot
   by construction for value-semantics data?
2. If a constrained, explicit form of per-instance customisation is wanted
   (for example, test doubles/mocks), should it live in a test-only library
   feature rather than the core language?
3. Is there a legitimate need for an explicit, statically-declared
   "catch-all" trait method (rather than an implicit runtime fallback) to
   cover the DSL/proxy use cases `method_missing` serves in Ruby?
4. How would a constrained reflection API (per
   [`REFLECTION_ALTERNATIVES.md`](REFLECTION_ALTERNATIVES.md)) cover the
   legitimate mocking/proxying use cases without reintroducing a hidden
   per-object metaclass?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [OBJECTS_AND_SINGLETONS.md](OBJECTS_AND_SINGLETONS.md)
- [METAOBJECTS.md](METAOBJECTS.md)
- [REFLECTION_ALTERNATIVES.md](REFLECTION_ALTERNATIVES.md)
- [COMPOSITION_OVER_INHERITANCE.md](COMPOSITION_OVER_INHERITANCE.md)
- [DATA_MODEL.md](DATA_MODEL.md)
