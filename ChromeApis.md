Chrome API compliance
---------------------

This document is a "whitelist" of the Chrome API implementation in Kitt. **If something is not mentioned, it is not implemented.**

`permissions` as declared in the [bundle manifest](./ChromeBundleManifest.md) are not implemented yet, so none of the following APIs requires specifying them.

The Kitt implementation complies with the [limitations of content script access](http://developer.chrome.com/extensions/extension.html) to chrome APIs. Specifically, only `runtime.sendMessage`, `runtime.onMessage` and cross-site `XMLHttpRequest` are accessible from content scripts.

## [`chrome.extension`](http://developer.chrome.com/extensions/extension)

### getURL

Fully supported (equivalent to **chrome.runtime.getURL**).

## [`chrome.runtime`](http://developer.chrome.com/extensions/runtime.html)

### onMessage.addListener

Implemented with the following limitations:

    chrome.runtime.onMessage.addListener(
      function(any message, MessageSender sender, function sendResponse) {
      ...
    })

`message` can only be an object (not a string, array, etc.).

`sendResponse` parameter must be an object, i.e. not a string, not an array, etc.

**Please note:** if you wish to call `sendResponse` asynchronously (i.e. after the listener returns), you must return `true` from the event listener function as described in the Chrome documentation.

### sendMessage

    chrome.runtime.sendMessage(string extensionId, any message, function responseCallback)

`extensionId` is accepted but is currently ignored. Messaging works only within one extension.

`message` must be an object.

`responseCallback` is fully supported but the parameter must be an object (see **onMessage.addListener**).

### getURL

Fully supported

## [`chrome.tabs`](http://developer.chrome.com/extensions/tabs.html)

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

`message` is expected to be an object.

`responseCallback` can have only object as a parameter.

If called with a nonexistent `tabId`, nothing will happen. `runtime.lastError` is not implemented.

### update

    chrome.tabs.update(integer tabId, object updateProperties, function callback)

`updateProperties` recognizes only the `url` property.

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

Kitt extension developers can create 3 types of context menus:

1. `page`: browser sharing menu (Activity View) accessible by button in bottom toolbar
2. `link`: link sharing menu (Activity Sheet) accessible by long tapping some anchored content in the webpage
3. `selection`: text selection context menu (Edit Menu)

The creation context `all` will add the menu to all contexts.

**Note:** The `page` context will use the `icon` as specified in the manifest. Optional icon sizes are **43px** for iPhone and **55px** for iPad. If you want the icons to look good under iOS 7, these icons should be black-and-white with the proper alpha channel transparency.

The context menu ID is implemented as 5-character pseudorandom string.

### create

    string chrome.contextMenus.create(object createProperties, function callback)

`createProperties` takes only the following properties:

- `id` (uniqueness is required but not checked, i.e. `callback` and `lastError` are not implemented)
- `title`
- `contexts` (as specified above)
- `documentUrlPatterns`
- `targetUrlPatterns`
- `enabled`

`createProperties` **does not recognize** the following properties:

- `onclick` (use `onClicked.addListener`)
- `parentId` (only flat menu structure is possible)

### update

    chrome.contextMenus.update(string id, object updateProperties, function callback)

`updateProperties` takes the same properties as **create**

### remove

    chrome.contextMenus.remove(string menuItemId, function callback)

### removeAll

    chrome.contextMenus.removeAll(function callback)

### onClicked.addListener

    chrome.contextMenus.onClicked.addListener(
      function(OnClickData info, tabs.Tab tab) {
        ...
    })

**OnClickData** contains the following members:

- ```menuItemId``` every time
- ```pageUrl``` every time
- ```linkUrl``` if the menu item **contexts** contains `link`
- ```selectionText``` if the menu item **contexts** contains `selection` or `page`

***Tab*** is complete but somewhat limited as described in `chrome.tabs` section

## [`chrome.storage`](https://developer.chrome.com/extensions/storage)

**StorageArea** `local` is implemented. `sync` is accepted but falls back to `local` at the moment.

For performance reasons, data stored via chrome.storage is cached. Cached data can only be retrieved from the context in which it was stored. It is therefore recommended that all storage be done in the background script and retrieved in other contexts using chrome.runtime.sendMessage.

### get

    StorageArea.get(string or array of string or object keys, function callback)

All variants of `keys` are supported.

### set

    StorageArea.set(object items, function callback)

- Simple JSONizable objects are serialized as expected.
- `Array` is serialized as expected.
- `Date` is serialized as `String` (i.e. it will retain its value but not deserialize as a `Date`).
- `Regex` is serialized as `{}` (i.e. it is not currently stored reconstructibly).

**Do not try to store complex objects or functions.**

### remove

    StorageArea.remove(string or array of string keys, function callback)

All variants of `keys` are supported.

### clear

    StorageArea.clear(function callback)

## [`chrome.browserAction`](https://developer.chrome.com/extensions/browserAction)

Implemented as a fullscreen popup over the main browser window.

### onClicked.addListener

    chrome.browserAction.onClicked.addListener(
      function(tabs.Tab tab) {
        ...
    });

Called as expected but **listener does not receive any parameters** (`tab` object is null).

### setIcon

    chrome.browserAction.setIcon(object details, function callback)

Only `path` detail is supported.

### [`chrome.declarativeWebRequest`](http://developer.chrome.com/extensions/declarativeWebRequest.html)

### onRequest.addRules

    chrome.declarativeWebRequest.onRequest.addRules(array of Rule rules, function callback)

`callback` is not implemented.

**Rule** checks only `conditions` and `actions` (i.e. it does not implement any of the optional properties).

#### `conditions`

The Declarative Web Request API supports a single condition type, [`RequestMatcher`](http://developer.chrome.com/extensions/declarativeWebRequest.html#type-RequestMatcher). This is specified by the Chrome API, not a Kitt limitation. The matcher criteria object [**events.UrlFilter**](http://developer.chrome.com/extensions/events.html#type-UrlFilter) is fully implemented.

    new chrome.declarativeWebRequest.RequestMatcher({
      url : { // type events.UrlFilter
        hostSuffix: 'doubleclick.net'
      }
    })

#### `actions`

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

The `callback` parameter `details` contains all properties [documented for Chrome](https://developer.chrome.com/extensions/declarativeWebRequest#event-onMessage) but with the following limitations:

- `stage` currently only `onBeforeRequest`.
- `frameId` fixed to 0 as if the request always happens in main frame.
- `parentFrameId` fixed to -1 as if there was no parent frame.
- `type` fixed to `main_frame`.

## [`chrome.webRequest`](https://developer.chrome.com/extensions/webRequest)

**WARNING:** If you declare any of the following listeners to use blocking behavior, do not use `alert()` for debugging the listener functions. Kitt will display the alert but the UI will deadlock afterwards. This is not a Kitt bug/deficiency, but an unavoidable side effect of the way `UIWebView` executes JavaScript on the UI thread.

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

The `opt_extraInfoSpec` flag `blocking` is fully supported.

### onHeadersReceived.addListener

    chrome.webRequest.onHeadersReceived.addListener(
      function(object details) {
        ...
      }, filter, opt_extraInfoSpec);

The `callback` takes a `detail` object with an additional `responseHeaders`property. It has same limitations documented in **onBeforeRequest**.

`BlockingResponse` object recognizes `cancel`, `redirectUrl` and `responseHeaders`.

> **WARNING:** due to certain technical limitations of iOS, `responseHeaders` are ignored at the moment, i.e. the target webview receives the original response headers, not the modified ones.


### handlerBehaviorChanged

    chrome.webRequest.handlerBehaviorChanged(function callback);

Clears the `UIWebView` cache. As in Chrome, this is an expensive operation and should be used sparingly.

### [`chrome.webNavigation`](https://developer.chrome.com/extensions/webNavigation)

> Since iOS applications run as a single process, the `processId` parameter has no effect as an output parameter (listener config) and is always set to 0 as an input callback parameter.

For `filter` parameter, the `url` array objects type [**events.UrlFilter**](http://developer.chrome.com/extensions/events.html#type-UrlFilter) is fully implemented.

In the absence of authoritative specification, the following behaviors were confirmed by testing against the Chrome browser:

- No filters defined means unrestricted match (matches all).
- Notwithstandign the wording of the spec _"Conditions that the URL ... must satisfy"_, which sounds like an **AND** operation (all conditions must match), it is really an **OR** evaluation (only one condition must match).

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
