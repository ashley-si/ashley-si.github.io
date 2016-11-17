---
layout: post
comments: true
disqus_identifier: /lifecycle
title: "Lifecycle hooks in Angular 2 "
permalink: "/lifecycle"
---

This is a long overdue post. I encountered a very interesting error, and I used a very hacky way to get around it. However, I haven't found a neat way to solve it.

Our App is about making event agenda easier to create and view. On the event page, there is a calendar view of all the event's sessions. When the user clicks on the session, a modal with the session's details pops up. We wanted to give each session modal a unique link so that user can share a particular session with others.

The implementation is simple. Since the modal is not always in the DOM, I can't treat it as an anchor. Instead, when a modal is opened, I append the `/session/:sessionId` to the end of the current url and remove it when the modal is closed. When the component is first loaded, I check if there is a `sessionId` in the parameter and open the modal accordingly. I do the check in `ngOnInit` lifehook.

However, I got the following error and the message ` It seems like the view has been created after its parent and its children have been dirty checked. Has it been created in a change detection hook ?` which seems to give away some crucial information.

![modal error](/images/modal_error.jpg)

I tried to open the modal in different lifecycle hooks, but none of them worked. It turns out that no binding can be changed in the lifecycle hooks. There's some [discussion](https://github.com/angular/angular/issues/10131) going on. Bascially the explanation given is that any change in binding should be followed immediately by a round of change detection. However, the lifecycle hooks themselves are the result if change detection. Normally to solve this, there's only need to be `ChangeDetectorRef.detectChanges ` after the change in binding. However, the modal library we used is standalone and the modals produced are outside the component template, so that magical line has no effect. I have to open the modal in a `setTimeout()` method, because this method will cause change detection in the entire app. Not very efficient, but it worked. I think there should be better solution but I couldn't find it and we couldn't afford to dwell on it long, so another thing weighing down on our app's performance.
