

Hey All! Critical Thinking has worked together with the fantastic CTer @busfactor to create an exclusive plugin for you all! This plugin detects Client-side Path Traversal vulnerabilities and logs them in the console for you via a Chrome Extension. Here is a quick video demoing it (see attached). The TLDR is:

1. Install the plugin & pin it to your extension tab in chrome
2. Turn on the places you'd like to search for
3. The plugin will detect values which originate from top-level query params and hashes being inserted into subresources such as fetch requests or iframes
4. See the notification on it
5. Click `Print Findings` to print the findings to the JS Console

We've already found multiple bugs with this! We think you will too!


