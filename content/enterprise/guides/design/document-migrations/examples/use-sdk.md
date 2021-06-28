---
title: "Example: Use SDK within a data migration"
linkTitle: Use SDK within a data migration
tags: [guides, maintenance]
menus:
  guides:
    parent: Livingdocs Data Migrations
---

If you need more data for a data migration, it's sometimes necessary to require the Livingdocs Server in a migration.

In this example we will first require and then initialise the Livingdocs Server. After this is done, you can continue with your migrations and change the needed data.


```js
const liSDK = require('@livingdocs/node-sdk')
const liServer = require('../../example-server/runtime_config')
let initialized

module.exports = {
  async migrateAsync ({serializedLivingdoc, metadata, systemdata}) {
    if (!initialized) {
      await liServer.initialize()
      initialized = true
    }

    // load the document (to get the projectId)
    const documentApi = liServer.features.api('li-documents').document
    const documentVersion = await documentApi.getLatestDocument(systemdata.document_id)

    // load the design design
    const designLoader = liServer.features.api('li-design-loader')
    const design = await designLoader.load({
      ...serializedLivingdoc.design, projectId: documentVersion.projectId
    })

    // Make your changes on the document with the livingdoc instance
    const livingdoc = liSDK.document.create({
      content: serializedLivingdoc.content,
      design
    })
    // ...

    return {serializedLivingdoc: livingdoc.serialize(), metadata}
  }
}
```
