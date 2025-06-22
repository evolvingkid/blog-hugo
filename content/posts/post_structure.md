---
title: "Building Grapevine’s Scalable Content Architecture"
date: 2025-06-22T15:58:30+05:30
draft: False
Summary: "In this post, we dive into how we designed Grapevine’s post data structure to be modular, performant, and future-proof. From supporting diverse content types to enabling rapid iteration, here’s how we built a content architecture that scales with our platform."
---

> you may not want to read the 1st para since it's just me festered toward JS Frontend community 😂

As many of you know, Grapevine uses React Native for our mobile application. When you look around the JavaScript community, you’ll often see developers releasing new state management libraries — many of which are just sugar-coated versions of existing ones. What’s rarely discussed, however, is how to effectively structure and handle the variety of data types that client applications deal with every day. Instead, the focus tends to be on plugging in the latest package and injecting backend data directly into the UI — and calling it a day.

## Problem with old data structure

As I mentioned in my previous blog about Grapevine’s comment structure, the app was originally built by a freelance developer with only limited functionality in mind. Naturally, many of the early decisions didn’t account for the range of features we would need to support as the product evolved. This led to structural limitations that made it difficult to scale or introduce new types of content without major refactoring.

### 1. Duplicated data

In the Grapevine app, users can view posts created by both identified and anonymous users. If you explore the app, you’ll notice that we technically maintain three different post lists within the Community tab — Popular, Recent, and Top of the Week. Beyond that, we also have the Company tab inside Community, which displays posts related to specific companies. Additionally, each Community Post page has its own post listings and a discover post list inside post detail page.

Across all of these areas, we support three core filters or types of post lists — meaning we’re not just dealing with one unified feed, but multiple filtered views that require consistency and performance.

We use React Redux to manage state across the app, where each type of post screen has its own dedicated slice. Think of our Redux store as an object with different keys, and each key represents a slice that holds data for a specific post list (like Popular, Recent, or Top of the Week). This means that at any given time, we’re maintaining up to four separate sets of post data in memory, depending on what the user is viewing.

To enhance performance — especially for users with slower internet connections — we also persist post data using AsyncStorage. This acts as a lightweight caching layer, similar to what apps like Instagram do, allowing previously viewed posts to load instantly even when the network is weak or unavailable.

![post structure image](/post_old_structure.png)

The image above shows how post data was previously handled — a very straightforward and minimal setup that simply appended new data as it came in.

### 2. Hard to keep track on changes

As you can see, this approach led to duplicated data across slices, and different user actions required individual state updates, making the system harder to maintain and prone to inconsistencies.

For example, when a user likes a post in the Home tab under the Popular filter, we update the liked state of that specific post to true. However, if the user then navigates to the Company tab within the Community section — where the same post might already be cached — we typically don’t re-fetch the data from the API, since changes at the top layer of posts are infrequent (we’re not Reddit-scale... yet). As a result, we have to manually update the post state individually across all relevant slices, to keep everything in sync.

### 3. Caching more post list

As you can see, most post sections — except for the Discover list — support multiple filters. In our current implementation, switching between filters requires calling the API each time, which leads to duplicate post lists in state and increases the complexity of keeping everything in sync.

We currently support five main filters: Popular, Recent, Top of the Week, Top of the Month, and Top of the Year. However, aside from Popular, most of these don’t need real-time updates. For example, filters like Top of the Year or Top of the Month can be cached and shown instantly, without repeatedly hitting the API.

To optimize performance and reduce unnecessary network calls, we aim to update all filters (except Popular) once every hour, since post interactions at this level don’t change frequently — unlike on high-traffic platforms like Reddit.

## New data structure

With all the issues mentioned above, we were constantly running into bugs caused by out-of-sync post data. Every time a new user interaction or feature was added — like bookmarking, reacting, or editing — we had to make sure that every instance of the post across different filters and screens stayed in sync. This not only introduced fragile code but also significantly slowed down our development speed.

### 1. New centralized data system

To address these issues, we decided to centralize all post data, since the root cause of most bugs was having to manage the same post across too many places. We introduced a dedicated postSlice in our Redux store, which acts as the single source of truth for all post data in the app. Instead of storing full post lists in each slice, we now store posts in a key-value format, where the key is the postId, and the value is the post object returned from the API — essentially like a Map of all posts.

Now that all post data is centralized in the postSlice, we needed a way to determine which posts belong to which screen or filter. Instead of storing full post objects in each slice like before, we now store arrays of postIds in the respective slices (e.g., communitySlice, companySlice, etc.).

When rendering, each slice simply refers to the centralized post data by calling a shared `getPost(postId)` function. This allows the UI to remain in sync, since any update to a post in the centralized postSlice automatically reflects across all screens — with no duplication or extra update logic.


![post structure image](/post_new_strcuture.png)

As you can see, we now maintain a dedicated object for each filter — such as Recent, Popular, and others — within their respective slices. Each of these objects holds only the list of relevant `postIds`, making the structure cleaner, easier to manage, and more predictable across screens.

### Easy to sync

Now that post data is centralized, updating any post is as simple as modifying the `postSlice` — and the change is instantly reflected across the entire app. This eliminates the need to manually sync state in multiple places.

Updating a post has also become much more efficient. Instead of iterating through multiple post lists, we can now directly access and update the post using its `postId`, making our state updates both faster and more reliable.

### Caching the post

Since we now cache post data centrally, any updates received from API responses are reflected immediately across the app. This allows us to reuse the same post data for the Post Detail screen, eliminating the need to block the UI while fetching fresh data.

Instead, we display the cached post instantly and trigger a background API call to fetch the latest version. This ensures a smooth user experience — the post loads immediately, and any server-side updates sync seamlessly in the background.