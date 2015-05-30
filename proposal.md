#Less Restricted Mixins (Work in Progress)

## Contact information

1. **Gilad Bracha.** 

2. **gbracha@google.com.** 

3. **https://github.com/gbracha/lessRestrictedMixins** 



##Summary 

We propose to remove the some of restrictions on mixins curently in Dart. Specifically

* Mixins can refer to super.
* Mixins can have superclasses other than Object.

When a mixin is applied, the resulting type is a subtype of the mixin type, as it is today. However, if the resulting type would not otherwise be a subtype of the declared supertypes of the mixin, a warning is issued. Hence, a mixin application would have to declare that it implements the mixin's superclass and superinterfaces in order to be warning-free.


##Motivation

Ad hoc restrictions are a sign of bad design. The above restrictions were always intended to be temporary. They were not part of the original 
design, which proposed a semantics that fully supported all of the restricted features. This should be motivation enough. 

For the more pragmatic, users have repeatedly complained about the restrictions, which prevent the use of mixins in many situations where they might otherwise be helpful.


See bugs 
https://code.google.com/p/dart/issues/detail?id=12905
https://code.google.com/p/dart/issues/detail?id=12456

##Pros

* The language is more expressive and cleaner
* This proposal avoids the complexity associated with adding constructors to mixins

##Cons

* No support for mixins with (non-trivial) constructors.

##Examples


Example 1 (due to Jacob McDonald)

In Polymer 0.9 there is the notion of Behaviors of elements:

A behavior can define lifecycle callbacks, declared properties, default attributes, observers, listeners.

Almost all of these features could be handled today by mixins (although the semantics would be slightly different), with the exception of the lifecycle callbacks. These callbacks need to get invoked on each behavior, in the reverse order they they were mixed in. If mixins could do super calls, then this could potentially work out of the box:

```
class FooBehavior extends Behavior {
  void ready() {
    super.ready();
    print('FooBehavior ready');
  }
}

class BarBehavior extends Behavior {
  void ready() {
    super.ready();
    print('BarBehavior ready');
  }
}

class PolymerElement {
  void ready() {
    print('PolymerElement ready');
  };
}

class MyElement extends PolymerElement with FooBehavior, BarBehavior implements Behavior {
  void ready() {
    print('MyElement ready');
    super.ready(); // Call super at the end to emulate the polymer js semantics.
  }
}
```

When the `ready` method is invoked on `MyElement`, this will print:

MyElement ready
PolymerElement ready
FooBehavior ready
BarBehavior ready

In this case, none of the constructors along the superclass chain take parameters, which simplifies matters considerably. 



##Semantics

We propose to revise section 12 of the specification as follows:


##12 Mixins

A mixin describes the difference between a class and its superclass. A mixin is always derived from an existing class declaration.

~~It is a compile-time error if a declared or derived mixin refers to super.~~ It is a compile-time error if a declared or derived mixin explicitly declares a constructor. ~~It is a compile-time error if a mixin is derived from a class whose superclass is not Object.~~

These restrictions are temporary. We expect to remove them in later versions of Dart.
~~The restriction on the use of super avoids the problem of rebinding super when the mixin is bound to difference superclasses.~~

The restriction on constructors simplifies the construction of mixin applications because the process of creating instances is simpler.
~~The restriction on the superclass means that the type of a class from which a mixin is derived is always implemented by any class that mixes it in. This allows us to defer the question of whether and how to express the type of the mixin independently of its superclass and super interface types.
Reasonable answers exist for all these issues, but their implementation is non-trivial.~~


Furthermore, in section 12.1 Mixin Application, we add the following:

Let M_A be mixin derived from a class M.
If a class declaration includes M_A in its extends clause, then if the class is not a subtype of M, a warning is issued.


## 16.15.1 Method Lookup

The result of a lookup of a method m in object o with respect to library L is the result of a lookup of method m in class C with respect to library L, where C is the class of o.
The result of a lookup of method m in class C with respect to library L is: If C declares a concrete instance method named m that is accessible to L, then that method is the result of the lookup, and we say that the method was lookeup in C. Otherwise, if C has a superclass S, then the result of the lookup is the result of looking up m in S with respect to L. Otherwise, we say that the method lookup has failed.
The motivation for skipping abstract members during lookup is largely to allow smoother mixin composition.


## 16.15.2 Getter and Setter Lookup

The result of a lookup of a getter (respectively setter) m in object o with respect to library L is the result of looking up getter (respectively setter) m in class C with respect to L, where C is the class of o.
The result of a lookup of a getter (respectively setter) m in class C with respect to library L is: If C declares a concrete instance getter (respectively setter) named m that is accessible to L, then that getter (respectively setter) is the result of the lookup, and we say that the getter was lookeup in C. Otherwise, if C has a superclass S, then the result of the lookup is the result of looking up getter (respectively setter) m in S with respect to L. Otherwise, we say that the lookup has failed.
The motivation for skipping abstract members during lookup is largely to allow smoother mixin composition.

The text for super invocation (16.7.3) needs to be slightly revised so it is clear that the call is late bound (at least conceptually) to the next mixin application up the inheritance chain. Likewise sections 16.18.2, 16.18.6 (16.18.10 is ok). In both cases, one has to distinguish between S<sub>static</sub>, the superclass of the enclosing class, and S<sub>dynamic</sub>, the superclass of the mixin application executimg at the point of the call.

 To achieve this, we revise the initial section of 16.17.3

## 16.17.3 Super Invocation

A super method invocation i has the form super.m(a1,...,an,xn+1 : an+1,...,xn+k : an+k).
Evaluation of i proceeds as follows:

First, the argument list (a1,...,an,xn+1 : an+1,...,xn+k : an+k) is evaluated yielding actual argument objects o1, . . . , on+k. Let g be the method currently executing, and let C be the class in which g was looked up. Let S<sub>dynamic</sub> be the superclass of C ~~the immediately enclosing class~~, and let f be the result of looking up method (16.15.1) m in S<sub>dynamic</sub> with respect to the current library L.


In addition, we revise the last paragraph of 16.17.3 to state:

Let S_static be the superclass of the immediately enclosing class. It is a static type warning if S<sub>static</sub> does not have an accessible (6.2) instance
member named m unless S<sub>static</sub> or a superinterface of S<sub>static</sub> is annotated with an annotation denoting a constant identical to the constant @proxy defined in dart:core. If S<sub>static</sub>.m exists, it is a static type warning if the type F of S<sub>static</sub>.m may not be assigned to a function type. If S<sub>static.m</sub> does not exist, or if F is not a function type, the static type of i is dynamic; otherwise the static type of i is the declared return type of F.






##A Working Implementation

TBD

##Tests

See example 1.

TBD

##Patents Rights

TC52, the Ecma technical committee working on evolving the open [Dart standard][], operates under a royalty-free patent policy, [RFPP][] (PDF). This means if the proposal graduates to being sent to TC52, you will have to sign the Ecma TC52 [external contributer form]() and submit it to Ecma.

[tex]: http://www.latex-project.org/
[language spec]: https://www.dartlang.org/docs/spec/
[dart standard]: http://www.ecma-international.org/publications/standards/Ecma-408.htm
[rfpp]: http://www.ecma-international.org/memento/TC52%20policy/Ecma%20Experimental%20TC52%20Royalty-Free%20Patent%20Policy.pdf
[external contributer form]: http://www.ecma-international.org/memento/TC52%20policy/Contribution%20form%20to%20TC52%20Royalty%20Free%20Task%20Group%20as%20a%20non-member.pdf

