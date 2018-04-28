Title: "Flex: 1046: Type was not found or was not a compile-time constant:"
Published: 2012-01-16
Tags: [ 'Adobe Flex' ]
---
#The Problem
At my work, we have a legacy application known internally as “The Flex app”, maintaining this piece of software was the very reason I was hired, but as it turns out, Some of my other skills turned out to be more useful than my adaptability with new languages. Having said that, every now and then I need to do maintanence on this application, so, in accordance with the bug requests, I booted up Flex Builder 3.0.2 rolled back any outstanding changes and prepared to start again with a fresh slate. One problem: The clean project wouldn’t build.

`1046: Type was not found or was not a compile-time constant: LogEntry`
 

#The Investigation
Anybody with any experience with Flex would know the solution to this problem within seconds: You’re forgetting an import. Bring in your type and everything will be fine.

Unfortunately, my case was not so simple. Three people checking, double checking, so on were not able to help me figure out why I had an undefined constant:

* My Imports were in place
* The IDE recognised the types
* Removing them and using the IDE’s Autocomplete added the imports back in
* The Files containing the types were fine
* Removing the import and using the full package path did not solve the problem.

The fact that the IDE was able to find the types, but the compiler couldn’t intrigued me most and after a long time of scratching my head, I had one of my “Gut Feelings”

As I’m primarily a C# developer, sometimes I forget what land I’m programming in. In this particular instance, I had imported my WCF services into flex and accidentally prefixed the package names with Uppercase letters, like .NET namespaces. During the lifecycle of working on the app, I had deleted and recreated the services using varying casing for packages.

#The Solution
So, the solution was to make my filesystem folders consistent with the casing of the packages.

#Final Thoughts
This is a “bug” in the compiler, the IDE is not affected by this problem. Clearly this would be an issue on Linux or on a case-sensitive OS X filesystem, but I was a little surprised to run against it on Windows.

I can’t help but wonder if even Java is this pedantic about casing.
