#Less Restricted Mixins (Work in Progress)

## Contact information

1. **Gilad Bracha.** 

2. **gbracha@google.com.** 

3. **https://github.com/gbracha/lessRestrictedMixins** 



##Summary 

We propose to remove the some of restrictions on mixins curently in Dart. Specifically

* Mixins can refer to super.
* Mixins can have superclasses other than Object.

Specifically, mixins can be derived from classes whose superclass is not Object. The code in a mixin can refer to the superclass via super.  When a mixin is applied, the resulting type is a subtype of the mixin type, as it is today. However, if the resulting type would not otherwise be a subytpe of the declared supertypes of the mixin, a warning is issued. Hence, a mixin application would have to declare that it implements the mixin's superclass and superinterfaces in order to be warning-free.


##Motivation

Ad hoc restrictions are a sign of bad design. The above restrictions were always intended to be temporary. They were not part of the original 
design, which proposed a semantics that fully supported all of the restricted features. This should be motivation enough. 

For the more pragmatic, users have repeatedly complained about the restrictions, which prevent the use of mixins in many situations where they might otherwise be useful.

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


When the `ready` method is invoked on `MyElement`, this will print:

MyElement ready
PolymerElement ready
FooBehavior ready
BarBehavior ready

In this case, none of the constructors along the superclass chain take parameters, which simplifies matters considerably. 



##Semantics


##A Working Implementation

TBD

##Tests

TBD

##Patents Rights

TC52, the Ecma technical committee working on evolving the open [Dart standard][], operates under a royalty-free patent policy, [RFPP][] (PDF). This means if the proposal graduates to being sent to TC52, you will have to sign the Ecma TC52 [external contributer form]() and submit it to Ecma.

[tex]: http://www.latex-project.org/
[language spec]: https://www.dartlang.org/docs/spec/
[dart standard]: http://www.ecma-international.org/publications/standards/Ecma-408.htm
[rfpp]: http://www.ecma-international.org/memento/TC52%20policy/Ecma%20Experimental%20TC52%20Royalty-Free%20Patent%20Policy.pdf
[external contributer form]: http://www.ecma-international.org/memento/TC52%20policy/Contribution%20form%20to%20TC52%20Royalty%20Free%20Task%20Group%20as%20a%20non-member.pdf

