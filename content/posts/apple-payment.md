---
title: "Apple Payment - (Store-kit 2)"
date: "2025-08-03"
draft: false
Summary: "In this post, we explore how to implement Apple’s StoreKit 2 in the backend—covering purchase verification, subscription management, and real-time notifications for a secure, scalable payment infrastructure."
---

Before implementing Apple payments, ensure you understand the difference between Apple Pay and StoreKit. Apple Pay (often integrated via Stripe) allows users to pay for physical goods and services, while StoreKit is required for handling in-app purchases of digital or non-physical items.

In this post, we’ll focus on implementing StoreKit from scratch—without relying on third-party providers like [RevenueCat](https://www.revenuecat.com/) to manage in-app purchases and subscriptions directly.


# Apple store-kit Payment Flow

![image alt text](/apple_payment_flow.png)

Unlike traditional payment systems, where the backend creates the order or subscription and passes its ID to the client SDK for payment processing, StoreKit works differently. In StoreKit, the payment flow is handled entirely on the client side through Apple’s StoreKit SDK. The client manages the purchase, and your backend receives the transaction response from the SDK for verification and entitlement updates.

Your backend receives payment state changes through Apple’s server-to-server webhooks, allowing you to update subscriptions and entitlements in real time.



# Backend Implementation

From Apple’s payment implementation structure, I noticed it follows a more frontend driven and less backend controlled approach. However, I prefer a backend driven flow since our backend already manages multiple payment providers like Razorpay, PhonePe, and Apple Pay. I want Apple payments to follow the same unified logic, where the backend decides when to show the payment page and handles payment orchestration consistently across all providers.


![image alt text](/plan_table.png)

> We also have a column named plan_type (on-time, subscription). Since we have payment handling for both type of payments but for apple we only implemented subscription payments

This is the Plan table, where I define all the plans users can choose. The table also includes a metadata field to store additional details such as `apple_product_id` and other provider-specific data.

## Plans handling from Admin dashboard

Since we manage multiple payment providers across different platforms, we wanted all plans to be controlled from our main Django admin dashboard. This ensures consistency across providers and avoids giving everyone access to individual payment dashboards. Before implementing Apple payments, our business team could freely create plans and set pricing through our dashboard. But with Apple payments, you can’t create plans via their APIs—you have to manage them in App Store Connect manually (seriously, what kind of idea is that, Apple?).

To create plans, go to your App Listing page in App Store Connect. Navigate to Monetization > Subscriptions (for subscription-based payments) or In-App Purchases (for one-time payments). Create a Subscription Group, then add a Subscription Plan.


> ⚠️ Important: Make sure you fill out every required field on the plan creation and review pages. Otherwise, payments may work in the sandbox environment but will be rejected in production.

> ⚠️ I will be using subscription plan since we where doing subscriptions using apple pay.

Now that we’ve created the plan in Apple’s dashboard, we decided to remove the “create” functionality from our internal dashboard since, unlike other payment providers, Apple doesn’t allow plan creation through APIs.

We also had to adjust how we handle pricing in our Plan table. Previously, the team could set any custom price freely. But with Apple, pricing must be chosen from their predefined pricing tiers, which you’ll see when creating a plan in App Store Connect.

While Apple does provide APIs to update plan details, pricing changes must use the tier IDs Apple defines. So, if you want to set a plan price like ₹324, tough luck—Apple basically says, “nope, fk yourself.”