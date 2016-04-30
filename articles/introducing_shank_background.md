Introducing Shank - a simpler Dagger

Dagger is one of those libraries that changed the way we develop Android
apps. It sucessfully popularized the concept of dependency injection in
the Android world. However, in my experience working with different
teams and people I found it a bit hard to ensure that everyone in the
team is perfecty comfortable in utilizing it. Although the principle of
dependency injection is very straightforward, using Dagger in large and
complex projects isn't. The API is very powerful and it relies a lot on
annotation processing and code generation. Although it is definitely
possible to spend time finding the generated classes, read the generated
code, and even debug it, that is something that not every developer is
ready to do in order to get their job done. There naturally will be
"experts" in a team, and the other members will rely on them to set up
and mantain Dagger-related stuff. For me, the turning point was when
working on a high profile project for a now defunct client. They had put
toghether a "dream team" and I had the pleasure to work with some really
talented professionals. The tech lead was extremely capable, with
decades of experience under his belt. It came a point when we needed to
use Dagger in a somewhat unusual way, as we were developing a pretty
unusual app. We spent several days trying hard to get Dagger to serve
our purpose, and in the end we gave up. I don't think that the issue was
that Dagger was inadequate to do the job, but that we, despite having
considerable experience in dependency injection and in Dagger, we were
still unable to completely understand how it worked under the hood. Now,
sure, we could have spent even more time debugging line by line and
digging deep in the generated source code, but at the end of the day,
that was just one task out of many, and we had other priorities. The
tech lead then made a suggestion that I initially thought was
ridiculous: we should get rid of Dagger and roll out our solution. Now
that's crazy. Dagger was rooted deeply at the core of our application,
and for all I knew at the time, it was a magic library that was
absolutely impossible to match unless a great team of super talented
indivituals was put to the task and spent a few months working on a
solution.

The project imploded soon after, for unrelated business reasons, but the
idea stayed with me. Maybe it was possible to create something that
worked better.



