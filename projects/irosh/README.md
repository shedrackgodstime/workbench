---
title: irosh - Secure P2P SSH
summary: A standalone peer-to-peer SSH tool and library written in Rust, built on Iroh to eliminate public IPs, open ports, and centralized accounts.
type: project
status: active
tags:
  - Rust
  - P2P
  - SSH
  - Networking
  - Iroh
  - Security
started: "2025-01"
updated: "2026-08"
context: Built out of a need for frictionless remote access without relying on public IP addresses, router port-forwarding, or centralized VPN accounts like Tailscale.
source: https://github.com/shedrackgodstime/irosh
---

## Overview & Motivation

Accessing remote machines traditionally requires static public IPs, router port forwarding, or centralized VPN accounts (like Tailscale). On dynamic cellular links, carrier-grade NATs (CGNAT), or restricted firewalls, standard SSH fails without external bastion infrastructure.

**irosh** solves this by combining a pure-Rust SSH implementation (\`russh\`) with **Iroh**'s QUIC-based P2P networking. It allows two machines to establish a direct, encrypted terminal session behind firewalls using temporary 3-word pairing codes, with zero open inbound ports and no third-party accounts.

---

## Architecture & Data Flow

irosh is built as a **Fat Library** (\`irosh\` crate) with a lightweight CLI frontend (\`cli/\`). The core engine handles transport orchestration, SSH handshakes, PTY lifecycle, and out-of-band file transfers.

\`\`\`mermaid
graph TD
    subgraph "Client"
        CLI[irosh CLI] -->|Stdin / TTY| Engine[Client Engine]
        Engine -->|SSH Channel Protocol| RusshC[russh Client]
    end

    subgraph "P2P Network (Iroh + Pkarr)"
        RusshC -->|QUIC Streams / UDP Hole Punching| Transport[Iroh Transport]
        DHT[Pkarr DHT] -.->|Resolve 3-Word Code to NodeId| Transport
    end

    subgraph "Host"
        Transport -->|Authenticated Stream| RusshS[russh Server]
        RusshS -->|PTY Allocation| Shell[Shell / Process]
        Auth[Pluggable Auth / Argon2] -->|Verify Keys| RusshS
    end
\`\`\`

---

## Core Subsystems

- **P2P Transport (\`irosh::transport\`):** Built on Iroh's QUIC implementation for automatic UDP hole punching and DERP relay fallback across restrictive NATs.
- **Discovery (\`irosh::discovery\`):** Uses Pkarr to publish Ed25519 node identities to the Mainline DHT, generating ephemeral 3-word pairing phrases (e.g. \`apple-pie-sunset\`).
- **Standalone SSH Server (\`irosh::server\`):** Native SSH server in pure Rust via \`russh\` and \`portable-pty\`. Operates without OpenSSH daemon dependencies.
- **Escape Toolkit (\`irosh::session\`):** Out-of-band multiplexed escape sequences (\`~.\`, \`~put\`, \`~get\`, \`~C\`) for inline file transfers and diagnostics over the active connection.

---

## Library Usage

The core engine can be embedded directly into custom Rust services:

\`\`\`rust
use irosh::{Server, ServerConfig};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let server = Server::new(ServerConfig::default()).await?;
    println!("Node ID: {}", server.node_id());
    
    server.run().await?;
    Ok(())
}
\`\`\`

---

## CLI Quick Start

### 1. Host
\`\`\`bash
irosh system install    # Run background service
irosh wormhole          # Generate 3-word pairing code (e.g. apple-pie-sunset)
\`\`\`

### 2. Client
\`\`\`bash
irosh apple-pie-sunset  # Connect by name
\`\`\`

### 3. Integrated File Transfers
- \`~put <local> [remote]\` — upload file or directory to peer
- \`~get <remote> [local]\` — download file or directory from peer
- \`~.\` — disconnect immediately

---

## Technical Observations

1. **NAT Traversal:** Direct UDP hole punching establishes in under 200ms on full-cone and restricted NATs. Symmetric cellular connections fall back cleanly to encrypted relays without dropping session state.
2. **Terminal IO Multiplexing:** Handling dynamic window resizing (\`SIGWINCH\`) and raw mode IO concurrently with high-throughput file chunk streams required clean channel separation inside the SSH multiplexer.
3. **Memory Zeroization:** Sensitive authentication material (passwords, private keys) is wrapped in \`secrecy\` to zeroize buffers upon drop.
