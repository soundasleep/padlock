Groups and Permissions
======================

Once we can pair identities, we need to be able to assign them to groups or hierarchies
so that messages can be published publicly/privately etc. This information is not exposed
to the endpoints API.

Use hierarchical RBAC; groups can contain groups for permissions to be inherited.
There is a system "anonymous" group that all groups inherit from but have no permissions.

/internal/groups
----------------

List all groups and their members.
This interface is also available in the Padlock interface.

{
	'groups': [
		{
			'group': 'anonymous',
			'types': ['twitter.tweet'],
		},
		{
			'group': 'friends',
			'parent': 'anonymous',
			'types': ['chat.message']
		},
		{
			'group': 'http://carl.org',
			'parent': 'friend',
			'types': [],
		},
		{
			'group': 'owner',
			'parent': 'friend',
			'types': ['chat.*']		# all chat.? message types
		}
	],
	'membership': [
		'http://alice.org': 'owner',
		'http://bob.org': 'friend',
		'http://carl.org': 'http://carl.org',
		'anonymous': 'anonymous',
	]
}

/internal/groups/update
-----------------------

Add or update a group.
This interface is also available in the Padlock interface.

Parameters:

{
	'group': 'friend meow',		# unique ID
	'parent': 'friend',
	'types': []
}

/internal/groups/delete
-----------------------

Delete a group and all children groups.
This interface is also available in the Padlock interface

Parameters:

{
	'group': 'friend meow',		# unique ID
	'newParent': 'anonymous',	# set all childen group parents to this group instead
}

/internal/groups/membership
---------------------------

Add or update a group membership.
This interface is also available in the Padlock interface.

Parameters:

{
	'identity': 'http://alice.org',
	'group': 'friend alice'
}

/internal/request
-----------------

Request a pairing with someone else.
This interface is also available in the Padlock interface so maybe this method can be removed.

Parameters:

{
	'to': 'http://dave.org',
	'group': 'friend dave'				# the group that the identity will be added to if paired
}

/internal/requests
------------------

Get all pending pairing requests.

{
	'requests': [
		{
			'to': 'http://dave.org',
			'submitted': '2012-05-05T08:00-0800'
		}
	]
}

/internal/unpair
----------------

Unpair an identity.
This interface is also available in the Padlock interface so maybe this method can be removed.

{
	'to': 'http://bob.org'
}

Internal Messaging
==================

Services can post messages internally. This should not be accessible to the web because
the protocol does not support authentication for internal methods; another service should
instead expose the services through some form of authentication.

/internal/post
--------------

Post a message.

{
	'messages': [
		{
			'type': 'chat.message'
			'date': '2012-01-01T01:00-0800',
			'format': 'text/plain',
			'body': 'Hello, world!'
			'id': '224283659816398111'		# allows messages to be merged etc; a unique (to this server) ID string for this message
			'for': [
				// will post this message to all identities that have the 'chat' permission
				'friends',
				'friend carl',
			]
		},
	]
}

Response:

{
	'success': true,
	'pending': 12		// the total number of messages * identities that will be sent
}
