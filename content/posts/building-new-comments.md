---
title: "Grapevine's Comment Architecture"
date: "2025-05-25"
Summary: "This is a brief overview of how we rebuilt the comment section of Grapevine."
---

If you've used Grapevine’s comment section before, you might remember that the design looked very different just a year ago. What we launched recently isn't just a visual redesign — it’s a complete rewrite of the entire comment system, rebuilt from the ground up.

Before we dive into why we decided to rebuild the comment system from the ground up, it’s important to understand what made the old structure such a nightmare for us.

## Problems with old structure

The initial version of Grapevine was built by an outsourced team, which meant that many architectural decisions were made with short-term goals in mind rather than long-term scalability or maintainability.

### 1. Flatlist virtualization not working

That’s right — we were relying on a FlatList for rendering comments, but virtualization was breaking down, leading to major performance issues as the number of comments grew.

This issue stemmed from the way we were rendering FlatList items. On Grapevine’s post details page, everything was inside a single FlatList — with the post content as the header component and the comments rendered in renderItem. But here’s where it gets tricky: Grapevine supports nested comments. So how were we displaying those? Well... the funny (and painful) part is that for every nested comment, we were rendering a new FlatList — again and again.

![image alt text](/comment_old_struct.png)

If you're wondering what the problem with this structure was, let me explain. FlatList handles virtualization based on the items it directly renders. But in our setup, we had another FlatList nested inside each top-level comment. That means the nested comment tiles — rendered by the inner FlatLists — weren’t getting virtualized at all. Now imagine this: each top-level comment had its own FlatList, and inside each of those were several level-2 comments. This was happening across potentially hundreds of comments per post. Unsurprisingly, things started to lag badly. In our testing, the app began to hang after rendering just 50 comments.

### 2. Over using lottie

If you take a closer look at our comment tiles, each one had three action buttons — and every button included a press animation. To handle those animations, we were using Lottie.

While Lottie is great for rich animations, using it on every single comment tile — especially in large lists — added significant rendering overhead. Each Lottie animation was consuming around 40ms to load and render. With three Lottie files per comment tile, and around 20 comment tiles visible at once, we were effectively loading 60 Lottie instances in a single render pass — that’s roughly 2400ms of processing time just for animations. Unsurprisingly, this was a major performance bottleneck.

### 3. Messed up code

On top of all that, we were also dealing with different types of posts and comments — text, GIFs, and new formats we planned to support in the future. The codebase had grown messy over time, with deeply nested if-else ladders and no consistent structure or design patterns. Debugging even a simple issue could take days, because tracking down the source of a bug in such a tangled system was incredibly time-consuming

### 4. Not scalable

We were also planning to run several experiments on the post details page — like adding a 'Discover More' section, supporting infinite nested comments like Reddit, and much more. But with the way things were structured, any new feature felt nearly impossible to implement. The existing architecture was so fragile that even small changes risked breaking the whole page

## New comment system

Before jumping into building a new comment system, we had to face reality: we didn’t have a big timeline, and several features were already blocked because of these issues. As a startup, our primary goal is to move fast, experiment often, and figure out what works — and what doesn’t. We also don’t have the luxury of a large team to parallelize everything or try overly ambitious solutions. For example, replacing Lottie with Rive might seem like a good idea on paper, but it would require our design team’s involvement, slowing down other critical work. So we had to be smart, practical, and focused on impact.

### 1. New rendering method

In the old design, our comment rendering logic was driven by a data structure that looked something like this:

```JSON

{
    "comments" : [
        {
        "content": "comment level 1",
        "comments" [
            {
                "content": "comment level 2"
            }
            ]
        }
    ]
}


```

In the new system, we transform the comments returned from the API into a flattened structure like this:

```JSON

{
    "comments": [
        {
            "content": "comment level 1",
            "level": 0,
            "type": "comment"
        },
        {
            "content": "comment level 2",
            "level": 1,
            "type": "comment"
        }
    ]
}

```


By flattening the data and removing all nested arrays, we no longer need to render nested FlatLists. Now, every comment — even deeply nested ones — exists as a single item in a unified list. This means we can take full advantage of FlatList’s virtualization, significantly improving performance and simplifying the rendering logic.

You might be wondering how we display nested comments in the UI if the array is flattened. That’s where the level property comes in. Each comment item includes a level field that tells us how deeply nested it is. In the UI, we use that level to visually indent the comment — for example, applying a paddingLeft equal to level * 10. This gives the illusion of a nested thread, without the performance cost of actual nested components.

### 2. Fixing slow renders from lottie

Removing Lottie and switching to something like Rive wasn't an option for us. We had to get this entire fix done within a week or two, and pulling in our design team to recreate all the animations would’ve taken up bandwidth we simply didn’t have.

To solve this, we switched to using SVGs for our action buttons. SVGs are lightweight and much easier to render, which significantly reduced our initial render time. Then, when a user interacts with a button, we dynamically render the corresponding Lottie animation for just that button and play the animation. This way, we keep the UI responsive while still delivering a delightful interactive experience — without overloading the render pipeline.


### 3. Fix the code

Since we had to rebuild the entire system from scratch, we also took the opportunity to clean up the codebase and introduce better structure. One of our key improvements was eliminating deeply nested if-else ladders. Instead, we now use a consistent pattern where each comment type is mapped to its corresponding component using a simple hash map. This not only makes the code easier to read and maintain, but also makes it far more scalable when introducing new comment types in the future.

### 4. Scalable

After all these changes, our new comment system can now handle 1000+ comments smoothly and supports infinite nesting — just like Reddit. Even better, since we're transforming the comment data ourselves, we have full control over how deeply comments are nested. This means we can easily enable or limit nesting based on our experiments and engagement metrics. We're still testing how users interact with this new structure, but the flexibility we've built in gives us the freedom to adapt quickly.

Now, the post details page loads significantly faster than before, providing a much smoother experience for users. The new structure also made it incredibly easy for us to add new item types to the FlatList — like a reply card, a 'Discover More Posts' section, or conditionally removing the 'View More' button. We even added a feature that automatically scrolls the selected comment to the top when a user chooses to reply, making interactions feel more natural and fluid.

It’s been over a year since we rolled out this new comment system, and we haven’t had to revisit the core structure even once. Since then, we’ve added multiple new card types to the comment section to drive engagement — and the system has handled it all without breaking a sweat. Of course, there are still things we could polish and improve, but as a fast-moving, growing company, it’s always a balancing act between performance, user experience, and the need to experiment. This rebuild gave us a solid foundation that lets us move fast without constantly worrying about breaking things — and that’s exactly what we needed.