Title: Hello Dynamics World
Lead: A quick summary of my reaction to being immersed in Dynamics, and why I started this blog.
Published: 5/8/2016
Tags: DynamicsCRM
---
Over the last month and a bit or so at my work, I've been working on replacing a legacy system with Dynamics CRM. 

I'm optimistic that Regardless of how people feel about it, I think it will be a good fit for the system it's replacing.

Straight out of the gate however, there are a number of things that worried me.

First, the plugin code that I had been given had no tests, everything was static methods with raw linq-to-crm, so it 
wasn't even designed with testing in mind.

Secondly, the front end, a BA had written a bunch of JavaScript to add some nice features, but the JavaScript was 
potentially quite buggy in the way it was implemented.

Thirdly, I was entering the Dynamics scene with no training, and the knowledge that people have done a lot with 
Dynamics, but it's also really easy to screw it up.

With that in mind, I'd like to start a series of blog posts about how I reengineered our plugins with the following 
goals in mind:
1. The plugins should be unit testable
2. The plugins should use IoC framework so that they only contain what they need to function.
3. The plugins should provide insight into how they're functioning

Meanwhile on the frontend, I've set up a WebPack workflow to allow us to write ES6/7 JavaScript using Async/Await to 
improve the readability and durabiliy of the JavaScript.

It is my hope that my Blog will help others to write better code and provide a place where people can offer me advice 
to how I can improve our workflow as well.