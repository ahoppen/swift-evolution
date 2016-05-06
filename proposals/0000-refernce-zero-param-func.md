# Referencing zero-parameter functions

* Proposal: [SE-NNNN](https://github.com/apple/swift-evolution/blob/master/proposals/NNNN-name.md)
* Author(s): [Alex Hoppen](https://github.com/ahoppen), [Pyry Jahkola](https://github.com/pyrtsa)
* Status: **Draft**
* Review manager: TBD

## Introduction

Since the approval of [SE-0021](https://github.com/apple/swift-evolution/blob/master/proposals/0021-generalized-naming.md) it is possible to reference a function by its argument names using the `foo(arg:)` syntax but there is no way to reference a zero-parameter function. 
`foo` currently references all methods with base name `foo`. If there are multiple methods with this base name, one has to disambiguate the referenced function by its type using `as`.

This proposal changes the behaviour of `foo` to always reference a zero-parameter function. To reference a function with more parameters, the argument labels have to be explicitly named (e.g. `foo(arg:)`).

Originally, the proposal sought to introduce the new syntax `foo(_)` to reference an overloaded function with zero parameters, but the idea was discarded based on the discussion on [swift-evolution](http://thread.gmane.org/gmane.comp.lang.swift.evolution/16150).

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

You can reference `foo(arg: Int)` using `Bar.foo(arg:)` but there is currently no syntax to reference `foo()` without using disambiguation by type `Bar.foo as () -> Void`. We believe this is a major hole in the current disambiguation syntax.

## Proposed solution

We propose that `Bar.foo` only references methods with no parameters just as `Bar.foo(arg:)` references the methods with one argument named `arg`.

This is a breaking change since `Bar.foo` currently refers to *all* methods with base name `foo`.

## Detailed design

The existing syntax `Bar.foo` is reinterpreted to not reference any method on `Bar` with base name `foo` but to only reference functions named `foo` that take no parameters.

If two overloads with zero-parameters exist with different return types, disambiguation has still to be done via `as` just like with the `foo(arg:)` syntax.

## Impact on existing code

Existing code that uses `Bar.foo` to reference methods with parameters needs to be changed. We believe this will effect many developers, so a fix-it would be essential.

## Possible issues

`Bar.foo` may be mistaken for a reference to a property, since all other references to functions will contain parenthesis if this proposal is accepted.

Most functions are not overloaded and using the base name only offers a shorthand way to reference these functions. If this proposal is accepted, this shorthand is no longer available for methods with parameters.

## Alternatives considered
### Alternative 1: `Bar.foo()` inside `#selector`

Let `Bar.foo()` refer to the zero-parameter function only inside `#selector` as it was proposed by Doug Gregor [here](https://github.com/apple/swift-evolution/pull/280#discussion_r61849122). This requires the proposal to disallow arbitrary expressions in `#selector` ([GitHub-Link](https://github.com/ahoppen/swift-evolution/blob/arbitrary-expressions-in-selectors/proposals/0000-arbitrary-expressions-in-selectors.md)) to be approved. Issues we see are:

* This gives the illusion that `foo` is actually called which it isn't
* It doesn't solve the issue of referencing a zero-parameter function in arbitrary expressions somewhere else in code.

### Alternative 2: `Bar.foo(_)`

The original idea of using `Bar.foo(_)` to refer to the zero-parameter method was discarded after discussion on the mailing list because of the following reasons:

* It is only a single typo away from `Bar.foo(_:)` which references the method with one unnamed argument
* In argument lists `_` implies the *presence* of an unnamed parameter, while `_` in `Bar.foo(_)` would mark the *absence* of a any parameters.