---
title: "YQM: Chrome Extension Concept"
seoTitle: "YouTube Queue Manager: Chrome Extension Idea"
seoDescription: "Optimize YouTube using our Chrome Extension for queue management, video length limits, automated playlists, and improved user experience"
datePublished: Fri Jun 02 2023 22:05:39 GMT+0000 (Coordinated Universal Time)
cuid: clif46fph000a09mmdtl36iy9
slug: yqm-chrome-extension-concept
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/4QmSdCP4bhM/upload/2f1057c3f7dec5d92b01371d59c8a1ee.jpeg
tags: chrome-extension, youtube, idea-to-build-project, queue-management, chromeextensions

---

I love YouTube. It's the go-to platform for research, education, and entertainment. I don't watch any TV except for live sports (and even then, only during playoffs), and I can't seem to finish any TV shows even though the number of episodes has been reduced from 22-25 to 13, 10, and finally 4-6 for mini-series or UK shows. However, I can watch entire documentaries or analysis videos spanning hours or multiple parts on YouTube.

I spend most of my time on YouTube, beginning with some "Skip and Shannon: Undisputed" for sports recaps and analysis, although I mainly watch it for their banter. Later in the afternoon, I watch videos from my subscriptions, and I end the day with videos from my "Watch Later" playlist. This routine might seem reasonable, and it could be, except for the fact that I often watch videos through a queue, starting with my daily routine videos and gradually adding any recommended ones. The problem is that my queues in the mornings usually consist of only 4-5 videos, totalling 25-40 minutes, which can easily fit between deep work sessions. However, now they are approaching 40-60 minutes, and I'm concerned that it may get further out of hand if I don't limit this practice.

## Youtube Add-on Objective

I need to create a Chrome Extension that will do the following:

* Either disable or remove all the `Add to queue` buttons (on hover and via the dropdown:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684342075626/41b55093-bd9e-4bfb-8d87-093495d1ace3.png align="center")
    
* Ability to toggle this feature on/off from a popup:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684342301450/cb878bae-8e6b-4a1c-8530-262a6fa75e2a.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684342361965/c0706767-4eeb-449f-a4a8-7226bb308035.png align="center")
    
* Features to limit videos added to the queue, either by setting a queue length or setting a max total time for videos. Update the queue playlist ui to display the total time of videos in the queue.
    
* Display a tooltip, alert, etc if the `Add to queue` button is disabled (if that's what's chosen in the first bullet) and the reason is due to the set queue length or max total time.
    
* Fix this UI issue:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684361705034/88bf1be6-5283-40da-9bbb-4ed402f33247.png align="center")
    
* Enlarge Video Duration so a user is doubly aware of what they are potentially adding to a queue
    
* (Optional) Display an analysis or recap if this extension is functioning to limit watch time or queue length when it's enabled, and also unintentionally when it's disabled. Ideally, I should be able to gradually reduce my reliance on this self-imposed restriction and only activate it when I start slacking off, based on the usage review.
    
* (Optional) Automate video removal from queue upon video end. Upon queue completion also clear the queue without having to interact with the modal
    
* (Optional) Automate queue generation, such as in the mornings, create a queue with 2-3 of the top Undisputed videos that are only 10-13 minutes. Additionally, maybe delete/clear the queue at a certain time. Do the same for the afternoon but focused on subscriptions. For subscriptions, further filtering will be needed to prioritize channels weighed with watch time. For later in the evening, maybe disable functionality altogether as it's my time.
    
* (Optional) Maybe in the evening automate plucking some videos from my ever-increasing "Watch later" playlist into a queue. After viewing a video selected from the "Watch Later" playlist, the video should be removed from the queue and playlist.
    

So, we have a plan on what to create, now it's just a matter of breaking ground on it. There are a few features that are a must-have for a v1 release (if I plan on releasing it on the web store), but I hope to keep on working on it to either implement the optional features or at least open source it and invite and help others contribute them.

In the next article in this series, we will start with setting up the structure and prerequisites for a Chrome Extension and implement the first two features.