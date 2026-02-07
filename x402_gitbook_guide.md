---
title: "x402\\_gitbook\\_guide"
description: "In Simple Terms - X402 is like a \"pay-per-use\" vending machine for the internet. Instead of subscribing to services or creating accounts, you pay small amounts (micropayments) instantly to access what you need — like paying for a single API call, reading one article, or getting one piece of data."
---



# x402\_gitbook\_guide

## What is X402?

**In Simple Terms:** X402 is like a "pay-per-use" vending machine for the internet. Instead of subscribing to services or creating accounts, you pay small amounts (micropayments) instantly to access what you need — like paying for a single API call, reading one article, or getting one piece of data.

**Technical Definition:** X402 is an open-source, chain-agnostic payment protocol built on top of HTTP that activates the long-dormant HTTP status code **402 "Payment Required"** to enable instant, frictionless payments for web resources using blockchain technology (primarily stablecoins like USDC).

### Visual Overview

```mermaid
graph LR
    A[User/AI Agent] -->|Requests Resource| B[Server]
    B -->|HTTP 402: Payment Required| A
    A -->|Sends Payment + Request| B
    B -->|Verifies Payment| C[Blockchain]
    C -->|Confirms| B
    B -->|Delivers Resource| A
    
    style A fill:#4A90E2
    style B fill:#50C878
    style C fill:#FF6B6B
```

## Why X402 Matters

### The Problem X402 Solves

Traditional payment systems are not well suited for internet-native micropayments:

* High fees (credit cards charge 2–3% + $0.30)
* Slow settlement (2–3 business days)
* Account friction (accounts, repeated personal info, API keys)
* Not built for machines (AI agents cannot autonomously pay)

### The X402 Solution

* Zero protocol fees (only blockchain fees)
* Instant settlement (\~2 seconds)
* No accounts required (anonymous payments possible)
* Machine-first (AI agents can pay autonomously)
* Open standard (community-driven)

```mermaid
mindmap
  root((X402 Solution))
    Zero Protocol Fees
      Only minimal blockchain fees
      Makes micropayments viable
    Instant Settlement
      2 seconds typical
      Money goes directly to merchant
    No Accounts Needed
      Anonymous payments
      No personal information
      No API keys
    Machine-First
      AI agents can pay autonomously
      Programmatic transactions
    Open Standard
      No single company controls it
      Community-driven development
```

## How X402 Works

### The Payment Flow (Simple Version)

Think of it like this:

* You knock on a door (Request a resource)
* The door says "That'll be $0.10" (Server responds with 402 Payment Required)
* You slide money under the door (Client sends payment authorization)
* The door verifies the money and opens (Server verifies payment and delivers content)

### The Technical Flow

```mermaid
sequenceDiagram
    participant Client as Client/AI Agent
    participant Server as Web Server
    participant Facilitator as Payment Facilitator (Optional)
    participant Blockchain as Blockchain Network

    Client->>Server: GET /api/data
    Note over Server: No payment detected
    Server->>Client: HTTP 402 Payment Required
    Note over Server: Returns payment details:<br/>Amount, Wallet Address,<br/>Network, Asset

    Client->>Client: Sign payment authorization
    Note over Client: Creates EIP-712 signature<br/>with payment details

    Client->>Server: GET /api/data<br/>Header: X-PAYMENT
    Server->>Facilitator: Verify payment signature
    Facilitator->>Blockchain: Submit transaction
    Blockchain->>Facilitator: Transaction confirmed
    Facilitator->>Server: Payment verified ✓
    Server->>Client: HTTP 200 OK<br/>+ Requested Resource<br/>+ X-PAYMENT-RESPONSE
    
    Note over Client,Server: Total time: ~2 seconds
```

### Step-by-Step Breakdown

{% stepper %}
{% step %}
### Initial Request (No Payment)

Example request:

```http
GET /api/premium-data HTTP/1.1
Host: example.com
```

The server detects no payment and responds with HTTP 402.
{% endstep %}

