---
title: Revoke Media
description: Handle the revoke event to complete the removal of media
menu: guides
tags: [guides]
---

The Livingdocs server handles the basic functionality of revoking media - removing the asset(s) from storage, removing the media library entry from the dashboard, and updating the media library entry state.
It is likely that each customer will have their own requirements for additional steps required to complete the revoke process based on their server configuration and editorial processes.
These steps may include:
- clearing caches
- regenerating pages
- replacing the media with a placeholder
- sending an email notification

To be as flexible as possible we have provided a server event and a webhook, both named `mediaLibraryEntry.revoke`, which the downstream server can subscribe to.

## Example

### imgix cache purge

As is common for many content delivery networks (CDN), imgix will cache your source image for up to one year on its own servers.
After the Livingdocs server deletes the asset from storage it could still take days, weeks or even months before the image is removed from the CDN.
In order to avoid this delay it is important to instruct imgix to clear it's cache for the asset and serve the latest version, or in this case a missing/broken image.
Below is an example of simple purge request sent to imgix using the `axios` library and an API key for authorization:

```js
liServer.events.subscribe('mediaLibraryEntry.revoke', async (event, {mediaLibraryEntry}) => {
  // Handle errors for async event listeners, because an unhandled rejection will stop the server process!
  try {
    const imageService = framework.imageServices.get('imgix')
    const url = imageService.getUrl(mediaLibraryEntry.asset.url)
    const response = await axios({
      method: 'post',
      url: 'https://api.imgix.com/api/v1/purge',
      validateStatus: null,
      headers: {Authorization: `Bearer ${process.env.IMGIX_API_KEY}`},
      data: {data: {attributes: {url}, type: 'purges'}}
    })
    if (response.status !== 200) {
      liServer.log.error(`Could not purge cache for media library entry "${mediaLibraryEntry.id}"`)
    }
  } catch (error) {
    liServer.log.error(error)
  }
})
```
