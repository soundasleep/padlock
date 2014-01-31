Services
========

Once two-way encrypted communication has been established, we can now start with service discovery
and actually passing messages (push/pull) between Alice and Bob.

For example, `/service#services`:

```
{
	'from': 'http://bob.org',
	'to': 'http://alice.org',
	'payload': {
		'method': 'services',
		// other arguments go here
	}
}
```

### #services

List all of the available services on this Padlock server. A map of (service endpoint) -> (service identity)

Response:

```
{
	'services': [
		'twitter': 'http://openpadlock.org/services/twitter',
		'chat': 'http://openpadlock.org/services/chat',
		// more go here
	]
}
```

### #chat

Provide information about the 'chat' service.
Chat NS: `http://openpadlock.org/services/chat`

```
{
	'enabled': true,
	'protocol': 1,
	'extensions': ["http://jevon.org/padlock/chat", /* any other extensions namespaces that might add values to the JSON responses? */],
	'history': 86400,		// how many seconds back messages can be accessed per channel for this service; -1 if ALL content can be requested
							// (for example having chat going back 1 year might not make any sense, but it does for blog posts)
}
```

### #pull

Pull all pending messages for Bob's identity from Alice.
Bob can receive each message any number of times, e.g. each message is sent to each device;
this is achieved through the 'channel' parameter to messages.

Parameters:

```
{
	'channels': ['desktop'],
	'since': '2012-01-01T01:00-0800'
}
```

Response:

```
{
	'success': true,
	'from': '2012-01-01T01:00-0800',
	'to': '2012-01-01T02:00-0800',		// this allows for message responses to be chunked into blocks rather than requesting GBs of stuff at once
	'channels': ['desktop'],			// the channels that are included in this batch (particularly used in '#push')
	'messages': [
		{
			'type': 'chat'
			'date': '2012-01-01T01:00-0800',
			'format': 'text/plain',
			'body': 'Hello, world!'
			'id': '224283659816398111'		# allows messages to be merged etc; a unique (to this server) ID string for this message
		},
		{
			'type': 'twitter'
			'date': '2012-01-01T01:00-0800',
			'format': 'text/plain',
			'body': 'I am awesome'
			'id': '224283659816398112'
		},
		// ... more
	]
}
```

OR

```
{
	'success': false,
	'error': 1,
	'message': 'Invalid channel specified.'
}
```

### #subscribe

Subscribe to all new messages of the given type intended for Bob's identity.

Parameters:

```
{
	'types': ['chat'],
	'channels': ['desktop']
}
```

Response:

```
{
	'success': true,
	'period': 640,	// how long the subscription will last (in seconds) before we need to re-subscribe (if necessary)	
}
```

OR

```
{
	'success': false,
	'error': 1,
	'message': 'Cannot add any more channels, subscribe ignored..'
}
```

```
{
	'success': false,
	'error': 2,
	'message': 'You supplied an invalid message type, subscribe ignored.'
}
```

```
{
	'success': false,
	'error': 3,
	'message': 'Subscription is not supported.'
}
```

### #push

Push a batch of messages to Alice's identity from Bob.

Parameters:

```
{
	// (same as #pull response)
}
```

### #channels

Return information about channels currently registered to this receiving identity.

Response:

```
{
	'channels': ['desktop', 'mobile', 'web'],		// the channels currently being tracked
	'maximum': 10			// the maximum number of channels that can be subscribed at any time
}
```

### #drop

Drop a channel and all historical messages.

Parameters:

```
{
	'channel': 'desktop'
}
```

Successful response, OR

```
{
	'success': false,
	'error': 1,
	'message': 'That channel does not exist.'
}
```

### #register

Register new channels. Without registering a channel, no messages will actually be sent. This means that
two servers which don't both support the same services don't have to receive messages that can never be used.

If the channel names already exist, only updates the 'types' and moves 'since' if it is after the last pulled/pushed message.

```
{
	'channels': ['desktop', 'mobile'],
	'since': '2012-01-01T01:00-0800',
	'types': ['twitter.*', 'chat.*', ...]			// so that servers don't receive messages they can't process
}
```

Successful response, OR

```
{
	'success': false,
	'error': 1,
	'message': 'Cannot add any more channels; bailed on adding any channels.'
}
```