# Technical Specification Document (Parent) - Rekla Platform

Document ID: TSD-REKLA-PARENT-009  
Version: 2.2  
Status: Draft  
Date: March 22, 2026  
Prepared By: Solution Architecture

## 1. Scope

This document is the parent high-level technical guide for the Rekla platform.

It explains:
- what applications and services are planned
- which technology stack is selected
- where each technology is used
- how the applications connect end-to-end
- how data and live updates move across the platform
- how the solution will be developed and deployed

This document is architecture-focused.

This document does not cover:
- business process explanation
- user journey details
- screen-level requirements
- detailed API contracts
- detailed database design

## 2. Solution Landscape

The platform contains 3 client applications and 2 backend services.

### 2.1 Client Applications

- Rider App
- Driver App
- Admin Portal

### 2.2 Backend Services

- Core Service
- SignalR Service

## 3. Confirmed Technology Stack

### 3.1 Client Stack

- Rider App: .NET MAUI
- Driver App: .NET MAUI
- Admin Portal: React

### 3.2 Backend Stack

- Core Service: ASP.NET Core Web API (.NET 8)
- SignalR Service: ASP.NET Core SignalR

### 3.3 Data and Platform Stack

- PostgreSQL: primary transactional database
- Redis: cache and fast operational state
- Azure Event Bus: notification event handling
- Cloudflare storage: file and document storage
- Azure Key Vault: secret and key management

### 3.4 Hosting and Delivery Stack

- Azure App Service: application and service hosting
- Azure DevOps: source control and work management
- Azure Pipelines: CI/CD and deployment automation
- Environments: Dev, Test, UAT, Prod
- Cloudflare storage: document storage and retrieval delivery

### 3.5 External Integration Stack

- Google Maps Platform: maps, location, route, ETA
- OTP provider: mobile number verification
- WhatsApp provider: operational messaging
- FCM and APNs: mobile push notifications

### 3.6 Recommended Supporting Service

- Azure Application Insights or Azure Monitor: logs, failures, performance, and health monitoring

## 4. Why This Stack Is Selected

### 4.1 MAUI

- one shared mobile codebase for Android and iOS
- reduced duplicate effort for Rider and Driver apps
- faster delivery in the initial stage

### 4.2 ASP.NET Core Web API and SignalR

- same .NET ecosystem across mobile, API, and real-time layers
- suitable for a modular monolith in the initial stage
- allows real-time traffic to be separated from standard API traffic

### 4.3 PostgreSQL and Redis

- PostgreSQL stores durable business data
- Redis supports fast-changing operational data and quick lookups

### 4.4 Azure Services

- Azure App Service simplifies hosting and deployment
- Azure Key Vault secures secrets and connection strings
- Azure Pipelines supports controlled multi-environment releases
- Azure Event Bus handles notification events outside the request-response flow

### 4.5 Cloudflare and External Providers

- Cloudflare is used for document storage and retrieval delivery
- Google Maps provides mapping and ETA capabilities
- OTP and WhatsApp providers handle communication channels
- FCM and APNs deliver push notifications to mobile devices

## 5. Architecture Overview

### 5.1 Initial Architecture

The initial architecture uses only 2 backend services:
- Core Service
- SignalR Service

This keeps implementation practical and avoids unnecessary microservice complexity.

### 5.2 Core Service

Core Service is the main backend service.

It will be implemented as a modular monolith with internal areas such as:
- authentication
- rider management
- driver management
- trip management
- dispatch
- pricing and policy
- support
- admin operations
- notification orchestration
- reporting preparation
- configuration management

These areas stay inside one deployable service, but the codebase should remain clearly separated by module.

### 5.3 SignalR Service

SignalR Service is used only for live communication.

It handles:
- live trip updates
- driver location updates
- assignment updates
- admin live monitoring
- urgent operational alerts

Business rules stay in Core Service. SignalR Service is not the system of record.

### 5.4 Future Direction

If scale increases later, some Core Service modules can be extracted as independent services.

Possible future split areas:
- dispatch
- notification
- support
- reporting
- configuration

### 5.5 Heart Functions of the Platform

The platform is designed around two critical heart functions.

1. Driver registration and activation end-to-end.
2. Real-time ride execution with no event loss and controlled failure recovery.

These are first-priority design concerns and must not be treated as optional enhancements.

### 5.6 Driver Registration and Activation Pipeline

Driver onboarding is an end-to-end controlled workflow:
- mobile OTP validation
- mandatory profile and vehicle details
- mandatory KYC document upload with preview and resubmission support
- admin approval or rejection with reason
- activation gating before online availability

Architecture rules:
- driver cannot go online before approval
- each approval/rejection action must be audit logged
- rejection reason must be delivered to driver app and retained in history
- document and KYC processing must be idempotent to survive retries

