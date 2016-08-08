Title:Logging to a remote server in Dynamics online
Tags: ['DynamicsCRM', 'Logging', 'Plugin']
Lead: Logging in Dynamics shouldbe easy right?
---

I like to reuse other people's work where possible. That's why when I wanted to introduce some logging from Dynamics, I first 
started investigating NLog, which we had been using in the past for our projects.

Our newer code was already logging to Seq, so I was happy to find that there was a Target for Seq.

Adding a reference to NLog was easy, and I wrote some preliminary configuration code to try it out.

When I ran this in the Plugin Registration Replay, I learned a few things.
* The Replay tool will accurately sandbox your plugin as if it were running in CRM.
* All of your plugin's code must be contained within a single asembly.
* NLog's initialization process uses code that is prohibited within the sandbox.

The first one is cool, it means I can relatively accurately test code against a Dynamics server.

The second point is annoying, but not insurmountable. ILMerge.MSBuild.Tasks NuGet package solved that issue by just installing it.

The third however is a complete roadblock. Try though I might, I couldn't figure out a way to avoid some filesystem calls deep
within the bowels of NLog.

Serilog, which is advocated by Seq, also suffers a similar problem, though if I recall correctly, the problem with Serilog was
reflecting over private properties, which is also disallowed by the sandbox.

With Nlog and Serilog being the only loggers that Seq natively supports, It's time to invest in writing my own logging framework.

I chose to recycle as little as possible from NLog, while keeping the expected ILogger interface and supporting Seq's structured logging.

Something I noticed in my earliest tests were that my logs were rarely being sent.

The most likely cause for this is the Replay tool immediately killing the AppDomain and it's threads before flushing the logs had completed.

Because Seq uses HTTP POST to receive it's I still have to find a good way to balance the 20 second time limit on plugins