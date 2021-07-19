---
title: Livingdocs Server Events
menus:
  reference-docs:
    parent: Server API
---

The events API is Livingdocs's implementation of the publish/subscribe pattern. This allows you to send messages back and forth between features or to listen for events from the core features. Events are fire and forget thus it is possible that some calls might be lost (e.g. when there is a connection outage). If you need reliable transaction-like hooks, see [the hooks feature]({{< ref "./hooks.md" >}}).


## Requiring events from the li-server

Example:
```js
const config = require('../conf')
const liServer = require('@livingdocs/server')(config)
liServer.events.subscribe('document.update', console.log)
```

## Event firing: `notify`

```js
events.notify(event, message)
```

- event `<String>`
- message `<Object>`

The events.notify() methods emits a named event alongside a message object.

## Event subscription: `subscribe`

```js
events.subscribe(event, listener)
```

- event `<String>`
- listener `<Function>`

The events.subscribe() method adds a listener that gets called each time the event is fired.

## Event unsubscription: `unsubscribe`

```js
events.unsubscribe(event, listener)
```

- event `<String>`
- listener `<Function>`

The events.unsubscribe() method removes the listener that was called each time the event was fired.

## Events unsubscription: `unsubscribeAll`

```js
events.unsubscribeAll()
```

The events.unsubscribeAll() method removes all listeners.

## Event subscribers: `subscribers`

```js
events.subscribers(event)
```

- event `<String>`

The events.subscribers() method lists all the listeners of an event.

## Available Events

The following lists all events, before the comma, the name of the event and behind the comma, the parameters received by a listener.

- user
  - user.create, `(eventName, {user})`
  - user.delete, `(eventName, {user})`
  - user.password.request, `(eventName, {userId, identityId})`
  - user.password.redeem, `(eventName, {userId, identityId})`
  - user.password.change, `(eventName, {id, connectionId, identityId, userId})`
  - user.email.change, `(eventName, identity)`
  - user.login.success, `(eventName, {user})`
  - user.login.fail, `(eventName, {error})`

- document
  - document.update, `(eventName, {user, documentVersion})`
  - document.delete, `(eventName, {user, documentVersion})`
  - document.create, `(eventName, {user, documentVersion})`
  - document.publish, `(eventName, {user, documentVersion, renditions})`
  - document.unpublish, `(eventName, {user, documentVersion})`

- document_list
  - document_list.delete, `(eventName, {user, documentList})`
  - document_list.publish, `(eventName, {user, documentList})`
  - document_list.update, `(eventName), {user, documentList}`
  - document_list.create, `(eventName, {user, documentList})`

- migration
  - migration.prepare, `(eventName, {migration})`
  - migration.accept, `(eventName, {migration})`
  - migration.cancel, `(eventName, {migration})`


- mediaLibraryEntry {{< added-in release-2020-12 >}}
  - mediaLibraryEntry.create, `(eventName, {userId, projectId, mediaLibraryEntry})`
  - mediaLibraryEntry.update, `(eventName, {userId, projectId, id, changes})`
  - mediaLibraryEntry.archive, `(eventName, {userId, projectId, id})`
  - mediaLibraryEntry.revoke, `(eventName, {userId, projectId, mediaLibraryEntry})`

- project
  - project.create, `(eventName, {project})`
  - project.update, `(eventName, {project})`