{% step %}
### Server Responds with Payment Requirement

Example response:

```http
HTTP/1.1 402 Payment Required
Content-Type: application/json

{
  "maxAmountRequired": "0.10",
  "resource": "/api/premium-data",
  "description": "Access requires payment",
  "payTo": "0xABCD...1234",
  "asset": "0xA0b8...EB48",
  "network": "base-mainnet",
  "facilitator": "https://x402.org/facilitator"
}
```

The response includes amount, recipient, asset, network, and optional facilitator.
{% endstep %}

{% step %}
### Client Sends Payment Authorization

Client signs a payment authorization (e.g., EIP-712) and includes it in the request header:

```http
GET /api/premium-data HTTP/1.1
Host: example.com
X-PAYMENT: {
  "scheme": "exact",
  "version": "1.0",
  "data": {
    "signature": "0x1234...",
    "validAfter": "1234567890",
    "validBefore": "1234567899",
    ...
  }
}
```
{% endstep %}

{% step %}
### Server Verifies & Returns Resource

Once payment is verified (potentially via a facilitator broadcasting to the blockchain), server returns:

```http
HTTP/1.1 200 OK
X-PAYMENT-RESPONSE: {
  "transactionHash": "0xabc...def",
  "blockNumber": "12345678",
  "status": "confirmed"
}

{
  "data": "Your premium content here..."
}
```

Total time is typically \~2 seconds.
{% endstep %}
{% endstepper %}

## Key Features

* Zero Protocol Fees: X402 charges nothing; only blockchain fees apply.
* 2-Second Settlement: Fast confirmation compared to traditional payments.
* Chain Agnostic: Works with any blockchain network.
* HTTP Native: Uses standard HTTP semantics (402 Payment Required).
* Privacy-Focused: No account creation or PII required.
* AI-Ready: Designed for autonomous, programmatic payments.

Feature comparison and visual charts in original spec illustrate these differences.

## Technical Architecture

### Core Components

```mermaid
graph TB
    subgraph Client Side
        C1[Web Browser]
        C2[AI Agent]
        C3[Mobile App]
        C4[API Client]
    end
    
    subgraph X402 Middleware
        M1[Express.js Middleware]
        M2[Next.js Plugin]
        M3[Custom Integration]
    end
    
    subgraph Payment Processing
        P1[Payment Signature Creation]
        P2[EIP-712 Standard]
        P3[ERC-3009 Gasless Transfer]
    end
    
    subgraph Optional Facilitator
        F1[Payment Verification Service]
        F2[Transaction Broadcasting]
        F3[Settlement Confirmation]
    end
    
    subgraph Blockchain Layer
        B1[Ethereum Mainnet]
        B2[Base L2]
        B3[Polygon]
        B4[Other EVM Chains]
    end
    
    C1 --> M1
    C2 --> M2
    C3 --> M3
    C4 --> M1
    
    M1 --> P1
    M2 --> P1
    M3 --> P1
    
    P1 --> P2
    P2 --> P3
    
    P3 --> F1
    F1 --> F2
    F2 --> B1
    F2 --> B2
    F2 --> B3
    F2 --> B4
    
    B1 --> F3
    B2 --> F3
    B3 --> F3
    B4 --> F3
    
    style M1 fill:#4A90E2
    style M2 fill:#4A90E2
    style M3 fill:#4A90E2
    style F1 fill:#FFA500
    style F2 fill:#FFA500
    style F3 fill:#FFA500
```

### Payment Schemes

X402 supports extensible schemes. Examples:

* Exact Scheme: fixed amount (e.g., $0.10)
* Up To Scheme: pay up to a maximum based on usage
* Deferred Scheme: aggregate many small payments into batch settlements

