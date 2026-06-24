---
layout: post
title: "Escaping Rented Land: Routing Agentic Computation Over Nostr"
date: 2026-06-24
author: "Khushvendra Singh"
categories: [ Stories ]
image: assets/images/blog_content/nostr_mcp.png
---

I’ve been spending a lot of time lately digging deep into decentralized systems, and recently, a realization hit me pretty hard: even on the modern internet, we’re mostly just building on someone else’s property. 🏰

Think about it. We hand our code to Amazon, our conversations to Google, and our APIs to Azure. Everything "useful" we've built is sitting on rented land. We play by their rules, and we pay rent forever. For a long time, the idea of real, large-scale decentralized systems felt like a complete pipe dream.

Then I found **Nostr**.

At first glance, Nostr looks like just another decentralized social network. **It's not.** It's a bare-bones, unapologetically simple message-passing protocol. And a message can carry *anything*. If Nostr can route a text post, it can route a request to run code. You can execute a function anywhere on the planet without asking a middleman for permission.

This is exactly what we’re solving at [ContextVM](https://github.com/context-vm), the project I’m contributing to for **Summer of Bitcoin 2026**.

---

## 🛰️ Enter ContextVM

ContextVM sits right at the intersection of Nostr and the **Model Context Protocol (MCP)**. People still think MCP is just an "AI agent" thing, but it’s really a protocol for calling remote functions. 

ContextVM lets you take a script or service running on your own local hardware and expose it as a secure, globally accessible API. It gets routed over Nostr and paid for over the Lightning Network. 
- **No AWS.** 
- **No SaaS middlemen.** 
- **No surprise cloud bills.**

I'm Khushvendra Singh, and for the last six weeks, I've been building the transport primitives and the reference client to make this actually work.

---

## 🛠️ What I've Built So Far

When you're building decentralized infrastructure, the "protocol work" isn't just writing new logic from scratch—it's meticulous specification modeling, threat analysis, and edge-case hunting. Every line of code is scrutinized, every protocol boundary carefully defined. Here's what I've shipped in the first half of the program:

### 🌊 CEP-41 (Open-Ended Streams)
Co-authored the spec and implemented the Nostr transport layer for open-ended streaming. Before a single line of feature code merged, we trapped and fixed six critical edge cases, including outbound buffer limits and a nasty stale-chunk bug that could wedge sessions indefinitely. The protocol had to be bulletproof from day one.

### 🏗️ Transport Layer Refactor
Stripped down and modularized the core Nostr transport layer, splitting two monolithic 1,500+ line files into clean, protocol-scoped coordinators and dispatchers. We kept strict backward compatibility, passing all 254 tests without breaking the public API. Clean architecture and zero regressions. 

### 💻 Web Chat Client (Phase 1)
Built the Svelte 5 production web client to prove the protocol works end-to-end. It features Bring-Your-Own-Token (BYOT) management, automated model rotation, and streaming markdown rendering. This is the visible interface to the whole system—the gateway between browsers and sovereign hardware.

### 🤖 MCP Agentic Execution Loop & CEP-8
Built the client-side orchestration layer with abort safety, permissioned approval tiers for sensitive tools, and multi-server registry using pubkey-based collision disambiguation. Implemented the `-32042`/`-32043` payment-required lifecycle (CEP-8) in the TypeScript SDK. This is the scaffolding for pay-per-call agentic execution over Nostr.

---

## 🤯 The Hardest Problem I Faced

By far, the most maddening issue I ran into was inside the web client, specifically dealing with **Svelte 5’s `$state()` reactivity** when wrapping complex native objects.

I needed to manage `AbortController` instances and the OpenAI SDK client inside our reactive state. It seemed harmless enough. But because Svelte 5 wraps `$state` objects in a `Proxy`, instances relying on internal private slots or internal state machines would silently corrupt. 🫠

The failure mode was the worst kind of bug: **zero console errors, no thrown exceptions.** Just a chat stream that would permanently wedge and die. I spent hours chasing ghosts—debugging IndexedDB cloning errors and stream teardown logic—before finally tracing it to the proxy boundary itself. 

**The fix?** A strict architectural boundary: isolating instances with internal identity inside plain `let` bindings outside the reactive graph, ensuring only pure, serializable data ever enters `$state`. Lesson learned: don't let your proxies touch your native instances.

---

## 🧠 Lessons from the Trenches

**1. Idempotency is terrifying when money is involved.** 💸

Implementing CEP-8's payment gating taught me exactly how fragile idempotency becomes once a network retry involves actual financial transactions. You need an absolute guarantee that a paid authorization maps to exactly *one* execution of a specific tool invocation. I ended up implementing a canonical-invocation-identity scheme (applying RFC 8785 JCS to the method, parameters, pubkey, and request ID before hashing via SHA-256). It isn't just metadata; it's the cryptographic safety boundary preventing double-charging during network drops.

**2. Maintainer pushback is a feature, not a bug.**

During the transport refactor, I initially extracted obvious standalone helpers just to reduce file line counts. My mentor (@gzuuus) immediately pushed back. He showed me that extracting code purely for size reduction actually fragments execution flow and obscures module responsibility. We established a core principle: only extract along a genuine protocol or lifecycle boundary. That back-and-forth completely elevated how I approach system architecture.

---

## 🚀 What's Next

Because ContextVM moves fast, I'm operating dynamically against the live production roadmap rather than an isolated timeline.

My immediate goal for the next few weeks is to bridge the gap between the SDK and the UI. I will be wiring the web client to consume the explicit gating mode (`site#32`). By the final evaluation, a user will be able to prompt the chat client, trigger a paid MCP tool on a remote server, see the Lightning invoice, pay it, and watch the SDK transparently complete the invocation.

Once that end-to-end flow is live, we'll hit the roadmap board and pull the next high-priority protocol feature. The cloud isn't going away anytime soon, but we are finally building a viable exit route.

---

## Links to my work:

* [CEP-41 Specification Draft](https://github.com/ContextVM/contextvm-docs/pull/40) 
* [CEP-41 Stream Transport Implementation](https://github.com/ContextVM/sdk/pull/71) 
* [Core Transport Layer Refactor](https://github.com/ContextVM/sdk/pull/72) 
* [Core LLM Web Chat Integration](https://github.com/ContextVM/contextvm-site/pull/29)
* [MCP Agentic Orchestration Layer](https://github.com/ContextVM/contextvm-site/pull/31) 
* [CEP-8 Explicit Gating Lifecycle](https://github.com/ContextVM/sdk/pull/75) 
* [Web Client Payment-Gating UX](https://github.com/ContextVM/contextvm-site/issues/32) 
