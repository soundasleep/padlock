Encrypted endpoints
===================

These endpoints require Alice to accept Bob's public key and use it to encrypt with Alice's private key, 
and require Bob to accept Alice's public key, so Bob can decrypt
the response with Alice's public key + Bob's private key.

### /service

Request an encrypted service.

```
{
	'from': 'http://bob.org',
	'to': 'http://alice.org',
	'payload': <encrypted JSON using Bob.private + Alice.public, decrypt with Alice.private + Bob.public>
}
```

Payload:

```
{
	'method': '<method name>',
	// other arguments here
}
```

Response:

```
{
	'success': true,
	'payload': <encrypted JSON using Alice.private + Bob.public, decrypt with Bob.private + Alice.public>
}
```

OR

```
{
	'success': false,
	'error': 1,
	'message': 'I don't recognise that identity as mine.'
}
```

```
{
	'success': false,
	'error': 2,
	'message': 'I don't accept your identity.'
}
```

```
{
	'success': false,
	'error': 3,
	'message': 'I could not decrypt your payload.' // this can happen if Alice's public key has been revoked
}
```

```
{
	'success': false,
	'error': 4,
	'message': 'You are requesting too many things, you are throttled - chill out and wait a bit.'
}
```
