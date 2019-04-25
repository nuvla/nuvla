---
layout: default
title: Embedded Portal
parent: Administrators
nav_order: 3
---

Embedded Portal
===============

You may want to embed the Nuvla user interface into another portal.
The easiest way to do this is simply to include the Nuvla browser
interface into the portal via an `iframe` element. 

To embed the Nuvla browser-based interface, add this content:

```html
<iframe src="https://nuv.la/webui"
        style="width:100%; height:100ex; v-scroll:auto; padding: 1ex !important; margin: 0 !important">
    <p>Your browser does not support iframes.</p>
</iframe>
```

to the `main` element of a page on the portal.

Embedding the interface in this way avoids issues with cross-site
scripting restrictions, conflicts with Javascript libraries, etc.
