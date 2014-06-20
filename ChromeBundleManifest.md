Chrome bundle manifest compliance
---------------------------------

This is a complete list of Chrome manifest features duplicated from

[http://developer.chrome.com/extensions/manifest.html](http://developer.chrome.com/extensions/manifest.html)

and modified to mark Kitt implementation specifics and deficiencies. "_KITT: unsupported_" means that the configuration is allowed and parsed, but ignored: there is no logic which would use it.

<pre>
<em><strong>// Required</strong></em>
"<a href="http://developer.chrome.com/extensions/manifest/manifest_version.html">manifest_version</a>": 2,
"<a href="http://developer.chrome.com/extensions/manifest/version.html">version</a>": "versionString",
"<a href="http://developer.chrome.com/extensions/manifest/name.html">name</a>": "MyExtension", <em>// KITT: the recommended form is "my-extension-name" or "MyExtensionName". No national characters and <strong>no spaces</strong> please. For historical reasons, name is used in places where such characters would break things. This is a top priority issue.

<em><strong>// Recommended</strong></em>
"<a href="http://developer.chrome.com/extensions/manifest/description.html">description</a>": "A plain text description",
"<a href="http://developer.chrome.com/extensions/manifest/icons.html">icons</a>": {...},
<em>// KITT:unsupported</em> "default_locale"
  
<em><strong>// Pick one (or none)</strong></em>
"<a href="http://developer.chrome.com/extensions/browserAction.html">browser_action</a>": {
  "default_icon": {...},
    <em>// KITT:unsupported</em> "default_title"
    "default_popup": "popup.html"
},
<em>// KITT:unsupported</em> "page_action"

<em><strong>// Optional</strong></em>
"author": ...,
"<a href="http://developer.chrome.com/extensions/background_pages.html">background</a>": {
  "<a href="http://developer.chrome.com/extensions/event_pages.html">persistent</a>": // KITT: only false
  "scripts": ["background.js"],
  <em>// KITT:unsupported</em> "page"
},
<em>// KITT:unsupported</em> "background_page": ...,
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/override.html">chrome_url_overrides</a>": {...},
<em>// KITT:unsupported</em> "commands": ...,
<em>// KITT:unsupported</em> "content_pack": ...,
"<a href="http://developer.chrome.com/extensions/content_scripts.html">content_scripts</a>": [
  {
    "matches" : ["<all_urls>"],
    "exclude_matches" : ["http://www.google.com/*"],
    <em>// KITT:unsupported</em> "css" : [],
    "js": ["jquery.js", "myscript.js"]  
    <em>// KITT:unsupported</em> "run_at" <em>// KITT:defaults to "document_end"</em>
    <em>// KITT:unsupported</em> "include_globs"
    <em>// KITT:unsupported</em> "exclude_globs"  
  },
  // KITT: only first object in content_scripts will be taken
],
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/contentSecurityPolicy.html">content_security_policy</a>": "policyString",
<em>// KITT:unsupported</em> "converted_from_user_script": ...,
<em>// KITT:unsupported</em> "current_locale": ...,
<em>// KITT:unsupported</em> "devtools_page": ...,
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/messaging.html#external-webpage">externally_connectable</a>": {},
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/fileBrowserHandler.html">file_browser_handlers</a>": [...],
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/manifest/homepage_url.html">homepage_url</a>": "http://path/to/homepage",
<em>// KITT:unsupported</em> "import": ...,
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/manifest/incognito.html">incognito</a>": "spanning or split",
<em>// KITT:unsupported</em> "input_components": ...,
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/manifest/key.html">key</a>": "publicKey",
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/manifest/minimum_chrome_version.html">minimum_chrome_version</a>": "versionString",
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/manifest/nacl_modules.html">nacl_modules</a>": [...],
<em>// KITT:unsupported</em> "oauth2": ...,
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/manifest/offline_enabled.html">offline_enabled</a>": true,
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/omnibox.html">omnibox</a>": {}
<em>// KITT:unsupported</em> "optional_permissions": ...,
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/options.html">options_page</a>": "aFile.html",
<em>// KITT:unsupported</em> "page_actions": ...,
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/declare_permissions.html">permissions</a>": [...],
<em>// KITT:unsupported</em> "platforms": ...,
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/npapi.html">plugins</a>": [...],
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/manifest/requirements.html">requirements</a>": {...},
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/manifest/sandbox.html">sandbox</a>": [...],
<em>// KITT:unsupported</em> "script_badge": ...,
<em>// KITT:unsupported</em> "signature": ...,
<em>// KITT:unsupported</em> "spellcheck": ...,
<em>// KITT:unsupported</em> "storage": {},
<em>// KITT:unsupported</em> "system_indicator": ...,
<em>// KITT:unsupported</em> "tts_engine": ...,
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/autoupdate.html">update_url</a>": "http://path/to/updateInfo.xml",
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/manifest/web_accessible_resources.html">web_accessible_resources</a>": [...]
</pre>