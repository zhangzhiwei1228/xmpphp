#The philosophy and approach to XMPPHP

# Introduction #

This document introduces XMPP and the approach XMPPHP uses.

# Overview #

XMPP is a socket protocol for messages and presence in XML.  It is particularly useful for eventing services and instant messaging.  XMPP consists of two XML documents that start at the beginning of the connection and end at the end of the connection.  One document is the server communicating with the client, and the other document is the client communicating to the server. They can last for as long or as short of a period as you desire.

## XMLStream ##

In XMPPHP, the main processing is done by XMLStream.php.  This contains a class that could be used to make any streaming XML socket server or client.  It handles class connections, starttls style encryption, sending through the socket, but most importantly handling incoming events. In an XMPP stream, there are stanzas one level into the document that describe different interactions: IQ, Messages, and Presence.  IQ interactions are any back-and-forth style communication, such as information requests, or publishes that require a confirmation.  Messages are stanzas that are routed to a specific entity, and contain payloads that might be an IM message or things a form to fill out, or various other direct communications.  Presence stanzas are broadcast to your entire roster, and generally contain payloads indicating your current status (online, offline, away, dnd) and a brief message related to your status.

XMLStream watches for these stanzas.  Currently in XMPP they can be matched by either the root attribute of "id" or a name/namespace match.  If a match is found, it is sent to the registered function pointer for that match.

XMLStream::addHandler($name, $namespace, $pointer, $obj=null, $depth=1) is used to register a standard handler pointer ($obj being the class object it is in, and $depth being the XML depth you want to match on).
XMLStream::addIdHandler($id, $pointer, $obj=null) is similar, but matches on the attribute "id."

In the future, handlers will broken out into classes, and you will be able to have different types of matching and pointer behaviors.  For now, we have these basic two, which serve the basic features of XMPPHP well enough.

On a match, the pointer is called with the XML that matched.  The function may then interpret the XML and do whatever it needs to with it, including sending replies.  You may also want to spawn an XMLStream event.  Events are created with a name and an array payload.  If there is a handler set for that name, then it will be called.  Events can also interrupt stream processing, which I'll get to in a bit.  The event array is specific for each event.  For example, in XMPPHP the "message" event contains a to, from, body, subject, etc.  XMLStream::addEventHandler($name, $pointer) will register an event handler function, and XMLStream::event($name, $payload) will cause an event.

## Stream Processing ##

You have 3 options when processing an XMLStream.  You may process indefinitely with XMLStream::process(), you may process for a specific number of seconds with XMLStream::processTime($seconds), or you may XMLStream::processUntil(array()/$event).  In XMPPHP, for example, it is particularly useful to processUntil('session'), which is, process the XML until we're authenticated and have an established session.  At that point, you will likely want to retrieve your roster and send out your presence (as in the examples).  You can also processUntil an array of possible events, which means that any of the events will cause processing to stop.  processUntil returns an array of event arrays.  It is often useful to processUntil a list of events in a loop, and then loop through the resulting events and then take appropriate action based on the event given.  You will see an example of this in cli\_longrun\_example.php.

## XMPP Specifics ##

XMPPHP's object init setups up most of your library settings, and ::message ::presence, as well as one offs like ::getRoster() handle the rest.  To see how these functions are used, please view the examples.  I will right a method list in a separate document.