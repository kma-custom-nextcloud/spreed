# Participant API

* Base endpoint for API v1 is: `/ocs/v2.php/apps/spreed/api/v1`
* Base endpoint for API v2 is: `/ocs/v2.php/apps/spreed/api/v2`
* Base endpoint for API v3 is: `/ocs/v2.php/apps/spreed/api/v3`

## Get list of participants in a conversation

* Method: `GET`
* Endpoint: `/room/{token}/participants`
* Data:

    field | type | Description
    ------|------|------------
    `includeStatus` | bool | Whether the user status information also needs to be loaded

* Response:
    - Status code:
        + `200 OK`
        + `403 Forbidden` When the participant is a guest
        + `404 Not Found` When the conversation could not be found for the participant
        + `412 Precondition Failed` When the lobby is active and the user is not a moderator

    - Data:
        Array of participants, each participant has at least:

        field | type | API | Description
        ------|------|-----|------------
        `userId` | string | v1 + v2| Is empty for guests
        `attendeeId` | int | v3 | Unique attendee id
        `actorType` | string | v3 | Currently known `users|guests|emails|groups`
        `actorId` | string | v3 | The unique identifier for the given actor type
        `displayName` | string | | Can be empty for guests
        `participantType` | int | | Permissions level of the participant
        `lastPing` | int | | Timestamp of the last ping of the user (should be used for sorting)
        `sessionId` | string | | `'0'` if not connected, otherwise a 512 character long string
        `status` | string | | Optional: Only available with `includeStatus=true` and for users with a set status
        `statusIcon` | string | | Optional: Only available with `includeStatus=true` and for users with a set status
        `statusMessage` | string | | Optional: Only available with `includeStatus=true` and for users with a set status

## Add a participant to a conversation

* Method: `POST`
* Endpoint: `/room/{token}/participants`
* Data:

    field | type | Description
    ------|------|------------
    `newParticipant` | string | User, group, email or circle to add
    `source` | string | Source of the participant(s) as returned by the autocomplete suggestion endpoint (default is `users`)

* Response:
    - Status code:
        + `200 OK`
        + `400 Bad Request` When the source type is unknown, currently `users`, `groups`, `emails` are supported. `circles` are supported with `circles-support` capability
        + `400 Bad Request` When the conversation is a one-to-one conversation or a conversation to request a password for a share
        + `403 Forbidden` When the current user is not a moderator or owner
        + `404 Not Found` When the conversation could not be found for the participant
        + `404 Not Found` When the user or group to add could not be found

    - Data:

        field | type | Description
        ------|------|------------
        `type` | int | In case the conversation type changed, the new value is returned

## Delete an attendee by id from a conversation

* API: Only `v3` or later
* Method: `DELETE`
* Endpoint: `/room/{token}/attendees`
* Data:

    field | type | Description
    ------|------|------------
    `attendeeId` | int | The participant to delete

* Response:
    - Status code:
        + `200 OK`
        + `400 Bad Request` When the participant is a moderator or owner
        + `400 Bad Request` When there are no other moderators or owners left
        + `403 Forbidden` When the current user is not a moderator or owner
        + `403 Forbidden` When the participant to remove is an owner
        + `404 Not Found` When the conversation could not be found for the participant
        + `404 Not Found` When the participant to remove could not be found

## Delete a participant from a conversation

* API: Only `v1` and `v2`
* Method: `DELETE`
* Endpoint: `/room/{token}/participants`
* Data:

    field | type | Description
    ------|------|------------
    `participant` | string | User to remove

* Response:
    - Status code:
        + `200 OK`
        + `400 Bad Request` When the participant is a moderator or owner
        + `400 Bad Request` When there are no other moderators or owners left
        + `403 Forbidden` When the current user is not a moderator or owner
        + `403 Forbidden` When the participant to remove is an owner
        + `404 Not Found` When the conversation could not be found for the participant
        + `404 Not Found` When the participant to remove could not be found

## Remove yourself from a conversation

* Method: `DELETE`
* Endpoint: `/room/{token}/participants/self`

* Response:
    - Status code:
        + `200 OK`
        + `400 Bad Request` When the participant is a moderator or owner and there are no other moderators or owners left.
        + `404 Not Found` When the conversation could not be found for the participant

## Remove a guest from a conversation

* API: Only `v1` and `v2`
* Method: `DELETE`
* Endpoint: `/room/{token}/participants/guests`
* Data:

    field | type | Description
    ------|------|------------
    `participant` | string | Session ID of the guest to remove

