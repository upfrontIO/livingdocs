---
title: Image Storage & Delivery Proxy
tags: [guides, integrations]
menus:
  guides:
    parent: Integrations
---

## Integration Guide for your own Image Storage and Image Delivery with an Imgix API

Most customers just use [Image Services]({{< ref "/enterprise/guides/image-services#overview" >}}), because they usually support everything you need. If you want to implement your own service (described in this guide), please first get in touch with us to discuss the advantages/disadvatages.

In the following examples you will see the most simple possible service and configuration to support your own upload, storage and delivery.

In a first step you need to configure your own service in the server-config of Livingdocs. Most of the values in the example below can't be changed, because otherwise the setup does not work anymore.
These values are changeable:
- `images.proxy.url` - your image upload server
- `documents.imageServices.liImageProxy.host` - your livingdocs server (used as proxy for upload/delivery)
- `documents.imageServices.imgix.host` - your image delivery server which must support the imgix query API

```js
// server config
imageProxy: {
  // Enable the proxy for the upload
  enabled: true,
  // Define the used image service for the url computation
  //       at the moment, only 'imgix' is supported
  proxiedImageService: 'imgix'
}

// server config
images: {
  proxy: {
    // define your custom server/service where images are uploaded
    url: 'http://localhost:4444'
  }
},

documents: {
  // info: when imageProxy: enabled, then you must define
  //       selectedImageService: 'liImageProxy'
  selectedImageService: 'liImageProxy',
  imageServices: {
    liImageProxy: {
      host: 'http://localhost:9090', // Livingdocs server adress
      proxyEndpoint: 'api/v1/images',
      preferWebp: true,
      backgroundImage: {
        maxWidth: 2048
      },
      srcSet: {
        defaultWidth: 1024,
        widths: [
          2048,
          1024,
          620,
          320
        ],
        sizes: ['100vw']
      }
    },
    imgix: {
      preferWebp: true,
      // define your custom server/service where images are downloaded
      // the service must support the imgix query API
      host: 'http://localhost:4444'
    }
  }
}
```

## A simple mock server example for image upload and image delivery

```js
// mkdir my-test-server
// cd my-test-server
// npm init
// npm install fastify fastify-multipart uuid
// -- add index.js (see example below)
// node index.js

// index.js
const fastify = require('fastify')()
const fs = require('fs')
const util = require('util')
const { pipeline } = require('stream')
const pump = util.promisify(pipeline)
const os = require('os')
const uuid = require('uuid')

fastify.register(require('fastify-multipart'))

fastify.get('/:image', async function (req, reply) {
  console.log('GET image', req.params.image)
  console.log('GET filepath', `${os.tmpdir()}/${req.params.image}`)

  const image = fs.createReadStream(`${os.tmpdir()}/${req.params.image}`)
  reply.send(image)
})

fastify.post('/', async function (req, reply) {
  const data = await req.file()

  const image = `${uuid.v4()}-${data.filename}`
  const filepath = `${os.tmpdir()}/${image}`
  await pump(data.file, fs.createWriteStream(filepath))

  const response = {
    url: `http://localhost:4444/${image}`,
    width: 1024,
    height: 768,
    size: 100000,
    mime: data.mimetype
  }

  console.log('POST response', response)
  console.log('POST image', image)
  console.log('POST filepath', filepath)


  reply
    .code(200)
    .header('Content-Type', 'application/json; charset=utf-8')
    .send(response)
})

fastify.listen(4444, err => {
  if (err) throw err
  console.log(`server listening on ${fastify.server.address().port}`)
})
```