Future-ready extension:
- AI-based document and profile validation can be inserted as an asynchronous validation step before admin final approval
- AI outcome must be advisory and traceable in audit logs in initial rollout

## 6. Technology Usage by Application

### 6.1 Rider App

- built using .NET MAUI
- calls Core Service APIs over HTTP
- connects to SignalR Service for live updates
- uses Google Maps SDK for location and trip tracking
- uses OTP verification and push notifications

### 6.2 Driver App

- built using .NET MAUI
- calls Core Service APIs over HTTP
- connects to SignalR Service for assignments and live trip updates
- uses Google Maps SDK for navigation and route display
- uploads files to backend storage flow
- uses OTP verification and push notifications

### 6.3 Admin Portal

- built using React
- calls Core Service APIs over HTTP
- connects to SignalR Service for live monitoring
- uses map integration for operational dashboards

### 6.4 Core Service

- hosted in Azure App Service
- built with ASP.NET Core Web API
- persists business data in PostgreSQL
- uses Redis for cache and fast operational state
- publishes notification events to Azure Event Bus
- integrates with Blob Storage, OTP provider, WhatsApp provider, FCM, and APNs
- reads secrets from Azure Key Vault

### 6.5 SignalR Service

- hosted separately in Azure App Service
- built with ASP.NET Core SignalR
- manages connected clients and real-time channels
- pushes live events from Core Service to connected apps
- can use Redis backplane later if scale-out is required

## 7. End-to-End Technical Flow

1. Rider App, Driver App, or Admin Portal sends a request to Core Service.
2. Core Service validates the request and executes business logic.
3. Core Service stores persistent data in PostgreSQL.
4. Core Service updates Redis when fast operational state is needed.
5. Core Service sends live events to SignalR Service when immediate client updates are required.
6. SignalR Service pushes those updates to connected Rider, Driver, or Admin clients.
7. Core Service publishes notification events to Azure Event Bus when notification delivery should happen asynchronously.
8. Notification handler or worker sends the message to push, OTP, SMS, or WhatsApp provider.
9. File uploads are stored in Cloudflare-managed storage.
10. Secrets and credentials are resolved from Azure Key Vault.

## 8. Data Transfer and Processing Model

### 8.1 Standard Request-Response

Used for:
- authentication
- profile operations
- admin actions
- support actions
- configuration reads

Flow:
- client -> Core Service -> PostgreSQL or Redis -> client response

### 8.2 Real-Time Updates

Used for:
- live trip status
- live driver location
- assignment updates
- admin monitoring updates
- urgent alerts

Flow:
- Core Service -> SignalR Service -> connected clients

### 8.3 Notification Event Flow

Used for:
- push notifications
- OTP-related async delivery handling
- WhatsApp notification delivery
- notification retry flow

Flow:
- Core Service -> Azure Event Bus -> notification handler -> external provider

### 8.4 Cache Usage

Used for:
- online driver state
- live trip snapshot
- OTP cooldown
- configuration cache
- pricing cache

Flow:
- Core Service reads Redis for fast state
- Core Service falls back to PostgreSQL when required

### 8.5 Lossless Real-Time and Failure Recovery Model

Real-time ride operation must tolerate network instability without silently losing business-critical state.

Mandatory safeguards:
- Core Service remains source of truth for ride state transitions
- every critical state transition is persisted in PostgreSQL before publish acknowledgment
- critical events are published to Redis channels for low-latency fan-out
- event payloads include event_id, trip_id, sequence_no, and timestamp for dedupe and ordering checks
- SignalR delivery uses reconnect-aware subscriptions and resumes streams from last acknowledged sequence where applicable
- client-side retry must use idempotency keys for Start Trip, End Trip, Cancel Trip, Force End, and No-Show updates
- failed notification dispatch must follow retry policy with dead-letter handling

Operational recovery requirements:
- if SignalR disconnects, clients auto-reconnect and request latest trip snapshot
- if publish delivery to real-time channel fails, Core Service keeps durable state and retries publish
- monitoring must alert on event publish failures, message lag, and abnormal reconnect rates

### 8.6 Failure Scenarios Matrix (Mandatory Reliability Contract)

