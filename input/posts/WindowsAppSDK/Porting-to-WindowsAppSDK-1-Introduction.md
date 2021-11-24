## Title: Porting Drawboard Projects to Windows App SDK 1.0 Lead: Trying out Microsoft's new framework Tags: [ WindowsAppSDK, WinUI, UWP, DrawboardProjects ] Published: 2021-11-22

This is a series of blog posts that details the experience I recently had trying
to port a large, existing codebase with all its complexities to Microsoft’s new
Software Development Kit (SDK) for modern applications.

# Introduction

As a UWP developer, I'm always interested in what's happening in the technology
space that I work in. News has been somewhat slow about developments in the UWP
space for a while as Microsoft had focussed its efforts on something called
"Project Reunion", They had been working to separate out the UI framework from
UWP away from the operating system so that it would be reusable in other places
in the operating system.

Unfortunately, as a result, UWP's future seems currently unwritten, and while
spokespeople from the company have come out and said various things that are
often construed by the technologist media as UWP is dying and on the way out.

I don't necessarily believe that to be the case, but the company just released
1.0 of the Windows App SDK, and I was curious to see what it was about and
porting the product I'm most familiar with to WinUI seemed like it would be a
bit of fun for a while.

## Who is this for?

I'll try to keep the jargon down in this introduction as much as possible for
technical leads and managers with technical knowledge, but the audience for the
subsequent posts will be targeted at developers who would be implementing such a
change.

## What is the Windows App SDK and WinUI?

In Microsoft's words,

>   Windows App SDK is a set of libraries, frameworks, components, and tools
>   that you can use in your apps to access powerful Windows platform
>   functionality from all kinds of apps on many versions of Windows. Windows
>   App SDK combines the powers of Win32 native applications alongside modern
>   API usage techniques, so your apps light up everywhere your users are.

There are many ways to interpret this, and in many cases, they are all true.

-   You can use the Windows App SDK to interface with modern windows features
    from within an existing Win32 application

-   You can use the Windows App SDK to write a modern application that calls
    older Win32 APIs, as a new desktop platform

-   In theory, you can use the Windows App SDK from within UWP to detach your
    reliance on the XAML that is provided by the operating system. This is not
    currently a supported feature however and is in beta for the foreseeable
    future.

WinUI 3 is the library in which Microsoft has taken the UWP XAML UI engine and
open sourced it, separating it from the Operating System.

There are several things to think about when considering a port to the new SDK,
to be weighed against the benefits and cons of continuing to use UWP.

## Is it worth it?

### Uncertainty

Microsoft have committed to at least providing [Bug, reliability, and security
fixes](https://github.com/microsoft/WindowsAppSDK/discussions/1615), but we will
not be seeing support for .net 6. In fact, the last available version of .net
for UWP is .net core 2.1, which is no longer supported by the company.

Normally this would be simply an eyebrow-raiser and I would just carry on with
my job. I can't use .net 5's runtime, but all the syntactic sugar of modern C\#
is available by setting a property in my project files, so I can live with that.

But there is a real issue here that I have been bit hard by in the past. .net
core 2.1 only supports libraries that are .net core 2.1 and earlier, or .net
standard 2.0 and earlier.

.NET Standard is a transitional library technology that allowed libraries to run
a common set of code against various versions of the .NET runtime. That
transition period is over now, and more and more developers will start to create
new libraries that specifically target .net 5 and 6, leaving .net standard 2.0
behind.

This was a trend I saw during my work at Somark, when we chose to develop our
application in UWP on top of Windows IoT Enterprise LTSC (Long term service
channel, windows 1703), which was limited to .NET Standard 1.3.

When one of our core libraries (Template10) dropped support for our version of
UWP, we had to start maintaining our own fork of it in order to fix bugs which
was no small amount of overhead.

The risk then, is that our ability to deliver features and fix bugs is already
going to be limited by vendors who wish to adopt the latest language and runtime
features. If they chose to no longer support our version of .net standard or
UWP, we could get stuck with the burden of maintaining someone else's old
project.

### The benefits of switching

To consider then the benefits of switching to the App SDK, we get all the
advancements that have been made in .net 3.0, 3.1, 5 and 6.

Every one of those releases has brought about amazing performance enhancements,
especially thanks to the new Span and Memory types, which, when used well, can
allow applications to use memory a lot more efficiently.

One of my favourite features that's been added since then are [Source
Generators](https://docs.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview).
These are small utilities that hook into the build process that can
significantly reduce the amount of code that developers need to write, while
simultaneously making applications run faster, by making code that writes more
code. The main benefit of this is it means we can pre-calculate a lot of things
when the application is being compiled, instead of when the user is running it.
They are not a silver bullet though! I've written a Source Generator and while I
perceive the benefit to be immense, they are complicated beasts in their own
right!)

