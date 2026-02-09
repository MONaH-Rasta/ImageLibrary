# ImageLibrary

Oxide plugin for Rust. Plugin API for managing images.

**Image Library** is a tool that other plugins can utilise to store and manage imagery for use in UI. On its own it serves no real purpose.

## Console Commands

`refreshallimages` - Retrieve's and stores all item icon URLs (include's workshop images). Will also download every icon to file storage if the config option "Images - Only download image's when required" is set to true

`cancelstorage` - This will cancel any pending file downloads

## Configuration

```json
{
  "Avatars - Store player avatars": true,
  "Steam API key (get one here <https://steamcommunity.com/dev/apikey>)": "",
  "Progress - Show download progress in console": true,
  "Progress - Time between update notifications": 20,
  "User Images - Manually define images to be loaded": {},
  "Version": {
    "Major": 2,
    "Minor": 0,
    "Patch": 47
  }
}
```

### Configuration Options

`"Avatars - Store player avatars"`: Downloads player's avatars image's (some plugins may require this)
`"Progress - Show download progress in console"`: This option will show you progress of load orders in console
`"Progress - Time between update notifications"`: The amount of time between progress updates
`"User Images - Manually define images to be loaded"`: User specified images to be downloaded (some plugins may require this)
`"Steam API key (get one here <https://steamcommunity.com/dev/apikey>)"`: This is required to download skin images from the workshop

## Setup Approved and Workshop skin support

For ImageLibrary to be able to access approved and Steam Workshop skin icons you must provide a valid API key. These keys are registered to your Steam account and have a limit of 100,000 API calls per day.

To get your Steam API key visit -> <https://steamcommunity.com/dev/apikey>

Once you have your Steam API key, copy and paste it in to the "Steam API key" entry in your config

## Developer API

```cs
(bool) AddImage(string url, string imageName, ulong imageId, Action callback = null)
// Used to download an individual image using a URL (Does not create a load order)

(bool)AddImageData(string imageName, byte[]array, ulong imageId, Action callback = null)
// Adds the image to file storage using raw image data in a byte[] (Does not create a load order)

(void) ForceFullDownload(string title)
// Can be called to force download every available image (Not recommended)

(string) GetImage(string imageName, ulong imageId = 0, bool returnUrl = false)
// Used to retrieve the ID of a stored image.
// By passing "returnUrl" as true the plugin will return the image's URL if the image has not been stored.
// Useful if you want to display a image from a URL whilst it is being downloaded to storage

(List<ulong>) GetImageList(string name)
// Returns a List of available skin Id's for the requested item

(Dictionary<string, object>) GetSkinInfo(string name, ulong id)
// Returns the skin data for workshop items

(bool) HasImage(string imageName, ulong imageId)
// Returns true if the specified image has been downloaded to storage

(bool) IsReady()
// Returns true if there are no pending load orders

(void) ImportImageList(string title, Dictionary<string, string> imageList, ulong imageId = 0, bool replace = false, Action callback = null)
// Used to create a new load order and download a list of images.
// You can pass a dictionary containing names and URL's, IL will then check which of those images have no yet been stored and will store any that are missing.
// Set your plugin title as the title parameter, imageId can be used to prevent plugins from overwriting your images with the same name, and replace is used to force overwrite existing images with the same name.

(void) ImportImageData(string title, Dictionary<string, byte[]> imageList, ulong imageId = 0, bool replace = false, Action callback = null)
// Same as above but used to mass import raw image data in a byte[] (See LustyMap for example usage)

(void) LoadImageList(string title, List<KeyValuePair<string, ulong>> imageList, Action callback = null)
// Used to load any item icons that are missing from the list you specify. (See ServerRewards for example usage)
```

**Note**: Image importing/loading methods have a optional callback parameter which will invoke a function when the load is complete

## Importing Workshop Skins From Your Plugin

Importing skins from the workshop can be done by calling the "LoadImageList" method.

This method is specifically designed to find and download item icons for your plugin. LoadImageList will sort through the list of item skin ID's you provide it, and any that are not already implemented by the game will be passed to Steams API to try and find the icon.

This method should be called with a callback specified and your plugin should NOT be enabled until that callback has been invoked!

For an example of using this method see ServerRewards

## WTF is a load order?

A load order is a set of images that a plugin requests to be downloaded. For example, in ServerRewards when the UI is being generated it creates a load order that requests every item in the store has its icon downloaded. Any of the item icons that have not yet been stored will be queued for processing in that load order. The user will then be able to view the progress of that load order in console via RCon. This breaks up mega queue lists of items into smaller categories for better optimisation and prevents doubling up on image downloads