* Response:
    - Status code:
        + `200 OK`
        + `400 Bad Request` When the target participant is not a guest
        + `403 Forbidden` When the current user is not a moderator or owner
        + `404 Not Found` When the conversation could not be found for the participant
        + `404 Not Found` When the target participant could not be found

## Join a conversation (available for call and chat)

* Method: `POST`
* Endpoint: `/room/{token}/participants/active`
* Data:

    field | type | Description
    ------|------|------------
    `password` | string | Optional: Password is only required for users which are of type `4` or `5` and only when the conversation has `hasPassword` set to true.
    `force` | bool | If set to `false` and the user has an active session already a `409 Conflict` will be returned (Default: true - to keep the old behaviour)

* Response:
    - Status code:
        + `200 OK`
        + `403 Forbidden` When the password is required and didn't match
        + `404 Not Found` When the conversation could not be found for the participant
        + `409 Conflict` When the user already has an active session in the conversation. The suggested behaviour is to ask the user whether they want to kill the old session and force join unless the last ping is older than 60 seconds or older than 40 seconds when the conflicting session is not marked as in a call.

    - Data in case of `200 OK`:

        field | type | Description
        ------|------|------------
        `sessionId` | string | 512 character long string

    - Data in case of `409 Conflict`:

        field | type | Description
        ------|------|------------
        `sessionId` | string | 512 character long string
        `inCall` | int | Flags whether the conflicting session is in a potential call
        `lastPing` | int | Timestamp of the last ping of the conflicting session

## Resend participant emails

* Method: `POST`
* Endpoint: `/room/{token}/participants/resend-invitations`
* Data:

    field | type | Description
    ------|------|------------
    `attendeeId` | int or null | v3 | Attendee id can be used for guests and users

* Response:
    - Status code:
        + `200 OK`
        + `403 Forbidden` When the current user is not a moderator or owner
        + `404 Not Found` When the given attendee was not found in the conversation

## Leave a conversation (not available for call and chat anymore)

* Method: `DELETE`
* Endpoint: `/room/{token}/participants/active`

* Response:
    - Status code:
        + `200 OK`
        + `404 Not Found` When the conversation could not be found for the participant

## Promote a user or guest to moderator

* Method: `POST`
* Endpoint: `/room/{token}/moderators`
* Data:

    field | type | API | Description
    ------|------|-----|------------
    `participant` | string or null | v1 + v2 | User to demote
    `sessionId` | string or null | v1 + v2 | Guest session to demote
    `attendeeId` | int or null | v3 | Attendee id can be used for guests and users

* Response:
    - Status code:
        + `200 OK`
        + `400 Bad Request` When the participant to promote is not a normal user (type `3`) or normal guest (type `4`)
        + `403 Forbidden` When the current user is not a moderator or owner
        + `403 Forbidden` When the participant to remove is an owner
        + `404 Not Found` When the conversation could not be found for the participant
        + `404 Not Found` When the participant to remove could not be found

## Demote a moderator to user or guest

* Method: `DELETE`
* Endpoint: `/room/{token}/moderators`
* Data:

    field | type | API | Description
    ------|------|-----|------------
    `participant` | string or null | v1 + v2 | User to demote
    `sessionId` | string or null | v1 + v2 | Guest session to demote
    `attendeeId` | int or null | v3 | Attendee id can be used for guests and users

* Response:
    - Status code:
        + `200 OK`
        + `400 Bad Request` When the participant to demote is not a moderator (type `2`) or guest moderator (type `6`)
        + `403 Forbidden` When the current participant is not a moderator or owner
        + `403 Forbidden` When the current participant tries to demote themselves
        + `404 Not Found` When the conversation could not be found for the participant
        + `404 Not Found` When the participant to demote could not be found

## Get a participant by their pin

Note: This is only allowed with validate SIP bridge requests

* API: Only `v3` or later
* Method: `GET`
* Endpoint: `/room/{token}/pin/{pin}`

* Response:
    - Status code:
        + `200 OK`
        + `401 Unauthorized` When the validation as SIP bridge failed
        + `404 Not Found` When the conversation or participant could not be found

    - Data: See array definition in `Get user´s conversations`

## Set display name as a guest

* Method: `POST`
* Endpoint: `/guest/{token}/name`
* Data:

    field | type | Description
    ------|------|------------
    `displayName` | string | The new display name

* Response:
    - Status code:
        + `200 OK`
        + `403 Forbidden` When the current user is not a guest
        + `404 Not Found` When the conversation could not be found for the participant
