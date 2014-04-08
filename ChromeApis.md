Chrome API compliance
---------------------

This document is a "whitelist" of Chrome API implementation in Kitt. **If something is not mentioned, then it is not implemented.**

```permissions``` as declared in [bundle manifest](./ChromeBundleManifest.md) are not implemented yet, so none of the following APIs requires specifying them.

Kitt implementation complies with the [limitations of content script access](http://developer.chrome.com/extensions/extension.html) to chrome APIs. Specifically, only `runtime.sendMessage` and `runtime.onMessage` is accessible from content script currently.

## [```chrome.runtime```](http://developer.chrome.com/extensions/runtime.html)

### onStartup, onInstalled, onSuspend
```addListener``` implemented for API completeness **but does not receive any events**.
### onMessage.addListener
Implemented with the following limitation of callback function:

    chrome.runtime.onMessage.addListener(
      function(any message, MessageSender sender, function sendResponse) {
      ...
    })

```message``` is expected to be an object, i.e. not a string, not an array, etc.

```MessageSender``` does not contain ```url```, only ```id``` and ```tab``` (if sender was content script)

If you wish to call ```sendResponse``` asynchronously (that is outside the scope of callback),
you need to return ```true``` from the event listener exactly as described in Chrome doc.

### sendMessage

    chrome.runtime.sendMessage(string extensionId, any message, function responseCallback)

```extensionId``` is accepted but does not make any difference. Messaging works only within one extension.

```message``` is expected to be an object.

```responseCallback``` is fully supported but will return only objects (see **onMessage.addListener**)

## [```chrome.tabs```](http://developer.chrome.com/extensions/tabs.html)
### query

    chrome.tabs.query(object queryInfo, function callback)
    
```queryInfo``` properties are ignored, which means the call always returns all existing tabs.

**Tab** objects returned in ```callback``` contain only ```id```.
### sendMessage

    chrome.tabs.sendMessage(integer tabId, any message, function responseCallback)
    
```message``` is expected to be an object.

```responseCallback``` will return only objects.

If called with nonexistent ```tabId```, nothing will happen. ```runtime.lastError``` is not implemented.

## [```chrome.contextMenus```](http://developer.chrome.com/extensions/contextMenus.html)
Context menus are attached to iOS long tap.

Context menu id is implemented as 5-character random-ish string.

### create

    string chrome.contextMenus.create(object createProperties, function callback)

```callback``` is not implemented

```createProperties``` is taking only the following properties:

- ```id``` : uniqueness is required but not checked (```callback``` and ```lastError``` is not implemented)
- ```title```
- ```contexts``` can have any strings, the only condition is "_selection_" (see below).
- ```onclick``` **not supported**, use ```onClicked.addListener```
- ```parentId``` **not supported**, only flat menu structure is possible
- ```documentUrlPatterns``` as with [bundle manifest](./ChromeBundleManifest.md), the strings must be regular expressions
- ```targetUrlPatterns``` as with [bundle manifest](./ChromeBundleManifest.md), the strings must be regular expressions
- ```enabled```

**Context "selection" :**

- **defined** = the menu will appear with plain text selection (```UIMenuController```)
- **undefined** = the menu will appear with URL selection (```UIActionSheet```)

### update

    chrome.contextMenus.update(string id, object updateProperties, function callback)

```callback``` is not implemented

```createProperties``` is taking only the following properties:

- ```title```
- ```contexts``` will change menu item appearance per "selection" description above
- ```documentUrlPatterns```
- ```targetUrlPatterns```
- ```enabled```


### remove

    chrome.contextMenus.remove(string menuItemId, function callback)

```callback``` is not implemented

### removeAll

    chrome.contextMenus.removeAll(function callback)

```callback``` is not implemented

### onClicked.addListener

    chrome.contextMenus.onClicked.addListener(
      function(OnClickData info, tabs.Tab tab) {
        ...
    })

**OnClickData** contain the following members:

- ```menuItemId``` every time
- ```pageUrl``` every time
- ```linkUrl``` if the menu item context was **not** ```selection```
- ```selectionText``` if the menu item context was ```selection```

***Tab*** contains only ```id```

### [```chrome.declarativeWebRequest```](http://developer.chrome.com/extensions/declarativeWebRequest.html)	
**Foreword:** iOS is low threadcount, event driven OS. The application developer is being pushed via API design to use as little threading as possible, ideally only one and that being the UI rendering thread. Calls between native code and JavaScriptCore are decoupled via single message queue. Attempt to blocking call from native interception of URL request to JavaScript code and still blocking callback to native code results in a deadlock. Hence, blocking [chrome.webRequest](http://developer.chrome.com/extensions/webRequest.html) API is impossible to implement and a non blocking beta API had to be adopted.

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

```callback``` parameter ```details``` contains:

- ```message```
- ```stage``` currently only "_onBeforeRequest_"
- ```url```
- ```timeStamp```

**Due to certain iOS limitations which weren't worked around yet, ```tabId``` is undefined. So the listener doesn't know in which tab the request occured.**
