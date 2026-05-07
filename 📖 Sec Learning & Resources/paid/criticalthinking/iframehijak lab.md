
It is possible for an attacker's page to hijack a victim page's `window.open` call if:

- the `window.open` uses a predictable `name`.
- the victim tab is in the same tab group as the attacker's tab.
- the attacker's malicious iframe that will do the hijacking is the same origin as the page the victim is trying to open. (Can be redirected after, ofc)

Here is a little lab to demonstrate this: [https://attacker.poc.rhynorater.com/iframeHijacking/attackerPage.html](https://attacker.poc.rhynorater.com/iframeHijacking/attackerPage.html "https://attacker.poc.rhynorater.com/iframeHijacking/attackerPage.html") The attacker's page first opens a victim page. The victim page thinks they are opening a pop up, but instead, that pop up is opened into the iframe on the attacker's tab. The attacker can then redirect that iframe to wherever they want and manipulate the relationship between the victim page and the page that it assumes to be it's popup.