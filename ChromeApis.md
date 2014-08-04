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

`MessageSender` does not contain `url`, only `id` and `tab` (if the sender is a content script).

`sendResponse` must be an object (not a string, array, etc.).

**Please note:** if you wish to call `sendResponse` asynchronously (i.e. after the listener returns), you must return `true` from the event listener function as described in the Chrome documentation.

### sendMessage

    chrome.runtime.sendMessage(string extensionId, any message, function responseCallback)

`extensionId` is accepted but is currently ignored. Messaging works only within one extension.

`message` must be an object.

`responseCallback` is fully supported but the parameter must be an object (see **onMessage.addListener**).

### getURL

Fully supported

## [`chrome.tabs`](http://developer.chrome.com/extensions/tabs.html)

The **tab** object referred to throughout the API as a callback parameter contains only `id`, `url` and `active`.

### query

    chrome.tabs.query(object queryInfo, function callback)

`queryInfo` recognizes only one filtering property: the `active` flag.

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

## [`chrome.contextMenus`](http://developer.chrome.com/extensions/contextMenus.html)

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

- `menuItemId` (always)
- `pageUrl` (always)
- `linkUrl` (if the creation context was `link`)
- `selectionText` (if the creation context was `selection` or `page`)

**Tab** contains only `id`.

### [`chrome.storage`](https://developer.chrome.com/extensions/storage)

**StorageArea** `local` is implemented. `sync` is accepted but falls back to `local` at the moment.

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

### [`chrome.browserAction`](https://developer.chrome.com/extensions/browserAction)

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

### [`chrome.webRequest`](https://developer.chrome.com/extensions/webRequest)

**WARNING:** If you declare any of the following listeners to use blocking behavior, do not use `alert()` for debugging the listener functions. Kitt will display the alert but the UI will deadlock afterwards. This is not a Kitt bug/deficiency, but an unavoidable side effect of the way `UIWebView` executes JavaScript on the UI thread.

### onBeforeRequest.addListener

    chrome.webRequest.onBeforeRequest.addListener(callback, filter, opt_extraInfoSpec);

`callback` receives a `detail` object with all properties [documented for Chrome](http://developer.chrome.com/extensions/webRequest#event-onBeforeRequest) but with the following limitations:

- `frameId` is fixed to 0 as if the request always happens in main frame.
- `parentFrameId` is fixed to -1 as if there was no parent frame.
- `type` is fixed to `main_frame`.
- `requestBody` is null.

The `BlockingResponse` object recognizes the `cancel` and `redirectUrl` parameters

`requestfilter` supports only `urls`.

The `opt_extraInfoSpec` flag `blocking` is fully supported.

### onHeadersReceived.addListener

    chrome.webRequest.onHeadersReceived.addListener(callback, filter, opt_extraInfoSpec);

The `callback` takes a `detail` object with an additional `responseHeaders`property. It has same limitations documented in **onBeforeRequest**.

The `BlockingResponse` object recognizes `cancel`, `redirectUrl` and `responseHeaders`.


### handlerBehaviorChanged

    chrome.webRequest.handlerBehaviorChanged(function callback);

Clears the `UIWebView` cache. As in Chrome, this is an expensive operation and should be used sparingly.

### [`chrome.webNavigation`](https://developer.chrome.com/extensions/webNavigation)

For the `filter` parameter, the `url` array object of type [**events.UrlFilter**](http://developer.chrome.com/extensions/events.html#type-UrlFilter) is fully implemented.

In the absence of authoritative specification, the following behaviors were confirmed by testing against the Chrome browser:

- No filters defined means unrestricted match (matches all).
- Notwithstandign the wording of the spec _"Conditions that the URL ... must satisfy"_, which sounds like an **AND** operation (all conditions must match), it is really an **OR** evaluation (only one condition must match).

### onCreatedNavigationTarget.addListener

    chrome.webNavigation.onCreatedNavigationTarget.addListener(function callback, object filters)

The `callback` parameter `details` defines only `tabId`, `url` and `timeStamp`.

**`sourceTabId` and `sourceFrameId` are not supported yet.**

### onBeforeNavigate.addListener

    chrome.webNavigation.onBeforeNavigate.addListener(function callback, object filters)

The `callback` parameter `details` defines only `tabId`, `url` and `timeStamp`.

**`frameId` and `parentFrameId` are not supported yet.**
