Chrome bundle manifest compliance
---------------------------------

**Regular expression** referenced in ```content_scripts``` are of format supported by [```NSRegularExpression```](https://developer.apple.com/library/mac/documentation/Foundation/Reference/NSRegularExpression_Class/Reference/Reference.html), which uses implementation of [International Components for Unicode](http://userguide.icu-project.org/strings/regexp). Enclosing slashes inside the expression string are tolerated (and stripped prior to regex evaluation). But it must be **inside** the string quotes, not **instead** of them. In other words, the expression is still a string, not a JavaScript ```RegeExp``` constructor.

**Duplicated and modified from**

[http://developer.chrome.com/extensions/manifest.html](http://developer.chrome.com/extensions/manifest.html)

<pre>
<em>// Required</em>
"<a href="http://developer.chrome.com/extensions/manifest/manifest_version.html">manifest_version</a>": 2,
"<a href="http://developer.chrome.com/extensions/manifest/name.html">name</a>": "My Extension",
"<a href="http://developer.chrome.com/extensions/manifest/version.html">version</a>": "versionString",

<em>// Recommended</em>
<em>// KITT:unsupported</em> "default_locale"
"<a href="http://developer.chrome.com/extensions/manifest/description.html">description</a>": "A plain text description",
"<a href="http://developer.chrome.com/extensions/manifest/icons.html">icons</a>": {
  <em>// KITT:only first entry will be taken, regardless of its size</em>
},
  
<em>// Pick one (or none)</em>
"<a href="http://developer.chrome.com/extensions/browserAction.html">browser_action</a>": {
  "default_icon": {  <em>// KITT:will fall back to main icon if not present</em>
    <em>// KITT:only first entry will be taken, regardless of its size</em>
    },
    <em>// KITT:unsupported</em> "default_title"
    "default_popup": "popup.html"
},
<em>// KITT:unsupported</em> "page_action"

<em>// Optional</em>
"author": ...,
"<a href="http://developer.chrome.com/extensions/background_pages.html">background</a>": {
  <em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/event_pages.html">persistent</a>"
  "scripts": [
    "background.js" <em>// KITT: only first entry will be taken</em>
    <em>// KITT: @TODO will change with persistent bundle unpacking</em>
  ],
  <em>// KITT:unsupported</em> "page"
},
<em>// KITT:unsupported</em> "background_page": ...,
<em>// KITT:unsupported</em> "<a href="http://developer.chrome.com/extensions/override.html">chrome_url_overrides</a>": {...},
<em>// KITT:unsupported</em> "commands": ...,
<em>// KITT:unsupported</em> "content_pack": ...,
"<a href="http://developer.chrome.com/extensions/content_scripts.html">content_scripts</a>": {
  "matches" : [
    "/^http.+$/" <em>// KITT:any number of strings</em>
    <em>// KITT:must be <b>regular expression</b>, NOT <a href="http://developer.chrome.com/extensions/match_patterns.html">chrome globbing patterns</a></em>
  ],
  "exclude_matches" : [
    <em>// see "matches" above</em>
  ],
  <em>// KITT:unsupported</em> "css" : [],
  "js" : [
    "myscript.js" <em>// KITT: only first entry will be taken</em>
    <em>// KITT: @TODO will change with persistent bundle unpacking</em>
  ],
  <em>// KITT:unsupported</em> "run_at" <em>// KITT:defaults to "document_end"</em>
  <em>// KITT:unsupported</em> "include_globs"
  <em>// KITT:unsupported</em> "exclude_globs"  
},
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