Initial pairing
===============

These endpoints are used to initialise the communication between two parties who wish to pair together.

Each party has a private key (Alice.Private and Bob.Private) and associated public key (Alice.public and Bob.public)
which are used to sign and verify communication.

## Unauthenticated endpoints

### /info

List information about the protocol supported, version, etc

Response:

```
{
	'protocol': 1,
	'server': 'Padlock reference implementation',		// implementation-specific
	'version': '0.1',
	'homepage': 'http://openpadlock.org/reference'
}
```

### /request

Request a pairing (from Bob to Alice).

```
{
	'from': 'http://bob.org',
	'to': 'http://alice.org',
	'publicKey': 'Bob.public'
}
```

Response:

```
{
	'success': true,
	'secret': 'e98427t92351'			// to prevent /contact without requests
}
```

OR

```
{
	'success': false,
	'error': 1,
	'message': 'I don't recognise that identity as mine',
}
```

```
{
	'success': false,
	'error': 2,
	'message': 'I am not accepting your identity',
}
```

```
{
	'success': false,
	'error': 3,
	'message': 'I am not accepting pairing requests right now',
}
```

### /pair

Accept a pairing request (from Alice to Bob). Parameters:

```
{
	'from': 'http://alice.org',
	'to': 'http://bob.org',
	'success': true,
	'publicKey': 'Alice.public'
	'secret': 'e98427t92351'
}
```

OR (note pair requests can be ignored forever)

```
{
	'from': 'http://alice.org',
	'to': 'http://bob.org',
	'success': false,
	'error': 1,
	'message': 'I have not accepted your pair request'
}
```

### /contact

Get basic contact information. This information could be exposed to everyone who makes a pairing request.
This information can also be exposed to third parties who snoop the connection.

The request and response is not encrypted because Alice hasn't agreed to Bob's pairing request yet.
The 'from'/'to' identities can be used to restrict requests to those which have requested pairs/etc. Restricting it to
valid pair requests (or send invalid data on invalid pair requests) allows one to reduce exposure.

```
{
	'from': 'http://bob.org',
	'to': 'http://alice.org',
	'secret': 'e98427t92351'
}
```

Response:

```
{
	'success': true,
	'name': 'Jevon Wright',
	'email': 'jevon@jevon.org'		// all fields are optional; providing email allows gravatars to be used
}
```

OR

```
{
	'success': false,
	'error': 1,
	'message': 'I don't recognise that identity as mine',
}
```

```
{
	'success': false,
	'error': 2,
	'message': 'I don't accept your identity',		// also invalid secret
}
```

### /revoked

Get all keys that have been revoked. A revokation "message" is one that has been signed by the private key.
It's safe to let compromised private keys generate new revocation certificates because all they can do is disable
keys. Note that revocation certificates should NOT be stored on the same server as the private/public key, otherwise
an attacker could revoke a certiciate, generate a new private/public key pair and be able to control their identity
forever.

It's assumed that providing these revocation messages are secure because they cannot be verified without
having already received Alice's old public key.

Response:

```
{
	'revoked': [
		{
			'certificate': '<Revocation message signed by Alice.old.private and verified by Alice.public>'
		}, 
		// more certificates here
	]
}
```

As soon as a public key is revoked all payload messages will return Error 3: I could not decrypt your payload,
which will trigger a repairing.

When Alice wants to revoke her public/private key pair:

1. She generates a new pair of public/private keys and signed revocation message.
2. She adds the old revocation message to her /revoked list
3. She is prepared for all her old paired identities to re-request pair requests (through /request and /pair) so that they can receive new public keys.