Project files have been truncated, they no longer need to list every single file
that makes up the project, which means less merge conflicts.

Then there is WinUI 3, the UI library provided by the Windows App SDK. By moving
to WinUI 3, you can unshackle your code from Windows release cycles, if you've
been forced to test your application on many different versions of windows to
ensure the UI is consistent through the different versions, this is where WinUI
can help, because it gives you the ability to update when YOU and YOUR company
have time, rather than the windows release cycle. I will need to see a few
releases of windows to see if this turns out to be true.

Finally, your application will run at full-trust instead of partial trust,
granting you the ability to interact with a user's full filesystem, the registry
or interact with other processes in a way that was previously prohibited by
UWP's sandboxing.

### Time taken

I undertook this as a personal investigation, as such it was generally performed
after-hours, about 4 hours a night for a week, and about 12 hours over two
weekends, to get the application to a "functional, but buggy" state. Enough to
discover where our pain points for the future would be, and to make a call
whether the App SDK is mature enough to migrate to.

A large chunk of the time was spent looking for workarounds that had not yet
been discussed in the community or have been in flux through the various
previews of the SDK. As the SDK matures and these conversations are had, the
information I struggled to find will get easier to locate, and better
documented.

Considering the scale of the change, I’d say it was quite rapid to perform the
conversion.

### Bugs

There will be bugs. Ripping out the runtime that underpins your whole
application is a destabilizing action and updating your libraries could
potentially mean that the API surfaces will change. Features that were not
available to UWP may light up and become available to you.

Expect there to be a big effort in tracking down some obscure behaviour that's
been uncovered, and don't be surprised if you unearth bugs that you didn't know
you had. (I did!)

### Testing

Expect your testing effort to be monumental in this, it will be an
all-hands-on-deck, side-by-side full regression testing effort. You'll need to
test every button, and every permutation of every button, dragging, clicking,
double-clicking.

## Not covered

There are several topics that I will not be covering immediately in this series.
This experiment was to determine whether the App SDK is ready for use in our
application, there has been no decision made whether to commit to the port or
not, and as such, I haven't done anything with our build pipeline, and I haven't
investigated the impact of releasing it to the store, and discussions would need
to happen with my team about what new .net features do we embrace, and are there
any that we collectively wish to avoid.

## Will we be committing to moving Drawboard Projects to the new SDK?

I'll update this blog when we make that decision.

The application's core functionality works, it's possible to commit to it, but
we absolutely need a workaround for the lack of support for
[CameraCaptureUI](https://portal.productboard.com/winappsdk/1-windows-app-sdk/c/49-support-cameracaptureui?utm_medium=social&utm_source=portal_share)
which is not yet available for 1.0 and not planned for 1.1, but should be later.

We're also acutely aware of the fact we want to migrate to using UWP's
InkControl for better inking, the Windows App SDK team are considering [Inking
Controls](https://portal.productboard.com/winappsdk/1-windows-app-sdk/c/31-inking-controls?utm_medium=social&utm_source=portal_share),
but it's also something we have not yet committed time to do either. There is
potentially time for Microsoft to implement it.