| Scenario | Risk | Required Handling | Recovery Target (SLO) | Data Loss Allowed |
|---|---|---|---|---|
| Driver/Rider mobile network drop during trip | missed live updates, stale UI | WebSocket reconnect with backoff; fetch latest trip snapshot from Core Service; resume from last acknowledged sequence | reconnect and snapshot sync <= 10 seconds in normal conditions | none for critical trip state |
| SignalR node restart or disconnect | temporary broadcast interruption | clients auto-reconnect; SignalR resubscribes channels; replay latest trip state from Core Service | client stream recovery <= 15 seconds | none for critical trip state |
| Redis pub/sub lag or short outage | delayed live events | Core Service keeps durable event record and retries publish; monitoring alarm on lag threshold | lag back to normal <= 60 seconds after recovery | none for critical trip state |
| Core Service transient error on critical API (Start/End/Cancel/Force End) | duplicate or partial execution | idempotency key enforcement + transactional persistence + safe retry response | retry-safe response <= 5 seconds for healthy dependency state | none |
| PostgreSQL transient failover | write delay on source-of-truth updates | retry with circuit breaker; queue non-critical actions; block unsafe transitions until durable write confirms | critical write restore <= 30 seconds during failover window | none for committed writes |
| Notification provider outage (OTP/Push/WhatsApp) | undelivered notifications | retry policy with exponential backoff and dead-letter queue; operator alerting | first retry <= 30 seconds; dead-letter visibility immediate | allowed only for non-critical notifications after retry exhaustion |
| Admin portal disconnect during live monitoring | missed visual updates | reconnect and request event snapshot window; show sync status banner until current | monitor resync <= 15 seconds | none for persisted event timeline |
| Driver app sends duplicate requests due to poor network | duplicate state transitions | idempotency key + server-side dedupe by trip_id/action + sequence checks | duplicate rejection immediate, <= 2 seconds | none |

This matrix is a delivery contract between engineering, QA, and operations. Release readiness for production requires test evidence for each listed scenario.

## 9. Environment and Delivery Model

The solution will be managed across these environments:
- Dev
- Test
- UAT
- Prod

Delivery model:
- Azure DevOps for planning and source management
- Azure Pipelines for build and release automation
- Azure App Service for deployment targets
- Cloudflare storage for document storage and retrieval delivery

## 10. Development Plan

### Phase 1

Foundation:
- Azure DevOps project setup
- Azure App Service setup
- PostgreSQL and Redis setup
- Azure Key Vault setup
- environment setup for Dev, Test, UAT, and Prod
- Azure Pipeline baseline

### Phase 2

Application baseline:
- Rider App shell in MAUI
- Driver App shell in MAUI
- Admin Portal shell in React
- Core Service baseline
- SignalR Service baseline
- auth integration baseline

### Phase 3

Core implementation:
- module-wise Core Service development
- SignalR live integration
- Redis integration
- Azure Event Bus notification integration
- Blob Storage integration

### Phase 4

Operational hardening:
- secret management enforcement
- logging and monitoring
- notification retry handling
- deployment validation across all environments

### Phase 5

Scale readiness:
- improve module isolation in Core Service
- identify future split candidates
- scale SignalR and Redis independently when required

## 11. High-Level Architecture Diagram (Text View)

### 11.1 Layered Architecture

1. Client Layer
- Rider App (.NET MAUI)
- Driver App (.NET MAUI)
- Admin Portal (React)

2. Application Layer
- Core Service (ASP.NET Core Web API)
- SignalR Service (ASP.NET Core SignalR)

3. Data and Platform Layer
- PostgreSQL (transactional data)
- Redis (cache and fast operational state)
- Cloudflare storage (files and documents)
- Azure Key Vault (secrets)

4. Integration Layer
- Azure Event Bus (notification event handling)
- OTP provider
- WhatsApp provider
- FCM and APNs
- Google Maps Platform

### 11.2 End-to-End Technical Path

Client Apps -> Core Service (Azure App Service) -> PostgreSQL/Redis/Cloudflare Storage

Core Service -> SignalR Service -> Live updates to Rider/Driver/Admin

Core Service -> Azure Event Bus -> Notification Handler -> OTP/WhatsApp/Push Providers

Core Service -> Google Maps Platform for route and ETA operations

## 12. Clear Architecture Summary

Rekla will be built using:
- .NET MAUI for Rider and Driver apps
- React for Admin Portal
- ASP.NET Core Web API for Core Service
- ASP.NET Core SignalR for real-time communication
- PostgreSQL for transactional data
- Redis for cache and live operational state
- Azure Event Bus for notification event handling
- Cloudflare storage for files and documents
- Azure Key Vault for secret management
- Azure App Service for hosting
- Azure DevOps and Azure Pipelines for delivery
- Cloudflare storage for document storage and retrieval delivery
- Google Maps, OTP provider, WhatsApp provider, FCM, and APNs for external integrations

The initial architecture is a modular monolith plus a dedicated real-time service.

This gives:
- faster initial development
- simpler deployment and maintenance
- clearer technical understanding for BA and engineering teams
- a future path to scale without redesigning the full platform
