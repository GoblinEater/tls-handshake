# TLS Handshake Visualizer

Interactive companion tool for Week 8 of *How Protocols Work, and How They Break* (Goblin Eaters Cyber Cavern). Covers OSI Layers 5 and 6, session establishment and presentation encryption, using TLS as the working example.

## What It Shows

A single TLS connection walked step by step, split visually into two bands:

- **L6 (Presentation)**, upper band: encryption and formatting work
- **L5 (Session)**, lower band: handshake negotiation and session establishment

Client and server each appear twice, once per band, connected by a faint dashed tether. Same host, two layers of work. A step where the connection moves from handshake to encrypted data is drawn as a literal upward transition between bands, the visual argument for the article's closing claim: TLS does L5 work and L6 work in one protocol, which is why it resists a single clean layer assignment.

## Modes

**Normal Handshake**
ClientHello, ServerHello and certificate, chain validation, Finished, then the rise into L6 and the first encrypted application data record. Six steps, no attacker.

**Downgrade Attack**
Demonstrates the POODLE-era failure mode. An on-path attacker forces the connection to fail until the client retries with a legacy-only offer (SSL 3.0), the server accepts it since it still supports the old protocol for compatibility, and the resulting CBC-mode cipher is shown as decryptable through a padding oracle. The dissector marks falsified fields with the same Sent Value / Honest Value split used in the ARP visualizer.

**Forged Certificate**
Demonstrates a CA compromise (the DigiNotar/Comodo class of incident). An on-path attacker presents a certificate with the correct domain name but a fraudulently signed issuer chain. The chain validates because the client only checks that a trusted CA signed it, not whether that CA was itself compromised at the time. Ends in a two-hop MITM at L6: client-to-attacker and attacker-to-real-server, both genuinely encrypted, neither the party the client believes it is.

## Controls

- **Mode toggle** (top right): switches between the three scenarios above and resets the walkthrough
- **Next / Prev**: step through the handshake one message at a time
- **Reset**: return to the beginning of the current mode
- **Auto Play**: advances automatically every 2.6 seconds; toggles to Pause

## Reading the Dissector

The bottom-left panel shows the handshake message or TLS record for the current step.

- **Normal steps**: a single Value column, no falsified fields
- **Attack steps**: a second Honest Value column appears. Rows flagged with a red **FALSIFIED** badge show what was actually sent; the Honest Value column shows what should have been true, or would have been true absent the attack

The session strip beneath the step panel surfaces state that persists across steps, such as chain validation results, the same role the ARP cache strip plays in the Week 2 tool.

## Architecture

Single self-contained HTML file, no external dependencies beyond the Google Fonts CDN (JetBrains Mono, Syne). Same step-engine pattern as the Week 2 ARP visualizer:

- `steps` array per mode, each entry describing the packet animation, dissector frame key, and step-panel copy
- `FRAMES` object holding the dissector content, keyed by frame name, referenced from each step
- Canvas-based node and packet rendering, redrawn every frame via `requestAnimationFrame`
- Auto-repeating packet animation for the current step (`startReplay`) so the motion doesn't go static while a reader is reading the explanation

Font sizing follows the project standard throughout: `rem`/`clamp()` only in CSS, base `font-size` set once on `html`, all type scales from there. Canvas-drawn text (`ctx.font`) uses raw pixel values since it's a canvas API call, not CSS, and isn't part of the page's type scale.

## Known Limitations

- Packet labels use emoji (lock, unlock, up-arrow) for compactness; not yet checked against the ARP tool's text-label convention for print or accessibility consistency
- Attacker node visibility logic keeps the attacker on-screen for the remainder of an attack sequence once introduced; not yet verified during Reset/Prev navigation edge cases
- No responsive/mobile layout pass yet; built and reviewed at desktop width only

## File

`tls-handshake-visualizer_v1.html`, single file, drop into `~/ubritsa/static/tools/` per the site's standard tool path.
