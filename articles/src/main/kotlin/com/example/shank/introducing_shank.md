## I made a Shank to do Dagger's job

Dependency injection is a simple concept, and a superbly useful one too.
Most of the time dependency injection is done simply by adopting some
coding practices, such as passing dependencies into a class from the constructor. However, in certain cases this is not possible as your classes are not instantiated by your code, but by the framework you are using. In such cases it becomes necessary to use some more complicated patterns, which may lead to a lot of boilerplate code. In light of this, there has been a succession of (in my opinion) increasingly better frameworks that helped developers adopt dependency injection in their applications. I think it's possible to group dependency injection frameworks into different generations, each generation defined by some kind of new technology or innovation that came about before it. 
Generation 0:
Service locator pattern based. Handwritten. GOF was published and popularized the service locator pattern. This resulted in a lot of boilerplate code, but it can be accomplished by using even a fairly primitive OO language, and thre is very little magic involved. Many teams still go for this in new apps in spite of its drawbacks.

Generation 1:
Reflection and XML- Spring is the posterchild of this generation. As Java introduced reflection abilities and XML seemed to be all the rage, it seemed like a neat idea to use both to come up with a framework that tackled dependency injection. Not sure who'd still prefer all that XML in a modern project, but hey.

Generation 2:
Generics - Guice used the neat new language feature introduced by Java 5 to add type safety to Spring and get rid of all that XML. It became hugely popular and it's still going strong! 
Generation 3:
Generics + codegen + reflection: Dagger 1 and 2. Although codegen had been around for ages, it was a
stroke of genius that of trying to apply it to dependency injection.
This allowed to really get rid of a lot of boilerplate, while achieving
much better performance, as the code created looks like the hand written
code from generation 0. Regarded by most as the pinnacle of DI
frameworks. Hugely popular in the Android community due to performance
and memory considerations.


We have come a long way from the beginning, and there's
little doubt that for most use cases Dagger 1 and 2 are a much better
alternative to what came before them. But while they do have a lot of
advantages, they still come at a cost. They're rather tough to learn and
truly understand how they work. I think Uncle Bob has a point in his article 
https://blog.8thlight.com/uncle-bob/2015/08/06/let-the-magic-die.html
If you do not fully understand the libraries you use (as in you could
actually write them yourself) you are essentially practicing magic, and
with magic come surprises and sometimes even superstitious behaviours 
("I don't know if this really works, but I'll put in there because it seems to work").

Now it's true that Dagger generates human-readable code, but I think
it's safe to say very few people actually know exactly *how* this code
is generated and why. Now some might say this is not a big problem,
because as long as it works it's fine, but still you are essentially
practicing magic, and at some point you'll want to do a new magic trick,
and you're out of luck. I've seen this happen in various projects, and
I've seen really senior devs completely melt their brains understanding
how a complex app that used nested components and scopes fit toghether
(although they were amongst the ones who wrote it!).

This lead me to think of a question: if dependency injection is such a
simple concept, why does it have to be so hard to learn and use a
dependency injeciton framework? And more importantly, is all this magic
really necessary to avoid having a lot of boilerplate?

I went back to the basics, the good old Service Locator Pattern from the
Gang of Four, which many teams still prefer it to modern libraries. That
gave me the concept of factories for the dependencies. Nothing new.
Then I married it to the concept of Typesafe Etherogeneous containers from Effective
Java, which I use for getting stuff created by the factories out of the
container in a typesafe way.
Then I took advantage of a new Java feature, lambdas and functional
interfaces, which allows to really reduce a lot of boilerplate when
writing the factories. Again nothing you can't do with anonymous inner
classes, but the much better syntax really can open new doors to what's possible.

So I put all of this pre-existing stuff into a very simple library.
It's plain Java, it does not generate any code and it does not use
reflection. As it's based on the Service Locator Pattern, and there's no
reflection involved is still pretty fast and performant. And you won't
have to write more code than you do in production-like apps that use
Dagger.

Nothing you could not write yourself, nothing you could not understand
by reading the few hunded lines of its source code. In an homage to
Dagger, and in recognition of how relatively less sophisticated it is, I
called it Shank. It does the same job of Dagger, but anyone could make
one if they needed to. Yeah it's a bit ghetto but hey, I just need
something to get the job done man!

I'm not sure in what generation this would fall. It's been enabled by
new stuff (lambdas) but it's mostly a rehash of old concepts. Also it's
a bit meh technically, so for now I'm putting it in "generation meh".


Let's have a look at the most basic need for injection, a
framework-instantiated class C, which depends on B which depends on A.

# dependencies
class B {
  A a;
  B(A a) {
    this.a = a;
  }
}

class A {
}

# class instantiated by other framework (can't call this constructor)
class C {
  B b;

  void doSomething() {
  }
}

Here's what the most basic injection looks in Shank (static imports for brevity):

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
