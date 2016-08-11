Title: Logging to a remote server in Dynamics online
Tags: ['DynamicsCRM', 'Logging', 'Plugin']
Published: 2016-08-11
Lead: Logging in Dynamics should be easy right?
---

As I mentioned in [a-poor-dynamics-plugin], diagnosing an issue can be challenging in Dynamics CRM Online. I have
not yet had the pleasure of working with an on-premise install of Dynamics. To simplify debugging issues in my
plugins, I felt it was desirable to log to a centralized log server.

I like to reuse other people's work where possible, so I first started investigating using NLog, which we had been 
using in the past for our projects and our newer code was already logging to Seq, so I was happy to find that there
was an NLog target for Seq.

Adding a reference to NLog was easy, and I wrote some preliminary configuration code to try it out.

When I ran this in the Plugin Registration Replay, I learned a few things.
* The Replay tool will accurately sandbox your plugin as if it were running in CRM.
* All of your plugin's code must be contained within a single asembly.
* NLog's initialization process uses code that is prohibited within the sandbox.

The first one is cool, it means that my code is highly likely to fail in the same manner as it would in CRM Online.

The second point is annoying, but not insurmountable. ILMerge.MSBuild.Tasks NuGet package solved that issue by just 
installing it, no further work was required (No furthwr work on the plugin itself at least, more on this in a later blog post).

The third however is a complete roadblock. Try though I might, I couldn't figure out a way to avoid some filesystem 
calls deep within the bowels of NLog.

Serilog, which is advocated by Seq, also suffers a similar problem, though if I recall correctly, the problem with 
Serilog was reflecting over private properties, which is also disallowed by the sandbox.

With NLog and Serilog being the only loggers that Seq natively supports, I decidedmit was time to unfortunately invest
in writing my own logging framework.

I chose to keep ot as minimal as possible, recycling as little as possible from NLog, keeping the expected ILogger 
interface and supporting Seq's structured logging.

Something I noticed in my earliest tests were that my logs were rarely being sent.

I do not know definitively what the problem was, but I suspect the most likely cause for this is the Replay tool 
immediately killing the AppDomain and it's threads before flushing the logs had completed. Because of this, I 
reluctantly implemented IDisposable and did the flush on the same thread as the plugin itself.

This has some really bad implications though, because Seq uses HTTP POST to receive it's I still have to find a good way 
to balance the 20 second time limit that is imposed on Plugins. If the logging service is down, there's a good chance
that CRM will rollback the transaction.

I'm of the opinion that a logging service is a nice debugging tool, if you need tracability, use CRM's auditing, if the
logging service is down, everything should continue, so I expect further development into this train of thought. 
Ideally I'd like to cause the AppDomain to delay unloading until a background thread is finished flushing.

Another issue I have yet to find a really good solution to is where to store the configuration of the Seq server, and
it's normal for us to include the environment in our logging. I don't really like the idea of having to configure the
server address and api key on a per-message basis, and using an entity worries me as the startup behaviour of plugin 
assemblies appears to be undefined, so I don't want to introduce another trip through the organisation service to discover
settings.

I've decided in the short term to introduce the values via the build system and keep them in the assembly until a better
solution comes to me.

I'll be putting the code on GitHub as soon as feasible.