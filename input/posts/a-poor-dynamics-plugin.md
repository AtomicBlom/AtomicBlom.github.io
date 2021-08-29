Title: A Poor Dynamics Plugin
Lead: A dissection of the first Dynamics plugin I ever saw
Tags: [ 'DynamicsCRM', 'Plugin', 'Unit Testing', 'IoC' ]
Published: 2016-08-07
---
I'd like to show you a slightly modified version of a plugin. This was the first piece of code I had ever seen for the purpose of extending Microsoft Dynamics' backend.

The reason I was asked to look at this code was because someone had changed something and the plugin was no longer working.

The first bit of code I ever looked at for dynamics looked something like this:

```csharp
public class WebInteraction : IPlugin
{
    /// <summary>
    /// Wrapper function for plugin.
    /// </summary>
    /// <param name="serviceProvider"></param>
    public void Execute(IServiceProvider serviceProvider)
    {
        IPluginExecutionContext context = (IPluginExecutionContext)serviceProvider.GetService(typeof(IPluginExecutionContext));
        IOrganizationServiceFactory serviceFactory = (IOrganizationServiceFactory)serviceProvider.GetService(typeof(IOrganizationServiceFactory));
        IOrganizationService service = serviceFactory.CreateOrganizationService(context.UserId);

        ITracingService tracingService = (ITracingService)serviceProvider.GetService(typeof(ITracingService));
        
        Entity preImageEntity = new Entity();
        Guid OpportunityID = Guid.Empty;
        //The InputParameters collection contains all the data passed in the message request.
        try
        {
            if (context.Depth == 1)
            {
                string messageName = context.MessageName.ToUpper();

                if (messageName == "CREATE")
                {
                    CreateHandler(context, service);
                }
                if (messageName == "UPDATE")
                {
                    UpdateHandler(context, service);
                }
                if (messageName == "DELETE")
                {
                    DeleteHandler(context, service);
                }

            }
            
        }
        catch (Exception ex)
        {
            tracingService.Trace(ex.Message);
        }
    }   // end of method Execute()

    private void CreateHandler(IPluginExecutionContext context, IOrganizationService service)
    {
        if (context.Stage == 20)    // pre-operation stage
        {

        }
        else if (context.Stage == 40)   // post-operation stage
        {
            if (context.InputParameters.Contains("Target") && context.InputParameters["Target"] is Entity)
            {
                Entity target = (Entity)context.InputParameters["Target"];
                
                // only care if Type = Web,
                if (target.Contains("new_type") && target.GetAttributeValue<OptionSetValue>("new_type").Value == 863580000)
                {
                    DoImportantBusinessLogic();
                }
            }
        }   // end of else if

    }   // end of method CreateHandler()

    private void UpdateHandler(IPluginExecutionContext context, IOrganizationService service)
    {
        if (context.Stage == 20)    // pre-operation stage
        {

        }
        else if (context.Stage == 40)   // post-operation stage
        {
            if (context.PostEntityImages.Contains("PostImage") && context.PostEntityImages["PostImage"] is Entity)
            {
                Entity postImage = context.PostEntityImages["PostImage"];
                
            }
        }   // end of else if

    }   // end of method UpdateHandler()

    private void DeleteHandler(IPluginExecutionContext context, IOrganizationService service)
    {
        if (context.Stage == 20)    // pre-operation stage
        {

        }
        else if (context.Stage == 40)   // post-operation stage
        {
            if (context.PreEntityImages.Contains("PreImage") && context.PreEntityImages["PreImage"] is Entity)
            {
                Entity preImage = context.PreEntityImages["PreImage"];
                
            }
        }   // end of else if

    }   // end of method DeleteHandler()
}
```

This strikes me as some code that someone has grabbed from a template, messed with the bare minimum to get it running and hasn't even stripped out the code that does nothing.

I'm grateful that they at least left in the comments as to what context.Stage is, because if I'd just seen `if (context.Stage == 40)` I'd be mighty confused.

Let's strip this down to what they actually used.

```csharp
public class WebInteraction : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var context = (IPluginExecutionContext)serviceProvider.GetService(typeof(IPluginExecutionContext));

        try
        {
            if (context.Depth != 1) { return; }
            if (context.Stage != 40) { return; } // post-operation stage
            if (!"CREATE".equals(context.MessageName.ToUpper())) { return; }
            
            if (!context.InputParameters.Contains("Target") || !context.InputParameters["Target"] is Entity) { return; }
            
            var target = (Entity)context.InputParameters["Target"];
            
            // only care if Type = Web,
            if (target.Contains("new_type") && target.GetAttributeValue<OptionSetValue>("new_type").Value == 863580000)
            {
                DoImportantBusinessLogic(context);
            }
        }
        catch (Exception ex)
        {
            var tracingService = (ITracingService)serviceProvider.GetService(typeof(ITracingService));
            tracingService.Trace(ex.Message);
        }
    }
}
```

