<!-- START_METADATA
---
title: How the eCommerce API works in the store
sidebar_label: How it works online
sidebar_position: 9
description: How the eCommerce API works in the store.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# How the eCommerce API works in the store

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/docs/APIs/ecom-api).

<!-- END_COMMENT -->

Vipps-in-store is designed for stores that want to let customers pay easily and quickly from their own phone, without the need to touch the terminal. This page shows an example of how you can offer contactless payment to your customers by integrating Vipps in your POS system.

It is also possible to use our simpler solution Vippsnummer, but then the amount must be entered manually at checkout, and there will be some follow-up with accounting and settlement.

## The payment process in store

![In store process](../images/vipps-in-store-process.svg)

## 1. Add products to sell

Add the products that the customer wants to buy in the POS system.

![The POS system](../images/vipps-in-store-step1.svg)

## 2. Enter the customer's phone number

When the customer is ready to pay, choose “Pay with Vipps”, and enter the customer’s phone number.

![Enter phone number](../images/vipps-in-store-step2.svg)

## 3. The customer confirms payment in Vipps

The customer confirms the payment in Vipps on their phone.

![Confirm payment](../images/vipps-in-store-step3-2.svg)

## 4. Payment is registered

The payment is registered in the POS system.

![Payment confirmation](../images/vipps-in-store-step4.svg)

Great! Now you know how the Vipps in store payment process works.

Read all the technical details in the [Vipps eCommerce API Guide](../vipps-ecom-api.md)
