---
title: Chat specification
layout: default
---


# Chat specification v0.9.0

This document describes how [Delta Chat](https://delta.chat) and compatible messenger use
email messages to piggyback the needed information while staying compatible to existing MUAs.

- [Outgoing messages](#outgoing-messages)
- [Incoming messages](#incoming-messages)
- [Groups](#groups)
    - [Outgoing group messages](#outgoing-group-messages)
    - [Incoming group messages](#incoming-group-messages)
    - [Add and remove members](#add-and-remove-members)
    - [Change group name](#change-group-name)
    - [Set group image](#set-group-image)
- [Set profile image](#set-profile-image)
- [Miscellaneous](#miscellaneous)
- [Encryption](#encryption)
- [Old header names](#old-header-names)


# Outgoing messages

Messengers MUST add a `Chat-Version: 1.0` header to outgoing messages.
For filtering and smart appearance of the messages in normal MUAs, 
the subject SHOULD start with the characters `Chat:` and SHOULD be an excerpt of the message.
Note, that the subject is normally encoded using the encoded-word mechanism.

    From: sender@domain
    To: rcpt@domain
    Chat-Version: 1.0
    Subject: =?utf-8?Q?Chat=3A?= Hello
    
    Hello world!


# Incoming messages

The `Chat-Version` header MAY be used to detect if a messages comes from a compatible messenger.

The subject MUST NOT be used to detect compatible messengers, groups or whatever. Messenger MAY prefix the subject to the text.
The email-body SHOULD be converted to plain text, full-quotes and similar regions SHOULD be cutted from the text.


# Groups

Groups are chats with usually more than one recipient, each defined by an email-address.
The sender plus the recipients of a group are the group members.

To allow different groups with the same members, groups are identified by a group-id.
The group-id MUST be created only from the characters 0-9, A-Z and a-z.

Groups MUST have a group-name. The group-name is any non-zero-length UTF-8 string.

Groups MAY have a group-image.


# Outgoing groups messages

All group members MUST be added to the `From`/`To` headers. 
The group-id MUST be written to the `Chat-Group-ID` header.
The group-name MUST be written to `Chat-Group-Name` header (the forced presence of this header makes it easier to join a group chat on a second device any time).

To identifiy the group-id on replies from normal MUAs, the group-id MUST also be added to
the message-id of outgoing messages.  The message-id MUST have the 
format `Gr.<group-id>.<unique data>`.

    From: member1@domain
    To: member2@domain, member3@domain
    Chat-Version: 1.0
    Chat-Group-ID: 1234xyZ
    Chat-Group-Name: My Group
    Message-ID: Gr.1234xyZ.0001@domain
    Subject: =?utf-8?Q?Chat=3A?= My =?utf-8?Q?Group=3A?= Hello
    
    Hello group - this group contains three members


# Incoming group messages

The messenger MUST search incoming messgages for the group-id in the following headers: `Chat-Group-ID`,
`Message-ID`, `In-Reply-To` and `References` (in this order).

If the messenger finds a valid and existent group-id, the message SHOULD be assigned to the given group. 
If the messenger finds a valid but not existent group-id, the messenger MAY create a new group.
If no group-id is found, the message MAY be assigned to a normal single-user chat with the email-address given in `From`.


# Add and remove members 

Messenger clients MUST construct the member list from the `From`/`To` headers only on the first group message or if they see a `Chat-Group-Member-Added` or `Chat-Group-Member-Removed` action header.
Both headers MUST have the email-address of the added or removed member as the value.
Messenger clients MUST NOT construct the member list on other group messages (this is to avoid accidentially altered To-lists in normal MUAs; the user
does not expect adding a user to a _message_ will also add him to the _group_ "forever").

The messenger SHOULD send an explicit mail for each added or removed member. 
The body of the message SHOULD contain a localized description about what happend 
and the message SHOULD appear as a message or action from the sender.

    From: member1@domain
    To: member2@domain, member3@domain, member4@domain
    Chat-Version: 1.0
    Chat-Group-ID: 1234xyZ
    Chat-Group-Name: My Group
    Chat-Group-Member-Added: member4@domain
    Message-ID: Gr.1234xyZ.0002@domain
    Subject: =?utf-8?Q?Chat=3A?= My =?utf-8?Q?Group=3A?= Hello
        
    Hello, I've added member4@domain to our group.  Now we have 4 members.

To remove a member: 

    From: member1@domain
    To: member2@domain, member3@domain
    Chat-Version: 1.0
    Chat-Group-ID: 1234xyZ
    Chat-Group-Name: My Group
    Chat-Group-Member-Removed: member4@domain
    Message-ID: Gr.1234xyZ.0003@domain
    Subject: =?utf-8?Q?Chat=3A?= My =?utf-8?Q?Group=3A?= Hello
        
    Hello, I've removed member4@domain from our group.  Now we have 3 members.


# Change group name

To change the group-name, the messenger MUST send a message with the action header `Chat-Group-Name-Changed: 1` to all group members.

The messenger SHOULD send an explicit mail for each name change. 
The body of the message SHOULD contain a localized description about what happend 
and the message SHOULD appear as a message or action from the sender.

    From: member1@domain
    To: member2@domain, member3@domain
    Chat-Version: 1.0
    Chat-Group-ID: 1234xyZ
    Chat-Group-Name: Our Group
    Chat-Group-Name-Changed: 1
    Message-ID: Gr.1234xyZ.0004@domain
    Subject: =?utf-8?Q?Chat=3A?= Our =?utf-8?Q?Group=3A?= Hello
    
    Hello, I've changed the group name from "My Group" to "Our Group".


# Set group image

A group MAY have a group-image. 
To change or set the group-image, the messenger MUST attach an image file to a message and MUST add the header `Chat-Group-Image` with the
value set to the image name.

To remove the group-image, the messenger MUST add the header `Chat-Group-Image: 0`.

The messenger SHOULD send an explicit mail for each group image change.
The body of the message SHOULD contain a localized description about what happend 
and the message SHOULD appear as a message or action from the sender.


    From: member1@domain
    To: member2@domain, member3@domain
    Chat-Version: 1.0
    Chat-Group-ID: 1234xyZ
    Chat-Group-Name: Our Group
    Chat-Group-Image: image.jpg
    Message-ID: Gr.1234xyZ.0005@domain
    Subject: =?utf-8?Q?Chat=3A?= Our =?utf-8?Q?Group=3A?= Hello
    Content-Type: multipart/mixed; boundary="==break=="
    
    --==break==
    Content-Type: text/plain

    Hello, I've changed the group image.
    --==break==
    Content-Type: image/jpeg
    Content-Disposition: attachment; filename="image.jpg"
    
    /9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAYEBQYFBAYGBQYHBw ...
    --==break==--

The image format SHOULD be image/jpeg or image/png.


# Set profile image

A user MAY have a profile-image that MAY be spreaded to his contacts. 
To change or set the profile-image, the messenger MUST attach an image file to a message and MUST add the header `Chat-Profile-Image` with the
value set to the image name.

To remove the profile-image, the messenger MUST add the header `Chat-Profile-Image: 0`.

To spread the image, the messanger MAY send the profile image together with the next mail to a given contact
(to do this only once, the messange has to keep a `profile_image_update_state` somewhere).
Alternatively, the messenger MAY send an explicit mail for each profile-image change to all contacts using a compatible messenger.
The messenger SHOULD NOT send an explicit mail to normal MUAs.

    From: sender@domain
    To: rcpt@domain
    Chat-Version: 1.0
    Chat-Profile-Image: photo.jpg
    Subject: =?utf-8?Q?Chat=3A?= Hello
    Content-Type: multipart/mixed; boundary="==break=="
    
    --==break==
    Content-Type: text/plain

    Hello, I've changed my profile image.
    --==break==
    Content-Type: image/jpeg
    Content-Disposition: attachment; filename="photo.jpg"
    
    AKCgkJi3j4l5kjoldfUAKCgkJi3j4lldfHjgWICwgIEBQYFBA ...
    --==break==--

The image format SHOULD be image/jpeg or image/png. Note that `Chat-Profile-Image` may appear together with all other headers, eg. there may be a
`Chat-Profile-Image` and a `Chat-Group-Image` header in the same message.


# Miscellaneous

Messengers SHOULD use the header `Chat-Predecessor` instead of `In-Reply-To` as
the latter one results in infinite threads on typical MUAs.

Messengers SHOULD add a `Chat-Voice-message: 1` header if an attachted audio file is a voice message.

Messengers MAY add a `Chat-Duration` header to specify the duration of attached audio or video files. 
The value MUST be the duration in milliseconds.
This allows the receiver to show the time without knowing the file format.

    Chat-Predecessor: foo123@domain
    Chat-Voice-Message: 1
    Chat-Duration: 10000

# Encryption

Messages SHOULD be encrypted by the [Autocrypt](https://autocrypt.org) standard using `prefer-encrypt=mutual` by default.

Meta data (at least the subject and all chat-headers) SHOULD be encrypted by the [Memoryhole](http://modernpgp.org/memoryhole/) standard. 
If Memoryhole is not used, the subject of encrypted messages SHOULD be replaced by the string 
`Chat: Encrypted message` where the part after the colon may be localized.


# Old header names

Older versions of Delta Chat use the header names `X-MrMsg` (instead of `Chat-Version`), `X-MrPredecessor`, `X-MrGrpId`, `X-MrGrpName`,
`X-MrRemoveFromGrp`, `X-MrAddToGrp`, `X-MrGrpNameChanged`, `X-MrVoiceMessage` and `X-MrDurationMs`.

For outgoing messages, messenger MAY send the old names together with the new ones.
For incoming messages, messenger MAY recognize the old names but MUST prefer the new ones on conflicts.