That wasn't so hard, so, to sum this up:
* the context must be Depth 1
* in Post-Operation stage (Stage 40)
* the message must be "Create"
* it must contain an InputParameter called "Target" which must be of type Entity
* the target must contain an attribute called "new_type" which has an option set value of 863580000, which the comment alludes to is "web"

Only after all this is met can the plugin execute its purpose.

A few more things come to mind after cleaning this up.
1. Why are all the properties of Dynamics objects keyed by string?
2. What is Depth?
3. What are the stages?
4. Why do option sets have such obtuse numbers behind them
5. The plugin was clearly failing, how can I get those trace logs.
6. Why catch and log? Why not report an error to the caller?

To add to all this and make things more interesting, we'll take a quick peek at an example of the business logic.

```csharp
private void DoImportantBusinessLogic(IPluginExecutionContext context, IOrganizationService service, Entity interaction)
{
    string email = interaction.GetAttributeValue<string>("new_email");
    Entity contact = SearchContactByEmail(context, service, email);
    
    if (contact != null)
    {
        //Further Process account
    }
}

private EntityCollection SearchContactByEmail(IPluginExecutionContext context, IOrganizationService service, string email)
{
    string fetchXml = @"<fetch version='1.0' mapping='logical'>
                        <entity name='contact'>
                            <order attribute='modifiedon' descending='true' />
                            <filter type='and'>
                                <condition attribute='statecode' operator='eq' value='0' />
                                <condition attribute='emailaddress1' operator='eq' value='{0}' />
                            </filter>
                        </entity>
                        </fetch>";

    fetchXml = string.Format(fetchXml, email);
    EntityCollection ec_contacts = service.RetrieveMultiple(new FetchExpression(fetchXml));

    return ec_contacts;
}
```

This code at least is only the core logic, there's no extraneous boilerplate code here which is nice, other than their passing around the context and the IOrganizationService to every method.

This code also suffers from the magic string madness for attribute values and then takes it to the next level using Dynamics' "FetchExpression" xml system.

To figure out why the plugin wasn't working, I started with trying to get at the tracing service, and I was completely blindsided by the way the tracing service works, specifically with CRM Online.

You see, in a CRM instance that has been provisioned using the on-premise version of CRM, the trace logs are actually available to you. In the Online version however, getting the trace logs requires a bit of work, some googling lead me to this workflow:
1. Start by Downloading the SDK.
2. Open the PluginRegistration Tool
3. Find the message that's failing
4. Start profiling the message.
5. (You may need to install the Plugin Tracing solution)
6. Trigger the message on CRM
7. You should get an error
8. Inspect the error to get a package you can save to your PC.
9. Go back to PluginRegistration Tool
10. Open the Plugin Replay Tool
11. Pass in your context from the error
12. Give it the path to a copy of the assembly
16. Invoke the plugin.

If all goes well, anything sent to the tracing service will be logged in the replay tool.

If that doesn't give you enough information, you'll need to debug the plugin:
14. Attach Visual Studio to the PluginRegistation tool with your solution open
15. Set a breakpoint where you wish to inspect.
16. Invoke the plugin again.

In my case, there were a few caveats to this. Turns out my message wasn't being invoked from the UI, but rather from an external API, with that in mind, and given the nature of the actual plugin, I'm not completely surprised that they weren't sending any information back to the service. It does however leave me wondering, how is a user of the system supposed to know that this business process failed? I have to go through 16 steps to figure out why as a developer?

In my case, the error was that one of the columns that used to exist and no longer does, I spoke with the various power users, and one explained he had renamed it because it had been inappropriately and confusingly named to begin with.

With that as my first experience, there were a few things I had resolved to do while I'm working on this system.
1. Introduce logging to an external log server.
    * This would give me insight into the fact there was an error in the first place
2. Use Strong typing for entities
    * Using CrmSvcUtil to generate classes for us means that a change in schema will break code
3. Use Linq to CRM where possible
    * Linq is commonly used by my workmates, they're more likely to understand it than Fetch XML.
4. Refactor the code to support unit testing.
    * This means bringing out the core logic into a seperate class and splitting the inputs and outputs into their own mechanisms.
5. Clean up the code using IoC.
    * passing around the context doesn't help you know what parts of the plugin need what.
6. Invest in continuous integration
    * The plugin should be doing a nightly build, and part of the nightly build should be updating the strong typing mappings.
    * Any changes in schema can be caught nightly.
7. Isolate your code from CRM
    * I've heard from others that CRM is easy to use, and easy to screw up.
    * Separating my classes from the CRM interactions means I can adjust for anything I'm doing wrong relatively quicker
    * Also, my code should be far easier to test and care less about misconfiguration issues like wrong message types and wrong stages.
8. Invest in continuous deployment.
    * make it so that a checkin on my code should run all the tests and deploy to the staging site.
    * unit tests should pass before the code is deployed.

Over the next few blog posts, I'll be discussing approaches to these points and how I've tackled them so far.