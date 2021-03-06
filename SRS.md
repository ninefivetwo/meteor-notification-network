# Software Requirement Specification

## Objective

To gain an understanding and hands on experience of building a client-server system using Meteor.js by building a simple system that manages and distributes JSON objects.

## Introduction

The aim is to develop a basic version of Twitter that alerts different types/levels of clients with notifications. The goal is to be able to handle many users that may be subscribed to follow select alerts from 100 or more systems that are able to send alerts.

## Components

### Notification Systems

Notification systems represent maintained and service related systems that generate notifications. Some alerts are generated by humans and may occur irregularly and with mistakes. The notifications systems use JSON objects to transfer data over the network. A JSON notification must have the following attributes:

* sender - A descriptive name for the source of the notification
* location - A descriptive name for the location of the notification
* message - Text describing what happened
* timestamp - A time and date of when the notification occurred
* severity - A tag with options of notice, caution or urgent to indicate the severity

You may add extra attributes as needed.

Notifications systems will communicate to the server with a notification when an event has occurred. Notifications generated by humans need to be able to be updated when mistakes are made (i.e. typos). Updates must be idempotent. There is no limit on the number or frequency of notifications a notifications system can send. A notification system will not be a subscriber of alerts.

### Users

Users will only subscribe to alerts that they wish to be notified about. There is no limit to the number of subscriptions a user may follow. However, the notifications must arrive in it's original ordering. When a user subscribes it will receive all available alerts for the subscription. Any previous notifications must be received in the order they were generated as well as any new notifications. Due to high traffic predictions, users will receive of notifications from the server based on the following timers:

* notice - Every 30 minutes
* caution - Every 1 minute
* urgent - As fast as possible

These timers are separate for every user. To clarify, a user will only receive notice notifications every 30 minutes. The timer resets if the client disconnects and reconnects.

### Server

The server sends, receives and stores all notifications in the system. The server is required to send notifications it receives to users who are subscribed to the respective notification system(s). All notifications must be stored or sent in the order they were received in. Therefore all the notifications in your system must be capable of maintaining a synchronized clock. If a "push" notification from a notifications system, followed by a "fetch" request made by a user, followed by another "push" notification from a notifications system, the user should receive the first notification and not the second one because of the time at which the requests we made.

Notifications are not stored forever on the server as they will cause the file system to bloat. Notifications will expire and should be deleted based on the following rules:

* notice - 100 notifications
* caution - 500 notifications
* urgent - 1000 notifications

## Requirements

### Other Functional Requirements

You will also need to including a failure management system to deal with as many of the possible failure modes that you can think of for this problem. This obviously includes client, server and network failure, but now you must deal with the following additional constraints (come back to these constraints after you read the description below):

* Multiple clients may attempt to fetch simultaneously and are required to fetch the notifications that is correct for the synchronized clock adjusted time if interleaved with anything else. Hence, if a push, a fetch and another push arrive in that sequence then the first push must be applied and the content server advised, then the fetch returns the updated feed to the client then the next push is applied. In each case, the participants will be guaranteed that this order is maintained if they are using synchronized clocks.

* Multiple notification systems may attempt to simultaneously push notifications. This must be serialized and the order maintained by synchronized clock timestamp.

* All elements must be capable of implementing synchronized clocks, for synchronization and coordination purposes.

### Non-functional Requirements

Your clients are expected to have a thorough failure handling mechanism where they behave predictably in the face of failure, maintain consistency, are not prone to race conditions and recover reliably and predictably.

Messages must be sent over the network and in JSON format. The system must be designed such that JSON objects to and from other servers/agents should work correctly when communicating over a network.