# Business Requirements Document (BRD) – Referral Reward & Coupon Management

## Overview

The platform SHALL implement a structured reward system where rider benefits are managed via coupons and driver incentives via wallet credits, with strict tracking, validation, and fraud prevention mechanisms.

---

## Table of Contents

1. [Rider Reward Handling (Coupon System)](#1-rider-reward-handling-coupon-system)
2. [Driver Reward Handling (Wallet Credit)](#2-driver-reward-handling-wallet-credit)
3. [Reward Processing Flow](#3-reward-processing-flow)
4. [Storage Requirements](#4-storage-requirements)
5. [Expiry & Cleanup](#5-expiry--cleanup)
6. [Fraud Prevention](#6-fraud-prevention)
7. [Transparency](#7-transparency)
8. [Final Statement](#8-final-statement)

---

## 1. Rider Reward Handling (Coupon System)

### 1.1 Reward Type
- Rider rewards SHALL be issued as **discount coupons**
- No wallet balance SHALL be maintained for riders

### 1.2 Coupon Creation
- When referral conditions are met, the system SHALL create a coupon for:
  - Referrer
  - Referee (new user)

### 1.3 Coupon Properties
Each coupon SHALL include:
- Coupon Code / Identifier
- User ID
- Discount Amount (e.g., ₹50)
- Expiry Date
- Status (Active / Used / Expired)

### 1.4 Coupon Usage Rules
- Coupon SHALL be applied during ride booking
- Only ONE coupon can be used per ride
- Coupon SHALL be applied BEFORE ride confirmation

### 1.5 Validation Rules
Coupon SHALL be valid ONLY IF:
- Status = Active
- Not expired
- Not previously used

### 1.6 Post Usage
- After successful ride completion, coupon status SHALL be updated to “Used”

---

## 2. Driver Reward Handling (Wallet Credit)

### 2.1 Reward Type
- Driver referral rewards SHALL be credited to **driver wallet**

### 2.2 Reward Trigger
Reward SHALL be credited ONLY IF:
- Referred driver:
  - Completes onboarding
  - Completes required number of rides

### 2.3 Wallet Credit Behavior
- Reward amount SHALL be added to wallet balance
- Example: Wallet = -₹50, Reward = ₹300, New balance = ₹250

### 2.4 Usage
- Wallet balance SHALL be used ONLY for platform fee settlement

---

## 3. Reward Processing Flow

### Rider Flow
1. Referral code applied at signup
2. First ride completed
3. System generates coupon(s)
4. Coupons assigned to users

### Driver Flow
1. Referral code applied at registration
2. Driver completes onboarding
3. Driver completes required rides
4. System credits wallet reward

---

## 4. Storage Requirements

### Coupons
- User-wise coupon list
- Status tracking
- Expiry handling

### Wallet Transactions
- All reward credits MUST be logged
- Each transaction MUST have reference (Referral ID)

### Referral Tracking
- Referrer ID
- Referred user/driver ID
- Status (Pending / Completed / Rewarded)

---

## 5. Expiry & Cleanup
- Coupons SHALL expire after configured duration (e.g., 30 days)
- Expired coupons SHALL NOT be usable

---

## 6. Fraud Prevention
System SHALL ensure:
- No duplicate accounts
- No self-referrals
- Rewards only after valid activity

---

## 7. Transparency
Users MUST be able to see:
- Available coupons
- Coupon expiry
- Referral reward status

Drivers MUST see:
- Wallet credits from referrals
- Reward history

---

## 8. Final Statement

The platform SHALL implement a structured reward system where rider benefits are managed via coupons and driver incentives via wallet credits, with strict tracking, validation, and fraud prevention mechanisms.
