---
title: Constants
---

## Conversation types
* `1` "one to one"
* `2` group
* `3` public
* `4` changelog

## Read-only states
* `0` read-write
* `1` read-only

## Listable scope
* `0` participants only
* `1` regular users only, excluding guest app users
* `2` everyone

## Participant types
* `1` owner
* `2` moderator
* `3` user
* `4` guest
* `5` user following a public link
* `6` guest with moderator permissions

## Participant in-call flag
* `0` disconnected
* `1` in-call
* `2` provides audio
* `4` provides video
* `8` uses SIP dial-in

## Participant notification levels
* `0` Default (`1` for one-to-one conversations, `2` for other conversations)
* `1` Always notify
* `2` Notify on mention
* `3` Never notify

## Participant read status privacy
* `0` Read status is public
* `1` Read status is private

## Actor types of chat messages
* `guests` - guest users
* `users` - logged-in users
* `bots` - used by commands (actor-id is the used `/command`) and the changelog conversation (actor-id is `changelog`)

## Webinary lobby states
* `0` no lobby
* `1` lobby for non moderators


## Signaling modes
* `internal` No external signaling server is used
* `external` A single external signaling server is used
* `conversation_cluster` An external signaling server is assigned per conversation
