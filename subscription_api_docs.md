# Subscription API Documentation

## Overview
The Subscription System enables users to subscribe to premium plans with support for multiple payment methods including Card payments (via Flutterwave) for web applications and In-App Purchases (IAP) for iOS and Android. The system includes real-time WebSocket notifications for subscription status updates and manages recurring billing automatically.

---

## Table of Contents
1. [Subscription Plans](#subscription-plans)
2. [User Subscriptions](#user-subscriptions)
3. [Payment Processing](#payment-processing)
4. [Payment Verification](#payment-verification)

---

## Subscription Plans

### 1. List Available Plans
Retrieve all active subscription plans with their pricing offers.

**Endpoint:** `GET /subscription/plans/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "count": 3,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "HafarPlus Weekly",
      "display_name": "HafarPlus",
      "plan_type": "plus",
      "description": "Access to premium features with weekly billing",
      "weekly_offer": {
        "id": 1,
        "price_naira": 1500.00,
        "price_dollar": 2.50,
        "card_discount": 10,
        "gateway_plan_id": "flw_plan_xyz123",
        "revenuecat_id": "hafar_plus_weekly",
        
      },
      "monthly_offer": {
        "id": 2,
        "price_naira": 5000.00,
        "price_dollar": 8.00,
        "card_discount": 15,
        "gateway_plan_id": "flw_plan_abc456",
        "revenuecat_id": "hafar_plus_monthly",
        
      }
    },
    {
      "id": 2,
      "name": "Premium",
      "display_name": "Premium",
      "plan_type": "premium",
      "description": "Full access to all premium features",
      "weekly_offer": {
        "id": 3,
        "price_naira": 2500.00,
        "price_dollar": 4.00,
        "card_discount": 10,
        "gateway_plan_id": "flw_plan_def789",
        "revenuecat_id": "hafar_premium_weekly",
        
      },
      "monthly_offer": {
        "id": 4,
        "price_naira": 8000.00,
        "price_dollar": 12.00,
        "card_discount": 15,
        "gateway_plan_id": "flw_plan_ghi012",
        "revenuecat_id": "hafar_premium_monthly",
        
      }
    }
  ]
}
```

**Notes:**
- Only active plans are returned
- Each plan includes both weekly and monthly pricing options
- `card_discount` is the percentage discount applied when paying with card
- `revenuecat_id` is used for In-App Purchases (IAP)
- `gateway_plan_id` is used for Flutterwave card payments

---

### 2. Get Single Plan
Retrieve details of a specific subscription plan.

**Endpoint:** `GET /subscription/plans/{plan_id}/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "id": 1,
  "name": "HafarPlus Weekly",
  "display_name": "HafarPlus",
  "plan_type": "plus",
  "description": "Access to premium features with weekly billing",
  "weekly_offer": {
    "id": 1,
    "price_naira": 1500.00,
    "price_dollar": 2.50,
    "card_discount": 10,
    "gateway_plan_id": "flw_plan_xyz123",
    "revenuecat_id": "hafar_plus_weekly",
    
  },
  "monthly_offer": {
    "id": 2,
    "price_naira": 5000.00,
    "price_dollar": 8.00,
    "card_discount": 15,
    "gateway_plan_id": "flw_plan_abc456",
    "revenuecat_id": "hafar_plus_monthly",
    
  }
}
```


**Status Values:**
- `ACTIVE` - Currently active subscription
- `CANCELED` - Canceled but still active until end date
- `EXPIRED` - Subscription has ended
- `PAYMENT_FAILED` - Payment processing failed
- `PENDING` - Awaiting payment confirmation
- `TRIAL` - Free trial period

**Payment Method Values:**
- `CARD` or `FLUTTERWAVE` - Paid via Flutterwave card payment
- `Google Play` - Paid via Google Play Store
- `Apple Store` - Paid via Apple App Store

---

## Payment Processing

### 4. Start Subscription Payment
Initialize a subscription payment process for card payments.

**Endpoint:** `POST /subscription/subscription-payment/`

**Authentication:** Required

**Content-Type:** `application/json`

**Request Body:**
```json
{
  "plan_id": 1,
  "offer": "WEEKLY",
  "payment_method": "CARD"
}
```

**Request Parameters:**
- `plan_id` (integer, required): ID of the subscription plan
- `offer` (string, required): Billing cycle - `WEEKLY` or `MONTHLY`
- `payment_method` (string, required): Payment method - `CARD`, `FLUTTERWAVE`, or `IAP`

**Success Response - Card Payment (200):**
```json
{
  "provider": "FLUTTERWAVE",
  "offer": "WEEKLY",
  "checkout_url": "https://checkout.flutterwave.com/v3/hosted/pay/abc123xyz"
}
```


**Notes:**
- **Card/Flutterwave Payment:** Returns hosted checkout URL for user to complete payment
- After successful card payment, user will be redirected with `tx_ref` parameter
- WebSocket notification will be sent upon successful payment verification

**Error Response (400) - Invalid Plan:**
```json
{
  "errors": {
    "plan_id": ["Invalid or inactive subscription plan."]
  }
}
```

**Error Response (400) - Missing Offer:**
```json
{
  "errors": {
    "offer": ["The selected plan does not have an active WEEKLY offer."]
  }
}
```

---

### 5. In-App Purchase (IAP) Flow
For In-App Purchases, the payment flow is handled entirely on the frontend using RevenueCat.

**Backend Processing:**
- RevenueCat webhook notifies backend of successful purchase
- Backend creates/updates subscription record automatically
- WebSocket notification sent to user
- No direct API call needed from frontend after purchase


## Testing Guide

### Testing Card Payments (Flutterwave)

#### 1. Test Card Details
```
Card Number: 5531886652142950
CVV: 564
Expiry: 09/32
PIN: 3310
OTP: 12345
```

#### 2. Start Payment
```http
POST /subscription/subscription-payment/
Authorization: Bearer {token}
Content-Type: application/json

{
  "plan_id": 1,
  "offer": "WEEKLY",
  "payment_method": "CARD"
}
```

#### 3. Complete Payment
Navigate to the returned `checkout_url` and complete payment using test card.