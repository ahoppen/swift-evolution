# Referencing zero-parameter functions

* Proposal: [SE-NNNN](https://github.com/apple/swift-evolution/blob/master/proposals/NNNN-name.md)
* Author(s): [Alex Hoppen](https://github.com/ahoppen), [Pyry Jahkola](https://github.com/pyrtsa)
* Status: **Draft**
* Review manager: TBD

## Introduction

Since the approval of [SE-0021](https://github.com/apple/swift-evolution/blob/master/proposals/0021-generalized-naming.md) it is possible to reference a function by its argument names using the `foo(arg:)` syntax but there is no way to reference a zero-parameter function. 
This proposal adds a new syntax `foo(_)` to reference an overloaded function with zero parameters.

This was one point in the discussion: [[Pitch] Richer function identifiers, simpler function types](http://thread.gmane.org/gmane.comp.lang.swift.evolution/15577/)

## Motivation

Consider the following example

```swift
class Bar {
  func foo() {
  }
  
  func foo(arg: Int) {
  }
}
```

You can reference `foo(arg: Int)` using `Bar.foo(arg:)` but there is currently no syntax to reference `foo()` without using disambiguation by type `Bar.foo() as () -> Void`. We believe this is a major hole in the current disambiguation syntax.

## Proposed solution

We propose that `Bar.foo(_)` references the function with no parameters just as `Bar.foo(arg:)` references the function with one argument named `arg`. 

In the context of functions declarations `_` already has the meaning of "there is nothing" (e.g. `func foo(_ arg: Int)`). Thus, we believe that `_` is the right character to mean that a function has no parameters.

## Detailed design

The `unqualified-name` grammar rule from [SE-0021](https://github.com/apple/swift-evolution/blob/master/proposals/0021-generalized-naming.md) changes to 

```
unqualified-name -> identifier
                  | identifier '(' ((identifier | '_') ':')+ ')'
                  | identifier '(_)'
```

If two overloads with zero-parameters exist with different return types, disambiguation has still to be done via `as` just like with the `foo(arg:)` syntax.

## Impact on existing code

This is a purely additive feature and has no impact on existing code.

## Possible issues
If Swift should ever support out-only parameters `Bar.foo(_)` could mean that the only out-only parameter shall be ignored. This would clash with the currently proposed syntax. However, since Swift functions may return multiple values as a tuple, we don't see this coming.

`Bar.foo(_)` may be mistaken for `Bar.foo(_:)` if there is also a one-parameter function without a label. This mistake would, however, be mostly detected by the compiler when later calling the function with an argument.

## Alternatives considered
### Alternative 1: `Bar.foo`

Let `Bar.foo` reference the function with zero parameters only. While this works around the possible issue of ignored out-only parameters described above, this has several minor drawbacks to the proposed solution (some of these drawbacks are mutually exclusive based on possible future proposals but one always applies):

* Most functions are not overloadad and using the base name only offers a shorthand way to reference these functions.
* This would block the way of allowing properties with the same name as a function with zero parameters by banning `Bar.foo` as a function reference (could be another proposal once this one is accepted).
* `Bar.foo(arg:)` hints that a function is referenced by its paranthesis. `Bar.foo` doesn't include paranthesis, which causes a subtle inconsistency.

### Alternative 2: `Bar.foo()` inside `#selector`

Let `Bar.foo()` refer to the zero-parameter function only inside `#selector` as it was proposed by Doug Gregor [here](https://github.com/apple/swift-evolution/pull/280#discussion_r61849122). This requires the proposal to disallow arbitrary expressions in `#selector` ([GitHub-Link](https://github.com/ahoppen/swift-evolution/blob/arbitrary-expressions-in-selectors/proposals/0000-arbitrary-expressions-in-selectors.md)) to be approved. Issues we see are:

* This gives the illusion that `foo` is actually called which it isn't
* It doen't solve the issue of referncing a zero-parameter function in arbitrary expressions somewhere else in code.

## Future directions

If this proposal is accepted there is no need that `Bar.foo` references a function with base name `foo` since there is a notation with paranthesis for every argument constellation. We could disallow `Bar.foo` to reference a function and allow a property named `foo` on `Bar`. `Bar.foo` would then refer to this property.