```mermaid
graph LR
    subgraph Exact Scheme
        E1[Fixed Price<br/>$0.10]
        E2[Pay Once<br/>Get Access]
    end
    
    subgraph Up To Scheme
        U1[Max Price<br/>Up to $1.00]
        U2[Pay for<br/>Actual Usage]
    end
    
    subgraph Deferred Scheme
        D1[Batch Payments<br/>100 requests]
        D2[Single Settlement<br/>Once per day]
    end
    
    E1 --> E2
    U1 --> U2
    D1 --> D2
    
    style E1 fill:#50C878
    style U1 fill:#4A90E2
    style D1 fill:#FFA500
```

### Security Model

* EIP-712 Signatures for structured data signing
* ERC-3009 Gasless Transfers to avoid requiring ETH for gas
* Trust-minimizing: no intermediary can move funds without authorization
* Open source for community auditing

## Use Cases

* AI agent payments: autonomous agents pay per API call or tool access
* Content monetization: pay-per-article instead of subscriptions
* API monetization: pay-per-request pricing
* Web crawling & data access: charge crawlers per page
* Compute & storage services: pay-for-usage (GPU, storage, inference)

Real-world integrations and ecosystem participants are noted in the original spec.

## Infrastructure & Providers

### Core Infrastructure

* Coinbase (Protocol Creator): https://www.coinbase.com/developer-platform
* Cloudflare (Major Backer): https://www.cloudflare.com
* X402 Foundation (Governance) — established July 2025

### Payment Facilitators

Facilitators are optional services to verify payments and broadcast transactions. Official facilitator: https://x402.org/facilitator

### Blockchain Networks Supported

Commonly used networks include Base, Polygon, Ethereum Mainnet, Arbitrum, Optimism. Base is highlighted as most popular for low fees and fast settlement.

### Developer Tools & Libraries

Official libraries (npm packages shown in original content):

```bash
npm install x402-express    # Express.js middleware
npm install x402-client     # JavaScript/TypeScript client
npm install x402-react      # React hooks
```

Language support: JavaScript/TypeScript (full), Python/Go community libraries, Rust in development, and any HTTP-capable language can implement from the spec.

## Implementation Guide

### For Developers: Adding X402 to Your Service

Basic Implementation (Express.js)

{% stepper %}
{% step %}
### Install the middleware

```bash
npm install x402-express dotenv
```
{% endstep %}

{% step %}
### Add to your server (1 line to enable)

```javascript
import express from 'express';
import { paymentMiddleware } from 'x402-express';

const app = express();

// Single line to enable X402 payments!
app.use(
  paymentMiddleware(
    "0xYourWalletAddress",
    {
      "GET /api/premium-data": { price: "$0.10", network: "base-mainnet" },
      "POST /api/ai-service": { price: "$0.50", network: "base-mainnet" }
    },
    { url: "https://x402.org/facilitator" }
  )
);

// Your regular endpoints
app.get('/api/premium-data', (req, res) => {
  res.json({ data: "Premium content here!" });
});

app.listen(3000);
```

That's it — your API now requires payment.
{% endstep %}
{% endstepper %}

### For Non-Developers: What You Need to Know

As a Service Provider (Merchant):

1. Get a crypto wallet (Coinbase Wallet, MetaMask, etc.)
2. Get some testnet tokens for testing (Base Sepolia, etc.)
3. Hire a developer and share this guide
4. Set your prices
5. Deploy and start earning

As a Service Consumer (User/AI Agent):

1. Get a crypto wallet with USDC
2. Use X402-enabled clients (many AI agents support this)
3. Browse services (no account needed)
4. Pay as you go — automatic micropayments

### Integration Checklist

```mermaid
graph LR
    A[✓ Crypto Wallet] --> B[✓ Testnet Tokens]
    B --> C[✓ Install Middleware]
    C --> D[✓ Configure Endpoints]
    D --> E[✓ Test Payments]
    E --> F[✓ Deploy to Mainnet]
    F --> G[✓ Monitor Transactions]
    
    style A fill:#50C878
    style B fill:#50C878
    style C fill:#50C878
    style D fill:#50C878
    style E fill:#50C878
    style F fill:#50C878
    style G fill:#50C878
```

## The Future of X402

### Current Status (October 2025)

