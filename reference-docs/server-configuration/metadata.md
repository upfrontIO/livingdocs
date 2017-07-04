# Metadata on the Server

## Scope

This documentation assumes that you have setup a channel and design within Livingdocs.
You also know what an environment and a channel config is.

To find out more about metadata on the editor you can read the [editor documentation](../editor-configuration/metadata.md) or study [examples](../../concepts/metadata/metadata-examples.md).

## Introduction

Every document(article, page) can have metadata. A metadata object consist of key-value pairs where the key is unique per document.
A metadata field has a configuration (config + used plugin) and can be configured in the channel config. It's possible to use existing metadata plugins to handle the behavior of the metadata field. It's also possible to create custom plugins.

Example of a metadata object:
```json
{
  "title": "Doctor Who 2",
  "url": "/test/doctor-who",
  "seo": {
    "title": "BBC TV Show - Doctor Who",
    "description": "Best TV show of all times",
    "newsKeywords": "tv bbc",
    "excludeFromSpiders": false
  }
}
```
Metadata can be seen as a public API for a document. Frontends usually access on metadata
and use them for rendering data(f.e. teasers).


## Configuration
A metadata configuration describes the metadata properties. The keys are identical to those of
the metadata object. Each channel must provide it's own configuration.
```js
// Metadata configuration example in the channel config
metadata: {
  title: {
    plugin: 'li-title',
    config: {
      maxLength: 200,
      required: true
    }
  },
  url: {
    plugin: 'li-text'
  },
  seo: {
    plugin: 'li-seo'
  }
}
```

## Plugin

A metadata plugin is a set of predefined aspects of a metadata
property. For every property the configuration references a plugin
to base the configuration on.
There is already a set of reusable metadata plugins which are stored in
`plugins/metadata` on the livingdocs-server.


## Write a Custom Plugin

When the behavior of the standard plugins are not enough, you can write custom plugins.


```js
module.exports = {
  // Plugin name
  // Use the name as plugin value in the configuration
  name: 'customername-pluginname',

  // JSON schema
  // Define the schema of the metadata property
  schema: {
    type: 'string'
  },

  // @param value property value after onPublish has been called.
  // @param config {Object} config property of `metadata configuration`
  // @return If you return `false` or even better a `string` with an explanation
  //   this will be treated as a failed validation. All other return values
  //   count as valid.
  validate: function (value, config) {
    // Example implementation
    if (value.length > 20) {
      return 'The value is too long'
    } else {
      return true
    }
  },

  // The onUpdate event will be called before a document gets stored
  // @param newValue Updated value when updating a document
  // @param oldValue Stored value before the document update
  // @param config {Object} config property of `metadata configuration`
  // @param documentVersion {DocumentVersion}
  onUpdate: function(newValue, oldValue, config, documentVersion) {
    // your implementation
  },

  // The onPublish event will be called before a document gets published
  onPublish: function(newValue, oldValue, config, documentVersion) {
    // your implementation
  },

  // The onUnpublish event will be called before a document gets unpublished
  onUnpublish: function(newValue, oldValue, config, documentVersion) {
    // your implementation
  },

  // DEPRECATED: WILL BE REMOVED AT SOME POINT.
  // USE THE RENDER PIPELINE INSTEAD.
  // The onRender event will be called before a document gets rendered
  onRender: function(newValue, oldValue, config, documentVersion) {
    // your implementation
  },

  // DEPRECATED. USE onUpdate INSTEAD.
  // This is the same as `onUpdate` without documentVersion as parameter
  set: function(newValue, oldValue, config) {
    // your implementation
  }
}
```