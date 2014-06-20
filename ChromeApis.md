Chrome API compliance
---------------------

This document is a "whitelist" of Chrome API implementation in Kitt. **If something is not mentioned, then it is not implemented.**

```permissions``` as declared in [bundle manifest](./ChromeBundleManifest.md) are not implemented yet, so none of the following APIs requires specifying them.

Kitt implementation complies with the [limitations of content script access](http://developer.chrome.com/extensions/extension.html) to chrome APIs. Specifically, only `runtime.sendMessage`, `runtime.onMessage` and cross-site `XMLHttpRequest` is accessible from content script.

## [`chrome.extension`](http://developer.chrome.com/extensions/extension)

### getURL

Fully supported (equivalent to **chrome.runtime.getURL**)

## [```chrome.runtime```](http://developer.chrome.com/extensions/runtime.html)

### onMessage.addListener
Implemented with the following limitation of callback function:

    chrome.runtime.onMessage.addListener(
      function(any message, MessageSender sender, function sendResponse) {
      ...
    })

```message``` can be only object, i.e. not a string, not an array, etc.

```MessageSender``` does not contain ```url```, only ```id``` and ```tab``` (if sender was content script)

`sendResponse` parameter must be an object, i.e. not a string, not an array, etc.

**Please note:** if you wish to call ```sendResponse``` asynchronously (that is after the listener returns), you must return ```true``` from the event listener function exactly as described in Chrome doc.

### sendMessage

    chrome.runtime.sendMessage(string extensionId, any message, function responseCallback)

```extensionId``` is accepted but does not make any difference. Messaging works only within one extension.

```message``` must be an object, i.e. not a string, not an array, etc.

```responseCallback``` is fully supported but the parameter can be only object (see **onMessage.addListener**)

### getURL

Fully supported

## [```chrome.tabs```](http://developer.chrome.com/extensions/tabs.html)
### query

    chrome.tabs.query(object queryInfo, function callback)
    
```queryInfo``` recognizes only one filtering property: `active` flag.

**Tab** objects returned in ```callback``` contains only `id`, `url` and `active`.
### sendMessage

    chrome.tabs.sendMessage(integer tabId, any message, function responseCallback)
    
```message``` is expected to be an object.

```responseCallback``` can have only object as parameter.

If called with nonexistent ```tabId```, nothing will happen. ```runtime.lastError``` is not implemented.
### update

    chrome.tabs.update(integer tabId, object updateProperties, function callback)
    
`updateProperties` recognizes only `url` property

**Tab** object returned in ```callback``` contains only `id`, `url` and `active`.

## [```chrome.contextMenus```](http://developer.chrome.com/extensions/contextMenus.html)

Kitt extension developer can create 3 types of context menus:

1. `page` browser sharing menu accessible by button in bottom toolbar
2. `link` link sharing menu accessible by long tapping some anchored content in the webpage
3. `selection` text selection context menu

The words are related creation contexts, plus `all` for all possible contexts.

**Note:** `page` context will use `icon` as specified in the manifest. Contrary to Chrome documentation, it will not look preferably for 16px icon, but **43px** for iPhone and **55px** for iPad. If you want the icons to look good in the new iOS7 UI paradigm, provide those sizes already in B&W with proper alpha channel transparency.

Context menu id is implemented as 5-character random-ish string.

### create

    string chrome.contextMenus.create(object createProperties, function callback)

```createProperties``` is taking only the following properties:

- ```id``` : uniqueness is required but not checked (```callback``` and ```lastError``` is not implemented)
- ```title```
- ```contexts``` : as specified at the beginning of the chapter
- ```documentUrlPatterns```
- ```targetUrlPatterns```
- ```enabled```

`createProperties` **does not recognize** the following properties

- ```onclick```: use ```onClicked.addListener```
- ```parentId```: only flat menu structure is possible

### update

    chrome.contextMenus.update(string id, object updateProperties, function callback)

```updateProperties``` is taking the same set of properties as **create**

### remove

    chrome.contextMenus.remove(string menuItemId, function callback)

### removeAll

    chrome.contextMenus.removeAll(function callback)

### onClicked.addListener

    chrome.contextMenus.onClicked.addListener(
      function(OnClickData info, tabs.Tab tab) {
        ...
    })

**OnClickData** contain the following members:

- ```menuItemId``` every time
- ```pageUrl``` every time
- ```linkUrl``` if the menu item context was `link`
- ```selectionText``` if the menu item context was `selection` or `page`

***Tab*** contains only ```id```

### [`chrome.storage`](https://developer.chrome.com/extensions/storage)

**StorageArea** `local` is implemented. `sync` is accepted but falls back to `local` at the moment.

### get

    StorageArea.get(string or array of string or object keys, function callback)

all variants of `keys` are supported
    
### set

    StorageArea.set(object items, function callback)
    
- Simple JSONizable objects will serialize as expected
- `Array` will serialize as expected
- `Date` will serialize as `String` (will retain its value but not come back as `Date`)
- `Regex` will serialize as `{}` (in other words, not stored reconstructibly)

**Do not try to store complex objects or functions**

### remove

    StorageArea.remove(string or array of string keys, function callback)
    
all variants of `keys` are supported

### clear

    StorageArea.clear(function callback)

### [`chrome.browserAction`](https://developer.chrome.com/extensions/browserAction)

Implemented as fullscreen popup over the main browser window.

### onClicked.addListener

    chrome.browserAction.onClicked.addListener(
      function(tabs.Tab tab) {
        ...
    });

Is called as expected but **Listener does not receive any parameters (`tab` object is null)**

Work in progress 

### setIcon

    chrome.browserAction.setIcon(object details, function callback)

only `path` detail supported

### [```chrome.declarativeWebRequest```](http://developer.chrome.com/extensions/declarativeWebRequest.html)

### onRequest.addRules

    chrome.declarativeWebRequest.onRequest.addRules(array of Rule rules, function callback)

```callback``` is not implemented

**Rule** checks only properties ```conditions``` and ```actions``` (does not implement any of the optional properties)

#### ```conditions```

Declarative Web Request API supports a single condition type, [```RequestMatcher```](http://developer.chrome.com/extensions/declarativeWebRequest.html#type-RequestMatcher). This is a Chrome API design, not a Kitt limitation. Currently implemented criteria subset is [**events.UrlFilter**](http://developer.chrome.com/extensions/events.html#type-UrlFilter) with ```hostSuffix```.

    new chrome.declarativeWebRequest.RequestMatcher({
      url : { // type events.UrlFilter
        hostSuffix: 'doubleclick.net'
      }
    })
    
#### ```actions```

Fully implemented subset:
    
    new chrome.declarativeWebRequest.CancelRequest(),
    
    new chrome.declarativeWebRequest.RedirectRequest({
      redirectUrl: 'http://www.google.com'
    }),

    new chrome.declarativeWebRequest.RedirectToEmptyDocument(),

    new chrome.declarativeWebRequest.SendMessageToExtension({
      message: 'Action was taken'
    })
    
### onMessage.addListener


    chrome.declarativeWebRequest.onMessage.addListener(
      function(object details) {
        ...
    })

```callback``` parameter ```details``` contains all properties as [documented in Chrome](https://developer.chrome.com/extensions/declarativeWebRequest#event-onMessage) but with some limitations:

- `stage` currently only "_onBeforeRequest_"
- `frameId` fixed to 0 like if the request happened in main frame
- `parentFrameId` fixed to -1 like if there was no parent frame 
- `type` fixed to "_main_frame_"

### [`chrome.webRequest`](https://developer.chrome.com/extensions/webRequest)

    chrome.webRequest.onBeforeRequest.addListener(callback, filter, opt_extraInfoSpec);
    
`callback` receives detail object with all properties as [documented in Chrome](http://developer.chrome.com/extensions/webRequest#event-onBeforeRequest) but with some limitations:

- `frameId` fixed to 0 like if the request happened in main frame
- `parentFrameId` fixed to -1 like if there was no parent frame 
- `type` fixed to "_main_frame_"
- `requestBody` is null

`callback` response object recognizes `cancel` and `redirectUrl` parameters

`requestfilter` supports only `urls`

`opt_extraInfoSpec` flag `blocking` is fully supported
