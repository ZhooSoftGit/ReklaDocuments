# Business Requirements Document (BRD) – Rating & Feedback System

## Overview

The platform SHALL implement a rating and feedback system to:
- Maintain service quality
- Identify poor behavior
- Improve trust between riders and drivers

---

## Table of Contents

1. [Rating Flow](#1-rating-flow)
2. [Rating Scale](#2-rating-scale)
3. [Feedback](#3-feedback)
4. [Rating Rules](#4-rating-rules)
5. [Storage Rules](#5-storage-rules)
6. [Rating Calculation](#6-rating-calculation)
7. [Visibility](#7-visibility)
8. [Low Rating Handling](#8-low-rating-handling)
9. [Abuse Prevention](#9-abuse-prevention)
10. [Admin Controls](#10-admin-controls)
11. [Transparency Rules](#11-transparency-rules)
12. [Optional Enhancements (Future)](#12-optional-enhancements-future)
13. [Final Statement](#13-final-statement)

---

## 1. Rating Flow

### Trigger
- Rating SHALL be requested ONLY after a ride is successfully completed

### Participants
- Rider SHALL rate Driver (Mandatory)
- Driver MAY rate Rider (Optional)

---

## 2. Rating Scale
- Rating SHALL be between 1 to 5 stars

### Meaning
- 5 → Excellent
- 4 → Good
- 3 → Average
- 2 → Poor
- 1 → Very Poor

---

## 3. Feedback (Optional)
- User MAY provide feedback text
- System MAY provide predefined reasons

#### For Riders:
- Driving behavior
- Vehicle condition
- Timeliness
- Cleanliness

#### For Drivers:
- Rider behavior
- Payment issues
- Delay at pickup

---

## 4. Rating Rules
- Rating MUST be submitted within defined time (e.g., 24 hours)
- If not submitted, system MAY auto-close without rating

---

## 5. Storage Rules
System SHALL store:
- Ride ID
- Rater (Driver / Rider)
- Rating value
- Feedback (if provided)
- Timestamp

---

## 6. Rating Calculation
### Driver Rating
- System SHALL calculate average rating based on last N rides (e.g., last 100 rides)
- Recent ratings SHOULD have higher importance (optional future)

---

## 7. Visibility
### Driver App
- Driver SHALL see average rating and total number of ratings

### Rider App
- Rider MAY see driver rating before accepting ride

---

## 8. Low Rating Handling
### Threshold
- Minimum rating threshold = 3.5 (configurable)

### Rules
If driver rating < threshold:
- System MAY:
  - Send warning
  - Temporarily restrict account
  - Flag for admin review

---

## 9. Abuse Prevention
System SHALL prevent:
- Multiple ratings for same ride
- Fake or duplicate ratings
- Self-rating

---

## 10. Admin Controls
Admin SHALL be able to:
- View all ratings
- Remove invalid ratings
- Take action on low-rated drivers

---

## 11. Transparency Rules
- Ratings MUST reflect actual user feedback
- No manual manipulation without audit

---

## 12. Optional Enhancements (Future)
- Tag-based feedback analytics
- Driver performance reports
- Incentives for high-rated drivers

---

## 13. Final Statement

The platform SHALL implement a simple and reliable rating system where riders rate drivers after each ride, enabling quality monitoring, fair evaluation, and improved user trust.
