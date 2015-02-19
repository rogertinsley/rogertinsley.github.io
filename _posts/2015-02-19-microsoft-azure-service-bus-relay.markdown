---
layout: post
title: "Microsoft Azure Service Bus Relay"
date: 2015-02-17 21:42:41 +0000
comments: true
---

##What is the Microsoft Azure Service Bus Relay?

A really powerful feature of Microsoft Azure is the Service Bus Relay. In architectural terms, relays provides a mechanism to connect distributed client applications or cloud services to a projected on premise endpoint. This allows for unidirectional or bi-directional communication.

This is a different concept to brokered messaging as the message is relayed directly to an endpoint without any brokering of the message. Applications establish an outbound connection to the relay and the relay manages the transport of the messages.

Relays are flexible and can help connect applications across complex NAT configurations, this is because the connection to the relay is an outbound connection from the application.

##WCF programming model and Relay bindings

If you want to code for Service Bus Relay, then you will need to understand Windows Communication Foundation. WCF is the programming model used by applications that participate in the relay. WCF has been around for a long time (relatively speaking!) and is a mature, stable messaging framework.

You leverage existing WCF concepts, such as System.ServiceModel, DataContract, DataMembers and so on. Relay applications use custom WCF binding based on existing bindings. 

There are *several bindings* that you can use with Service Bus Relay, identical to System.ServiceModel bindings:

* BasicHttpRelayBinding
	* This corresponds to BasicHttpBinding.

* WebHttpRelayBinding
	* This corresponds to WebHttpBinding.

* WS2007HttpRelayBinding
	* This corresponds to WS2007HttpBinding.

* NetTcpRelayBinding
	* This corresponds to NetTcpBinding.

And there are *two unique* bindings to Service Bus Relay:

* NetOnewayRelayBinding
	* Optimized for one-way messages.
	* All service contract operations must be marked as one-way operations using the IsOneWay attribute.

* NetEventRelayBinding
	* Derives from NetOnewayRelayBinding.
	* Supports multicast of a single message to multiple consumers.
	* Supports registration of multiple WCF services to the same endpoint.
	* When a message is sent, it is broadcasted to all registered WCF services for that endpoint.

##Let's walk through an example....

In this example I am going to run an on-premise SOAP WCF service with TCP transport. The web-client will be hosted on Azure Websites and will invoke a SOAP style RPC (remote procedure call) to the on-premise service via Service Bus.

    [ServiceContract]
    public interface IConsoleService
    {
        [OperationContract]
        void Write(string text);
    }

Note: I am going to use the Shared Assembly approach of WCF (System.ServiceModel.ChannelFactory<T>)

The next step is to create the server... 

You will need to start with bringing in the WindowsAzure.ServiceBus nuget package:
	
	PM> Install-Package WindowsAzure.ServiceBus

Now lets implement IConsoleService:

    class ConsoleService : IConsoleService
    {
        public void Write(string text)
        {
            Console.WriteLine(text); ;
        }
    }

The WCF service needs hosted - I will self host the service:

    class Program
    {
        static void Main(string[] args)
        {
            var host = new ServiceHost(
                typeof(ConsoleService), 
                new Uri("sb://NAMESPACE.servicebus.windows.net")
            );

            var endpoint = host.AddServiceEndpoint(
                typeof(IConsoleService), 
                new NetTcpRelayBinding(), 
                "console"
            );

            endpoint.Behaviors.Add(new TransportClientEndpointBehavior
            {
                TokenProvider = TokenProvider.CreateSharedSecretTokenProvider(
                    issuerName: "ISSUER-NAME", 
                    issuerSecret: "ISSUER-SECRET"
                )
            });

            host.Open();

            Console.WriteLine("The server is running");
            Console.ReadKey();

            host.Close();
        }
    }

Note: you'll need to replace NAMESPACE, ISSUER-NAME and ISSUER-SECRET with your credentials.

Ok, so that's the server implemented! On to the client...

I've gone with an ASP.NET MVC Service Bus client, so I can deploy it to Azure Websites and invoke the on-premise ConsoleWriter service.

This is my ASP.NET MVC actionmethod:

        public ActionResult Write(string text)
        {
            var factory = new ChannelFactory<IConsoleService>(
                new NetTcpRelayBinding(),
                new EndpointAddress("sb://rogertinsley.servicebus.windows.net/console")
            );

            factory.Endpoint.Behaviors.Add(new TransportClientEndpointBehavior
            {
                TokenProvider = TokenProvider.CreateSharedSecretTokenProvider(
                    issuerName: "ISSUER-NAME",
                    issuerSecret: "ISSUER-SECRET"
                )
            });

            var proxy = factory.CreateChannel();
            proxy.Write(text);

            (proxy as IClientChannel).Close();

            return Redirect(Request.ApplicationPath);
        }

It invokes the RPC method called "Write" via the Service Bus. And that's it. Deploy to Azure Websites, enter text and the service bus will relay the text to the console on your local development environment! 

Amazin'.

A working example of this example is available on my [Github](https://github.com/rogertinsley/servicebusrelay).