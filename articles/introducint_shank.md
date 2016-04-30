Dagger is awesome. It changed the way we write code especially on
Android. I felt awesome using it, .... 
I think it's fair to say Dagger is a difficult library to truly master.
It takes time to fully grasp how to use it, and it takes
even more time to understand how to use it in scenarios much more
complex than the ones found in the simple examples you can find online.
Ok, by now you may have become a master and find it intutitive, but if
you have tried to explain it to a junior colleague, you should
appreciate again how hard it is to fully wrap your head around
it. Is it because we're dumb? That may be part of the answer, but maybe
there simply is a lot going on at once in this library.

Let's have a look at the most basic example:

# dependencies
class A {
  A(B b) {
    this.b = b;
  }
}

class B {
}

class C {
  @Inject A a;

  void doSomething() {
  }
}

@Module
class Module {
  @Provides static A provideA() {
    return new A(new B());
  }
}

@Component(modules = Module.class)
interface Component {
  C maker();
}

class App {
  void onCreate(){
    Component component = DaggerComponent.builder().module(new
    Module()).build();
    component.maker().doSomething();
  }
}

This is the most basic example I can think of. We're not doing anything
fancy, not dealing with scopes, named factories, singletons, or nested components, this is just to
get the ball rolling and providing the most basic dependency.

Now, let's look at how many concepts we have to deal with:

Multiple interfaces:
@Module
@Component
@Inject
@Provides

Generated code (where did it come from? why?)
DaggerComponent, .make()

Ok, @Provides specifies how to provide something, it's an annotation for
a factory method. Makes sense.
@Inject injects the class. Makes sense. But sometimes it means that a
constructor should be called when doing injection. It is both a
way of putting stuff in the graph and getting it out of it. What?
@Module seems reasonable, it's a place in which several @Provides live.
@Component... what is this thing? It's an interface, but I'm not
implementing it anywhere. You can specify both void methods that take
arguments and non-void methods that take no arguments. Why? Ok it's
the glue between the injector and the injectee. But why do I need to
deal with this stuff? And where is the code coming from?

Now, dependency injection per-se is a super simple principle, so why is
using it so hard? Essentially all we want is to specify how to create
something, and then use it somewhere else. That's it, right? So here's
what the most basic injection looks in Shank (static imports for brevity):

registerFactory(A.class, () -> new A(new B()));
A a = provideNew(A.class);

That's it! This is essence of Shank: you specify how you create something,
and then you get it when and where you need it. Now this super trivial example
is useless in practice, but it helps learning about how this library
works. The principle is very very simple, so the most basic example
should be very very simple too.
Anyway, let's look at a super basic production-like setup, it's still pretty
straightforward:

class MyModule extends ShankModule {
  public void registerFactories() {
     registerFactory(A.class, () -> new A(new B()));
  }
}

class App {
  void onCreate(){
    initializeModules(new MyModule())
  }
}

class C {
  final A a = provideNew(A.class);
}

In total you need three static, documented methods, and a class that
implements an interface.

That's it. No annotations, no code generation, no magic. Just Java, and
the code you wrote. You are in the driving seat.

Let's explore it in more detail:
`registerFactory` does just that, registers a factory for the specified
class.
`provideNew` does just that, gives you a new instance of the specified
class.
`initializeModules` takes a vararg of `ShankModule`s and calls
`registerFactories` in them iteratively, which allows you to separate
and organize your creation logic by feature or whatever criteria you
might prefer.

Ok, so far this has been way too basic, new stuff all the time?
Boring. Let's make a singleton!

A c = provideSingleton(A.class);

There, done. Next time you call it you will get the same instance.

What about scopes? Say I want a the same instance of an object until I
am no longer interested and want to free the memory (think of an Android
activity and stuff that needs to survive rotation but not the total
lifetime of the Activity). For that you use a Scope. Scope is a class
that you instantiate, it takes an object as a parameter, and for equal objects,
you get an equal scope.

A a = with(scope(MainActivity.class)).provideSingleton(A.class);

What does this do? It provides a singleton for class A, which is scoped
to MainActivity.

Calling

A a = with(scope(OtherActivity.class)).provideSingleton(A.class);

will give you another singleton instance, because now you used a different scope.

When you are done with that scope, you may clear it with:

  scope.clear()

On Android, this is typically done in onDestroy when isFinishing is
true. That way you avoid leaks, but can deal with rotation/screen
reconfigurations.

Dagger has a way of specifying named @Provide and @Inject, so you can
have different "flavours" of the same class. That's often useful,
Shank also allows you to do that and it loooks like this:

registerNamedFactory(A.class, "flavourZero", () -> new A())
registerNamedFactory(A.class, "flavourOne", () -> new SubclassOfA())
registerNamedFactory(A.class, "flavourTwo", () -> new AnotherSubclassOfA())

and here's how you get these flavours, in new, singleton and scoped
singleton because why not:

A flavourZero = named("flavourZero").provideNew(A.class);
A flavouroOne = named("flavourOne").provideSingleton(A.class);
A flavouroTwo = with(scope).named("flavourTwo").provideSingleton(A.class);

This alone is enough to replace Dagger. But it doesn't end here. One of the
hardest things to do with Dagger for me was being able to inject a
parameter *into* the factory, so I can get the most appropriate object.
Say you have a UserPresenter, and in it you want do do a REST request on
a UserRestService that depends on a UserId.

This is how you do it. First, you just register a factory that takes a
parameter:

registerFactory(UserPresenter.class, (userId: String) -> {
  UserRestService userRestService = provideNew(UserRestService.class)
  return new UserPresenter(userRestService, userId)
}

and when you get it, you just pass it in!

UserPresenter userPresenter = provideSingleton(UserPresenter.class, mUserId);

And that's it. You just can't do that in Dagger as easily.

Performance


Downsides and gotchas:
Runtime errors
