Public Sharing
==============

Content can also be shared publicly, by publishing a set of keys publicly.
These can be revoked silently by Alice and the service will have to repair with the new set of public/private keys.

/anonymous
----------

Get the anonymous private and public keys, if supported.

{
	'success': true,
	'publicKey': 'Anonymous.public',
	'privateKey': 'Anonymous.private'
}

OR

{
	'success': false,
	'error': 1,
	'message': 'Anonymous keys are not available here.'
}

Publicly available sites can use a similar method to pairing to connect up to the 
messaging service. e.g. http://jevon.org/twitter uses the Padlock identity 'http://jevon.org/twitter/padlock'
with a public/private key etc to display all posts that are accessible publicly.
