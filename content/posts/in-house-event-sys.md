---
title: "In-House Event System"
date: 2025-12-15T14:52:26+05:30
draft: false
Summary: "Grapevine’s in-house event system enables near real-time user engagement analytics across multiple apps."
---

Grapevine builds consumer facing applications where user engagement metrics play a critical role. In many parts of the product, we need to show data such as how many people viewed a post, visited a profile, or interacted with specific content.

As the company grew, this requirement became more complex. We now operate multiple apps, and the same events need to be tracked consistently across different platforms and surfaces. More importantly, these metrics must be available to users in near real time. Similar to how LinkedIn displays profile views or how Instagram surfaces profile and post analytics.

This growing demand for accurate, real-time engagement data across apps was one of the key reasons we started investing in a unified, in-house event analytics system.

## Initial thought process

As you can see, Grapevine operates multiple products, so we needed our event system to behave like a shared package rather than a product-specific solution. The idea was to keep the integration simple and consistent across all apps.

Both client and backend services should be able to specify an event name along with a JSON payload of properties, send it to a common event ingestion endpoint, and let the event system handle the rest — validation, storage, and downstream processing.

By standardizing event ingestion in this way, we ensured that new products and services could start tracking events with minimal setup, while keeping the analytics pipeline uniform and scalable across the entire company.

# Infra

Since the system processes thousands to millions of events coming from multiple applications, the backend needs to handle this volume efficiently while using as few resources as possible. Scalability and performance were core requirements from day one, not afterthoughts.

This pushed us to design the ingestion layer to be lightweight, horizontally scalable, and optimized for high write throughput, ensuring the system could grow alongside our products without a proportional increase in infrastructure cost.

![image alt text](/event-sys-design.png)

All incoming event requests are captured by a lightweight event listener service. This service is intentionally minimal — it simply accepts the event payload and pushes it directly into Kafka. No validation or preprocessing happens at this stage.

This means that even if an event has structural issues, it is still written to Kafka. While this can result in some additional storage usage for invalid events, it’s an acceptable trade-off for us. Since all events are produced by internal developers, the likelihood of malformed event payloads is relatively low.

By avoiding any preprocessing at the ingestion layer, we keep API latency extremely low, with request times typically under 80ms. This design also makes the system easy to scale horizontally. As event volume increases, we can simply add more instances of the event listener service to handle the load and push events to Kafka without introducing bottlenecks.

Currently, our Kafka setup consists of three brokers, managed using KRaft mode, and deployed across multiple availability zones within the region. This setup ensures high availability — even if one Kafka broker goes down, the system continues to operate without disruption.

Designing for failure was a deliberate choice. In real-world systems, servers can fail at any time, and it’s not always possible to immediately fix an issue — whether that’s during travel, off-hours, or while asleep. Because of this, the system is built with built-in fallbacks, allowing critical event processing to continue for a reasonable period until the issue can be addressed.

By distributing Kafka across zones and removing single points of failure, we’ve made the event pipeline resilient by default, rather than relying on constant manual intervention.

The Kafka listener is where all the core validation logic lives. It consumes events from Kafka and performs schema checks, data validation, and any necessary transformations before the events move further down the pipeline.

Given the high volume of incoming events, we rely heavily on bulk write operations. Instead of writing each event individually, the listener aggregates events and, every five minutes, creates a batch that is pushed to the database in a single operation. This batching strategy significantly reduces the number of write operations and helps keep database load under control.

By processing events in bulk, we strike a balance between near real-time availability and system efficiency, ensuring the database remains performant even as event volume continues to grow.

Since we process millions of events, and these events are primarily used for analytics and aggregation, we chose ClickHouse as our database.

The Kafka listener sends batched write requests through a load balancer, which then distributes the traffic across multiple ClickHouse nodes. This setup allows us to scale write throughput horizontally while keeping ingestion latency predictable.

At the moment, we operate two ClickHouse instances, configured with asynchronous replication. This provides redundancy and fault tolerance — if one node becomes unavailable, the other can continue serving both writes and queries without blocking the event pipeline.

Using ClickHouse in combination with batched inserts and load-balanced ingestion gives us a system that can handle high event volume efficiently while remaining resilient as traffic grows.

# Consuming event 

Now that event ingestion and storage are in place, the next challenge is consuming these events to power user-facing features and business logic. This includes use cases like view counts, profile analytics, and other engagement metrics.

Since these events are also used for analytics, it’s critical that read queries do not slow down the system or impact event ingestion. User-facing requests must remain fast and predictable, even while large aggregation queries are running in the background.

![image alt text](/event_table.png)

For example, consider an event named profile_viewed. One of the product requirements is to show how many times a user’s profile has been visited as part of the user GET API response.

Each time this event occurs, it is stored in the events table along with its event name, parameters, and the user ID from which the event was generated. This raw event data serves as the source of truth for analytics, but querying it directly for every API request would be inefficient at scale.

That’s why we use ClickHouse materialized views to pre-process and organize event data. For the profile_viewed event, we created a dedicated table named profile_viewed.

This materialized view listens to the raw events table, filters only events with the profile_viewed event name, and extracts the required fields from the event parameters into structured columns. As a result, all profile view events are stored in a query-optimized format, making them easy and efficient to aggregate.