* Protocol v1.0 released
* Reference implementations available
* Major backing from Coinbase and Cloudflare
* X402 Foundation established
* Deferred payment scheme and additional schemes in development

Roadmap outlines expansion of scheme support, integration with traditional payment rails, and broader adoption.

### Vision

X402 aims to become a native payment layer for the internet — as fundamental as HTTP, DNS, and TLS — enabling seamless micropayments, autonomous AI commerce, and new business models.

## Key Takeaways

* X402 is pay-per-use for the internet (no subscriptions, no accounts)
* Fast: payments settle in \~2 seconds
* Cheap: only blockchain fees (\~$0.01 on L2s)
* Open and chain-agnostic
* Designed for machine-to-machine autonomous payments
* Simple to integrate for developers (often one middleware line)

## Resources

Official Links:

* Protocol Website: https://www.x402.org
* Documentation: https://x402.gitbook.io/x402
* GitHub Repository: https://github.com/coinbase/x402
* Whitepaper: https://www.x402.org/x402-whitepaper.pdf
* Coinbase Developer Platform: https://www.coinbase.com/developer-platform

Developer resources and infrastructure provider links are listed in the original content (including QuickNode, Cloudflare, thirdweb Nebula).

Community:

* GitHub Discussions: https://github.com/coinbase/x402/discussions
* Contributions: See CONTRIBUTING.md in the GitHub repo
* Cloudflare Contact: x402@cloudflare.com

## Glossary

| Term           | Definition                                                                        |
| -------------- | --------------------------------------------------------------------------------- |
| HTTP 402       | A rarely-used HTTP status code meaning "Payment Required" - now activated by X402 |
| Stablecoin     | A cryptocurrency pegged to a stable asset (e.g., USDC = $1 USD)                   |
| Layer 2 (L2)   | A secondary blockchain network built on Ethereum for faster, cheaper transactions |
| EIP-712        | Ethereum standard for signing structured data in a human-readable way             |
| ERC-3009       | Standard enabling gasless token transfers (user doesn't pay gas fees)             |
| Facilitator    | Optional service that helps verify payments and broadcast to blockchain           |
| Micropayment   | Very small payment (e.g., $0.001 - $1.00)                                         |
| Chain Agnostic | Works with any blockchain network                                                 |
| Middleware     | Software that sits between your application and incoming requests                 |
| Base           | Ethereum Layer 2 network built by Coinbase                                        |
| USDC           | USD Coin, a stablecoin commonly used with X402                                    |

## FAQs

<details>

<summary>Do I need cryptocurrency to use X402?</summary>

As a user, yes — you need a small amount of stablecoin (like USDC) to pay. As a merchant, you receive stablecoins which you can convert to regular currency.

</details>

<details>

<summary>Is this only for crypto people?</summary>

No. The goal is to make it invisible to end users. Users should see a simple "Pay $0.10" and click — the crypto happens in the background.

</details>

<details>

<summary>What about credit cards?</summary>

Future versions of X402 may support traditional payment rails, but currently it's crypto-based for speed and low fees.

</details>

<details>

<summary>Can my AI assistant use this?</summary>

Yes. Enabling AI agents to autonomously pay for services is a primary use case.

</details>

<details>

<summary>How much does it cost?</summary>

The protocol itself is free. You only pay blockchain transaction fees (typically \~$0.01 on L2s) and whatever price the merchant sets.

</details>

<details>

<summary>Is it secure?</summary>

Yes. X402 uses industry-standard cryptography (EIP-712, ERC-3009) and open-source code for community auditing.

</details>

<details>

<summary>Who controls X402?</summary>

Governance is handled by the X402 Foundation (Coinbase + Cloudflare + community). The protocol is intended to be open and neutral.

</details>

## Conclusion

X402 represents a shift toward native, internet-scale micropayments: autonomous AI commerce, fair creator monetization, instant global payments, and pay-per-use monetization models. It enables payments to be as seamless as loading a webpage.

Document Version: 1.0\
Last Updated: October 2025\
License: Apache 2.0 (Open Source)
