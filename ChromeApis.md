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

**Tab** object returned in ```callback``` has the full set of properties defined by [Chrome API doc](https://developer.chrome.com/extensions/tabs#type-Tab)but some values are not implemented yet or have fixed values due to natural limitations of the mobile environment:

- `windowId` is always 0 (browser has only one "window")
- `higlighted` is always *true*
- `pinned` is always *false*
- `incognito` is not supported, hence always *false*
- `width` and `height` are fixed to 3.5" iPhone size (480x320)
- `sessionId` is equal to `id`

### query

    chrome.tabs.query(object queryInfo, function callback)
    
Callback **Tab** parameter has limitations as documented above.

```queryInfo``` recognizes only one filtering property: `active` flag.

### sendMessage

    chrome.tabs.sendMessage(integer tabId, any message, function responseCallback)
    
```message``` is expected to be an object.

```responseCallback``` can have only object as parameter.

If called with nonexistent ```tabId```, nothing will happen. ```runtime.lastError``` is not implemented.
### update

    chrome.tabs.update(integer tabId, object updateProperties, function callback)
    
`updateProperties` recognizes only `url` property.

### create

    chrome.tabs.create(object createProperties, function callback)
    
`createProperties` recognizes `index`, `url` and `active`.

### remove

    chrome.tabs.remove(integer or array of integer tabIds, function callback)

Fully implemented.


### onCreated.addListener

    chrome.tabs.onCreated.addListener(function(Tab tab) {
      ...
    })

Callback **Tab** parameter has limitations as documented above.

### onUpdated.addListener

    chrome.tabs.onUpdated.addListener(
      function(integer tabId, object changeInfo, Tab tab) {
        ...
     })
   
Callback **Tab** parameter has limitations as documented above.

`changeInfo` does not track `pinned` state since Kitt does not currently support pinned tabs.

### onActivated.addListener

    chrome.tabs.onActivated.addListener(
      function(object activeInfo) {
        ...
    })

`activeInfo` callback object value `windowId` is fixed as documented above.

### onRemoved.addListener


    chrome.tabs.onRemoved.addListener(
      function(integer tabId, object removeInfo) {
        ...
    })

`activeInfo` callback object value `windowId` is fixed as documented above.

`isWindowClosing` is always *false*.

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
- ```linkUrl``` if the menu item **contexts** contains `link`
- ```selectionText``` if the menu item **contexts** contains `selection` or `page`

***Tab*** is complete but somewhat limited as described in `chrome.tabs` section

## [`chrome.storage`](https://developer.chrome.com/extensions/storage)

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

## [`chrome.browserAction`](https://developer.chrome.com/extensions/browserAction)

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

Declarative Web Request API supports a single condition type, [```RequestMatcher```](http://developer.chrome.com/extensions/declarativeWebRequest.html#type-RequestMatcher). This is a Chrome API design, not a Kitt limitation. The matcher criteria object [**events.UrlFilter**](http://developer.chrome.com/extensions/events.html#type-UrlFilter) is fully implemented.

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

## [`chrome.webRequest`](https://developer.chrome.com/extensions/webRequest)

> **WARNING:** if you declare any of the following listeners to blocking behavior, do not use `alert()` for debugging the listener functions! Kitt will display the alert but the UI will deadlock afterwards! This is not a Kitt bug/deficiency, but an unavoidable effect of `UIWebView` main thread only invocability. Main thread is also the UI rendering thread.

The `filter` object supports only `urls`, not `types` or `tabId`.

### onBeforeRequest.addListener

    chrome.webRequest.onBeforeRequest.addListener(
      function(object details) {
        ...
      }, filter, opt_extraInfoSpec);
    
`callback` receives `detail` object with all properties as [documented in Chrome](http://developer.chrome.com/extensions/webRequest#event-onBeforeRequest) except `requestBody` which is not provided yet. Some other values may not always be 100% correct:

- `type` is guessed from request headers, unless the request URL end with a type-specific file extension. In case of insufficient headers _and_ missing URL suffix, the fallback type is **image**
- `frameId` and `parentFrameId` are deduced from the `Referer` request header, if any. Result is always correct when `type` is `"main_frame"`. For subframes, there are edge cases where returned `parent` is different from reference Chrome API values. A request with missing `Referer` is assumed to be originated in `"main_frame"`. 

[`BlockingResponse`](https://developer.chrome.com/extensions/webRequest#type-BlockingResponse) object recognizes `cancel` and `redirectUrl` parameters

`opt_extraInfoSpec` flag `blocking` is fully supported

### onHeadersReceived.addListener

    chrome.webRequest.onHeadersReceived.addListener(
      function(object details) {
        ...
      }, filter, opt_extraInfoSpec);

`callback` receives detail object with the same limitations as documented in **onBeforeRequest**, with addition of `responseHeaders`

`BlockingResponse` object recognizes `cancel`, `redirectUrl` and `responseHeaders`.

> **WARNING:** due to certain technical limitations of iOS, `responseHeaders` are ignored at the moment, i.e. the target webview receives the original response headers, not the modified ones.


### handlerBehaviorChanged

    chrome.webRequest.handlerBehaviorChanged(function callback);

Clears the `UIWebView` cache, so it is expensive. Use reasonably as the Chrome doc suggests.

### [`chrome.webNavigation`](https://developer.chrome.com/extensions/webNavigation)

> Since iOS applications run as a single process, the `processId` parameter has no effect as an output parameter (listener config) and is always set to 0 as an input callback parameter.

For `filter` parameter, the `url` array objects type [**events.UrlFilter**](http://developer.chrome.com/extensions/events.html#type-UrlFilter) is fully implemented.

In the lack of authoritative specification, the following behaviors were confirmed by testing against Chrome browser:

- no filters defined means unrestricted match (matches all)
- contrary to the spec wording _"Conditions that the URL ... must satisfy"_ which sounds like **AND** evaluation (all conditions must match), it is really **OR** evaluation (any condition match is enough)

### onBeforeNavigate.addListener

    chrome.webNavigation.onBeforeNavigate.addListener(
      function(object details) {
        ...
      }, object filters)

The callback `details` parameter `processId` is 0 (see above)

### onCreatedNavigationTarget.addListener

    chrome.webNavigation.onCreatedNavigationTarget.addListener(
      function(object details) {
        ...
      }, object filters)

The callback `details` parameter `sourceProcessId` is 0 (see above)

**`sourceFrameId` is not implemented and fixed to -1.**

### onCompleted.addListener

    chrome.webNavigation.onCompleted.addListener(
      function(object details) {
        ...
      }, object filters)

The callback `details` parameter `processId` is 0 (see above)

### getFrame

    chrome.webNavigation.getFrame(object select,
      function(object details) {
        ...
    })

`select.processId` has no effect (see above)

`details.errorOccurred` is not implemented and is always **false**.

### getAllFrames

    chrome.webNavigation.getAllFrames(object select,
      function(array of object details) {
        ...
    })

`details.processId` is 0 (see above)

`errorOccurred` is not implemented and is always **false**.

## [`chrome.windows`](https://developer.chrome.com/extensions/windows)

> **WORK IN PROGRESS**. Minimal implementation for API coverage. The returned `Window` object is hardcoded as follows. Keep in mind that every iOS application has only one window, so this limitation is technically correct. Some form of "virtual windows" may be introduced in the future.

- `id` : 0
- `focused` : true
- `top, left, width, height` : equals to a 3.5" iPhone fullscreen (480x320) starting at zero coordinate.
- `incognito` : false
- `type` : `"normal"`
- `state` : `"fullscreen"`
- `alwaysOnTop` : false
- `sessionId` : `"KittWindow"`

`getInfo` value `populate` is recognized and the `window.tabs` array is populated with limitations of **Tabs** object documented in the `chrome.tabs` section.

### getAll

    chrome.windows.getAll(object getInfo,
      function(array of Window windows) {
        ...
    })

The `windows` array has always only one element.
    
### getLastFocused

    chrome.windows.getLastFocused(object getInfo,
      function(Window window) {
        ...
    })


### onFocusChanged.addListener

Function provided just for API completeness. The browser application window changes focus only when the whole application changes focus, i.e. goes to background or foreground. Backgrounded application is hibernated, so no code could act on the event. Callback is never called at the moment. May be activated with introduction of "virtual windows" UI concept.

## [`chrome.i18n`](https://developer.chrome.com/extensions/i18n)

> **WORK IN PROGRESS**. Minimal implementation for API coverage. Return value gives back the `messageName` parameter.

    string chrome.i18n.getMessage(string messageName, any substitutions)
