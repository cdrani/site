---
title: "YQM: Release and Open Source"
seoTitle: "YQM: Open Source Release"
seoDescription: "Learn to fix bugs, develop, publish, and monitor a YouTube Queue Manager Chrome extension in this all-inclusive guide"
datePublished: Mon Jun 19 2023 23:07:01 GMT+0000 (Coordinated Universal Time)
cuid: clj3guufp00020ajr9k2qhd4j
slug: yqm-release-and-open-source
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/GmvH5v9l3K4/upload/d3578f379633aeec6a71d69818ab31d8.jpeg
tags: chrome-extension, youtube, oss, yqm, chrome-web-store

---

**TLDR: Fix a bug, package and upload to Developer Dashboard, fill forms, submit for review, and look at analytics after going live.**

In this post, we want to create and submit an initial release for the Chrome Web Store and upload it to Github. Before that step though, let's fix a bug encountered.

## Bug Fix

### Issue

The issue originates from the removal of the "Add to queue" option from the menu and the thumbnail on disconnectObserver. The remove() method eliminates the node and its space in the DOM, rendering it inaccessible.

```javascript
class ObserverWrapper {
    //
    _removeQueueUI(mutationsList) {
        for (let mutation of mutationsList) {
            const { target, addedNodes } = mutation

            if (this._isPlaylistIconHover(mutation)) {
                const playlistIcon = Array.from(addedNodes)
                  .find(a => a.ariaLabel === 'Add to queue')
                playlistIcon?.remove()
            }

            if (this._isMenuOption(mutation)) {
                target?.parentElement?.parentElement?.remove()
            }
        }
    }  
}
```

So, when we turn off the extension and disconnect the observer, we can't undo the changes we made to the user interface. We'd need to recreate the menu options and playlist icons we removed. This is a tough job because each interface item has many parts, like children, styles, and events, that we need to rebuild and put back into the DOM. Yikes.

### Solution

Instead of removing the DOM elements, we can update their style instead. By setting a style of "display: none," we can hide the UI from view while still keeping it accessible. Therefore, when toggling off, we can reset the display attribute to "block" to bring it back into view. In our **ObserverWrapper** class, we now need to track a new property to either hideUI (set display: none) or showUI (set display: block).

```javascript
class ObserverWrapper {
    constructor() {
        // 
        this._isHidden = true
    }

    // . . 

    _toggleQueueUI(mutationsList) {
        const display = this._isHidden ? 'none' : 'block'

        for (let mutation of mutationsList) {
            const { target, addedNodes } = mutation

            if (this._isPlaylistIconHover(mutation)) {
                const playlistIcon = Array.from(addedNodes)
                  .find(a => a.ariaLabel === 'Add to queue')
                playlistIcon?.setAttribute('style', `display: ${display}`)
            }

            if (this._isMenuOption(mutation)) {
                const element = target.parentElement.parentElement
                element?.setAttribute('style', `display: ${display}`)
            }
        }
    }
    
    hideUI() {
        this._isHidden = true
        if (!this._observer) { 
            this._composeObserver()
        }
    }

    showUI() {
        this._isHidden = false 
    } 
}

function init() {
    const observerWrapper = new ObserverWrapper()

    function messageCallback(message) {
        message.active
          ? observerWrapper.hideUI() 
          : observerWrapper.showUI()
    }

    listenForMessage(messageCallback)
}

init()
```

## Release

Releasing an extension to the Chrome Web Store requires a developer account and a one-time payment of $5. Reference the [steps for publishing extensions](https://developer.chrome.com/docs/webstore/publish/) to set up an account and access the Developer Dashboard. Whether manual or automated deployments, we still need to zip our extension as a package, upload it to our account, submit it for review, and wait for 1-2 business days for review. If approved then our extension is published. This happens every single time we cut a release.

1. Zip the extension with only the necessary files. The "-x" flag is to not include our readme and license markdown files.
    

```bash
zip -r yqm.zip * -x '*.md'
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686265933646/f5bb53fe-ddd3-48b7-8a36-91832fcd75b8.png align="center")

1. Either drag or browse for and select the "yqm.zip" folder from the drag zone container in the Dashboard.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686266200603/fec6df68-6474-402a-b9f1-7a595c490349.png align="left")
    

1. Follow along with the listing form and enter input where necessary. Include links for support, which in this case will be the repo.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687197938962/d011d72d-6ce8-407e-9b43-f9b8c15e303a.png align="center")
    
2. Same for Privacy Policy where we need to justify our use of permissions such as activeTab, storage, etc. No need to be overly descriptive here.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687197810904/75b77324-6e0d-4bfe-8b0b-cb6960a73658.png align="center")
    
3. Distribution is the final step before submitting for review. The only concern with making the extension available worldwide is I18n, but we currently only have "on/off" text in our action icon. A quick solution would be either disabling/enabling the extension on action click or using a more universal on/off signifier like a lit/unlit lightbulb icon.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687197725835/689bd04b-08d4-4bde-bcbe-0f7e91edf898.png align="center")
    

## Publish

The package uploads and form edits are all in draft until one clicks the "Submit for Review" button. YQM's review took about 2 days, but this could have been due to it being submitted over the weekend, in other words, it was pretty fast.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687195562265/1e46f832-2f55-405d-b478-8906faeee9fd.png align="center")

Here it is on the web store: [YQM](https://chrome.google.com/webstore/detail/youtube-queue-manager/ffdonhchkjfnjhklpjijaihndnlkdbho)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687197617914/4eb258d0-5077-438e-ba43-0a5eeb9d9baa.png align="center")

## Analytics

Chrome Extensions feature built-in analytics for tracking actions such as adding or removing, as well as impressions and more. Furthermore, they can be easily integrated with Google Analytics for even more detailed data. There's a wealth of information available on the dashboard, but since this is a personal passion project, not much attention will be given to these dashboards. Instead, they will primarily be used to determine which project to prioritize based on the number of installations and active users.

#### Developer Dashboard Analytics

Impressions

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687197129671/0f2f0dfb-8130-41e8-858a-68933f51402f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687197159391/2d9b4899-1386-4fdd-8aba-e26e29e0961d.png align="center")

Installs/Uninstalls

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687197244516/97555a8b-d768-4565-80d3-7521583f93c1.png align="center")

#### Google Analytics

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687196874602/7695d010-db03-4dd1-9974-51b060dd5b1a.png align="center")

## Release

Here is the GitHub link: [YQM](https://github.com/cdrani/yqm). There's a general roadmap for what the finalized extension might have in terms of features, i18n, cross-browser support, etc. Contributors are welcome in any form - filing issues, updating docs/readme, updating icons, feature implementation, automating releases, etc.

### Wrap Up

Okay. This was a good introduction to setting up an extension and developing it from ideation to a v1 release. Additionally, we discovered the [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) web API to observe and manipulate the DOM based on changes in specific nodes. We started working on the next version's features, but right now, we're focusing more on a new project about controlling the web Spotify player, changing the playback speed, and playing only a specific part of a song.