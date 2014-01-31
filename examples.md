Examples
========

This covers a simple scenario of how two identities pair, register for new messages, and get pushed new messages.

### 1. Pairing

Alice: http://alice.org/internal/groups/update (group: friend bob, parent: friends, types: [])		# create new group

Alice: http://alice.org/internal/request (to=http://bob.org, group: friend bob)						# and make a friend request

ServerAlice: Discover Padlock server at http://bob.org/

ServerAlice: http://bob.org/padlock/info					# check protocol version

ServerAlice: http://bob.org/padlock/request (from: http://alice.org, to: http://bob.org, publicKey: Alice.public)
	Response: (success: true, secret: 12345abcde)

ServerBob: Discover Padlock server at http://alice.org/

ServerBob: http://alice.org/padlock/info					# check protocol version

ServerBob: http://alice.org/padlock/contact (from: http://alice.org, to: http://bob.org, secret: 12345abcde)
	Response: (success: true, name: Alice, email: alice@alice.org)

Bob: gets e-mail about pending request, accepts pairing request

ServerBob: http://alice.org/padlock/pair (from: http://alice.org, to: http://bob.org, secret: 12345abcde, publicKey: Bob.public)

ServerAlice adds new identity, adds identity to group

Alice: gets e-mail about successful pair

