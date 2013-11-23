# webinos App2App messaging API #

**Service Type**: http://webinos.org/api/app2app

App2app API provides a channel based communication for your applications. You can create a channel with a unique namespace and let your applications communicate between personal zones.

Peers can either create a channel with a specified URN or connect to a previously created channel using the specified URN. Peers of a channel then can either broadcast or unicast messages to other connected peers, i.e. either send a private message on a peer to peer basis or broadcast a message to all connected peers.

## Installation ##

To install the app2app API you will need to npm the node module inside the webinos pzp and your pzh.

For end users, you can simply open a command prompt in the root of your webinos-pzp and do: 

	npm install https://github.com/webinos/webinos-api-app2app.git

For developers that want to tweak the API, you should fork this repository and clone your fork inside the node_module of your pzp and your pzh.

	cd node_modules
	git clone https://github.com/<your GitHub account>/webinos-api-app2app.git
	cd webinos-api-app2app
	npm install


## Getting a reference to the service ##

To discover the service you will have to search for the "http://webinos.org/api/app2app" type. Example:

	var serviceType = "http://webinos.org/api/app2app";
	webinos.discovery.findServices( new ServiceType(serviceType), 
		{ 
			onFound: serviceFoundFn, 
			onError: handleErrorFn
		}
	);
	function serviceFoundFn(service){
		// Do something with the service
	};
	function handleErrorFn(error){
		// Notify user
		console.log(error.message);
	}

Alternatively you can use the webinos dashboard to allow the user choose the app2app API to use. Example:
 	
	webinos.dashboard.open({
         module:'explorer',
	     data:{
         	service:[
            	'http://webinos.org/api/app2app'
         	],
            select:"services"
         }
     }).onAction(function successFn(data){
		  if (data.result.length > 0){
			// User selected some services
		  }
	 });

## Methods ##

Once you have a reference to an instance of a service you can use the following methods:

### createChannel (configuration,requestCB, messageCB, successCB, errorCB)

Create a new channel.

The configuration object should contain "namespace", "properties" and optionally "appInfo".

- Namespace is a valid URN which uniquely identifies the channel in the personal zone.
- Properties currently contain "mode" and optionally "canDetach" and "reclaimIfExists".
 - The mode can be either "send-receive" or "receive-only". The "send-receive" mode allows both the channel creator and connected clients to send and receive, while "receive-only" only allows the channel creator to send.
 - If canDetach is true it allows the channel creator to disconnect from the channel without closing the channel.
- appInfo allows a channel creator to attach application-specific information to a channel. The channel creator can decide which clients are allowed to connect to the channel. For each client which wants to connect to the channel the requestCallback is invoked which should return true (if allowed to connect) or false. If no requestCallback is defined all connect requests are granted.

If the channel namespace already exists the error callback is invoked, unless the reclaimIfExists property is set to true, in which case it is considered a reconnect of the channel creator. Reclaiming only succeeds when the request
originates from the same session as the original channel creator. If so, its bindings are refreshed (the mode and
 appInfo of the existing channel are not modified).

### searchForChannels (namespace, zoneIds, searchCB, successCB, errorCB)

 Search for channels with given namespace, within its own personal zone. It returns a proxy to a found channel through the searchCallback function. Only a single search can be active on a peer at the same time, and a search automatically times out after 5 seconds


----------

When you get a reference to a ChannelProxy you can use the following methods:

### connect (requestInfo, messageCB, successCB, errorCB)

Connect to the channel. The connect request is forwarded to the channel creator, which decides if a client is allowed to connect. The client can provide application-specific info with the request through the requestInfo parameter.

### send (message, successCB, errorCB)

Send a message to all connected clients on the channel. 

### sendTo (client, message, successCB, errorCB)

Send to a specific client only. The client object of the channel creator is a property of the channel proxy. The App2App Messaging API does not include a discovery mechanism for clients. A channel creator obtains the client objects for each client through its connectRequestCallback, and if needed the channel creator can implement an application-specific lookup service to other clients. A client object only has meaning within the scope of its channel, not across channels. Note that the client object of a message sender can also be found in the "from" property of the message.

### disconnect (successCB, errorCB)

Disconnect from the channel. After disconnecting the client does no longer receive messages from the channel.
If the channel creator disconnects, the channel is closed and is no longer available. The service does not inform connected clients of the disconnect or closing. If needed the client can send an application-specific message to inform other clients before disconnecting.


## Links ##

- [Specifications](http://dev.webinos.org/specifications/api/app2app.html)
- [Examples](https://github.com/webinos/webinos-api-app2app/wiki/Examples)

