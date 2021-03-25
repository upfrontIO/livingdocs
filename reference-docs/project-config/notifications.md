# Notfications config
Added in: [`release-2021-03`](https://github.com/livingdocsIO/livingdocs-release-notes/blob/master/releases/release-2021-03.md)

You can configure `notifications` to enable the possibility that a user can get notifications for changes on a document.

```js
module.exports = {
  v: 2,
  notifications: {
    // Define actionGroups which the user can select per document
    actionGroups: [
      // First example group, interested in ALL changes
      {
        handle: 'all',
        label: 'All Events',
        description: 'Comments, Tasks, Publish and delete',
        // add actions on which the user should get a notification
        actions: [
          'task.change',
          'document.publish',
          'document.unpublish',
          'document.copy',
          'document.delete',
          'document.transform',
          'comment.add'
        ]
      }
    ],
    // the task requester is notified on task changed even without subscription
    notifyTaskRequester: true,
    // the author is auto subscribed to the document with the specified actionGroup
    autoSubscribeOwner: {enabled: true, actionGroup: 'all'}
  }
}
```

Possible action to register on at the moment:
- task.change
   config options:
   ```js
    // assign to all tasks and statusChanges
   'task.change'
   
    // assign to task proofreading on statusChanges 'requested', 'accepted', 'completed'
   {type: 'task.change', taskName: 'proofreading', statusChange: ['requested', 'accepted', 'completed']}
   ```
- document.publish
- document.unpublish
- document.copy
- document.delete
- document.transform
- comment.add