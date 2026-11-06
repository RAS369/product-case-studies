# MORSAL: Product Case Study

*A retrospective product analysis of my graduation project.*

**Role:** backend + system design · **Team:** 6 · **Timeline:** Oct 2024 – Jun 2025 · **Stack:** ESP32, .NET, SignalR, React · **Outcome:** working prototype, graded A+

## Summary

We built a working  prototype that lets deaf-blind people message anyone remotely. They type Morse code on a 7-button device and feel replies as vibrations, while the other person uses a normal web chat. The device costs about $12 in components; refreshable Braille displays cost $800 to over $8,000 ([Perkins](https://www.perkins.org/resource/overview-braille-devices/)).

The engineering risk is retired. The adoption risk is not: no deaf-blind user has tested the device. This analysis is about that gap.

**My role on the build:** I designed the database schema, built the messages API and part of the SignalR hub, and produced all system design diagrams (ERD, use case, sequence, activity). The team had 6 students with equal roles, and scope decisions were shared.

**My role in this document:** the user and market analysis, the alternatives review and the v2 plan are my own work, written after the project.

## Problem

Deaf-blind people have no affordable way to communicate remotely. They usually depend on an interpreter who is physically present, which costs them privacy and independence.

The main alternative, a refreshable Braille display, has three gaps:

- **Cost:** $800 to over $8,000 ([Perkins](https://www.perkins.org/resource/overview-braille-devices/)), out of reach for most users in Egypt.
- **Skill:** it requires Braille literacy, which many deaf-blind people never had the chance to learn. MORSAL does not remove this barrier: Morse is also a skill to learn. Whether it is easier to learn than Braille is the riskiest assumption, which v2 tests.
- **Direction:** it is built for reading, with no practical two-way, real-time messaging between people in different places.

**Size of the problem:** the World Federation of the DeafBlind estimates that 0.2% of the world's population lives with severe deafblindness, and about 2% including milder forms ([WFDB](https://wfdb.eu/?p=2291)). Applied to Egypt's 109.5 million people ([CAPMAS, Sept 2026](https://www.egypttoday.com/Article/1/149579/Egypt%E2%80%99s-domestic-population-reaches-109-5-Million-CAPMAS)), that is roughly 220,000 people with severe deafblindness. No Egypt-specific count exists, so treat this as an estimate.

## Target user and buyer

**First segment (hypothesis):** adults who became deaf-blind after learning to read and write. Morse code means spelling words letter by letter, so people born deaf-blind who never acquired written language are not a realistic first segment.

**Secondary user:** the sighted, hearing contact (family, friend, caregiver) who chats through the web app. A facilitator sets up the deaf-blind user's account and links the device once.

**Buyer (hypothesis):** not the user. Deaf-blind people are more likely to be poor and unemployed ([WFDB](https://www.internationaldisabilityalliance.org/wfdb-global-report)), so the realistic route is B2B2C: NGOs, special-needs schools and government disability programmes buy and distribute devices. The price of one Braille display could fund dozens of MORSAL devices.

**Contradictions we accepted in v1:**

- The interface is English-only, but most users in Egypt read Arabic.
- The goal is independence, but setup needs a sighted facilitator.

## Goals and success criteria

The project set goals, not formal success metrics. Below, each goal is paired with the check we actually ran. In hindsight, defining user-outcome metrics up front is the biggest thing I would change.

| Goal | How we checked it |
| --- | --- |
| Two-way remote messaging for BD users | A BD device and a web user exchange messages in real time over the internet |
| Fully usable without sight or hearing | Every BD action (compose, send, read, search, navigate chats) works through buttons and vibration only |
| Far cheaper than Braille hardware | Component cost of the final device |
| Reliable enough to depend on | Messages arrive without loss or noticeable delay in repeated testing |

## Alternatives, in hindsight

The team did not formally compare alternatives; we treated Morse and custom hardware as the premise. This is my retrospective comparison.

| Alternative | Its strength | Its weakness for this user |
| --- | --- | --- |
| Phone app (Morse as vibration, taps as input) | No extra hardware; most people already own a phone | The screen itself is not tactile: no buttons to find by feel, so dot vs dash, navigation and search all depend on gestures that are easy to get wrong |
| Refreshable Braille display | Mature, fast for fluent Braille readers | $800 to over $8,000; needs Braille literacy; built for reading, not two-way chat |
| Human interpreter | Richest communication, no learning curve | Must be physically present; costs privacy and independence |
| MORSAL device | About $12; physical buttons findable by touch; two-way and remote | Requires learning Morse; English-only; needs Wi-Fi |

The phone app is the strongest competitor, and we never tested against it. v2 should pilot both before investing in more hardware.

## Solution and key features

MORSAL has two parts that share one account system: a handheld device for the BD user and a WhatsApp-style web app for everyone else.

**How a message travels**

1. The BD user types dots, dashes and spaces on the device (ESP32, Wi-Fi).
2. On Short Enter, the device sends the buffer to the backend (.NET), which translates Morse to text and stores it.
3. SignalR pushes the message in real time to the recipient's React chat screen.
4. Replies go the other way: text is converted to Morse and played as vibration pulses (dot about 200 ms, dash about 600 ms).

**Device interaction:** 7 buttons (dot, dash, space, backspace, enter, up, down). Short and long presses map to different actions, so 7 buttons cover about 14 actions:

- Compose: dot, dash, space; Short Backspace deletes a character; Short Enter sends.
- Read: Long Enter plays the selected chat's name or message as Morse vibration.
- Navigate: Up/Down move through chats or messages; Long Space enters a chat; Long Backspace exits.
- Search: type a username in Morse. One match opens the chat directly; several matches can be browsed with Up/Down.

**Web app:** a WhatsApp-style real-time chat. Deaf-blind accounts register with a unique Device ID that links the hardware to the user.

## Scope decisions and trade-offs

We scoped v1 to the smallest product that proves a BD user can hold a two-way remote conversation alone. The team made these calls together under a fixed academic deadline.

**Cut from v1 for time and resources:** group chat, multimedia (voice, images), a mobile app, offline messaging, and Arabic support.

**Trade-offs we made**

- **7 buttons with short/long presses, rather than a single Morse key or one button per action.** A single key is simpler to build but makes navigation very hard. One button per action needs a larger device. Short/long presses keep the device small at the cost of more modes to learn.
- **Morse over Braille as the core premise.** Morse needs only one vibration motor and a few buttons, which drives the low cost, but it adds a learning curve. We took this on as a premise rather than testing it, and that is the assumption I would validate first.
- **Simple power design for a prototype.** A 9V battery with a linear regulator was easy to build but showed voltage drops when the motor fired. That was acceptable for a prototype, not for a product.
- **Wi-Fi.** Using Wi-Fi gave real remote messaging with no phone pairing, but it makes the device useless without internet.

## Testing and validation

All testing was internal. We had no access to deaf-blind users within the project's time and permissions, so the design rests on published research (for example, Tan et al., 1997, on reading Morse code through vibration), not on user interviews.

- **Blindfolded team testing drove the biggest design change.** At 100 ms dots and 300 ms dashes, the pulses were hard to tell apart by touch, so we doubled them to about 200 ms and 600 ms.
- **Demo:** an end-to-end demo was recorded and presented to the examiners.
- SignalR latency under 500 ms per message; 50+ consecutive Morse messages sent without loss.
- Each of the 7 buttons tested over 100+ presses; false double inputs fixed with hardware and firmware debouncing; the PCB went through 4+ revisions.

## Results

We delivered a working end-to-end prototype, graded A+. A deaf-blind user can complete every core task on the device alone: find a contact, read messages, reply and navigate.

| Item | Cost (USD) |
| --- | --- |
| Final device components (ESP32, motor, buttons, PCB, power) | \~$12 |
| Total project cost, including breadboard prototype and assembly labour | $34.92 |
| Refreshable Braille display, for comparison ([Perkins](https://www.perkins.org/resource/overview-braille-devices/)) | $800 to $8,000+ |

**Riskiest assumption, still open:** that deaf-blind people can learn Morse input and would choose this device over the alternatives. No user has tested it.

## What I'd build in v2 and how I'd measure it

v2 starts with discovery, not features. The goal is to test the riskiest assumption before writing new code.

**Step 1: Validate (before writing new code)**

- Partner with a deaf-blind support organisation or special-needs school (these are also the likely buyers). Interview 5–8 users from the first segment, plus their caregivers, about how they communicate today.
- Run a 2-week pilot with 3–5 users. Half start on the device and half on a phone-app version, then they switch.
- Decision rule: if most users cannot send a short message unaided by week 2, change the input method before changing anything else.

**Step 2: Prioritise (MoSCoW, ranked on pilot findings)**

| Priority | Feature | Why |
| --- | --- | --- |
| Must | Adjustable vibration speed and strength | Tactile sensitivity varies between users; fixed timing was our main usability fix |
| Must | Arabic Morse support | Needed for the Egyptian market |
| Should | Emergency button sequence that alerts a caregiver | High value for safety and for caregivers' trust |
| Could | Adaptive timing that learns each user's typing speed | Valuable once enough usage data exists |
| Won't (yet) | Wearable form factor | Hardware redesign; only after adoption is proven |

**Step 3: Measure**

**North Star:** conversations completed without a human intermediary, per user per week. It measures independence, which is the problem we set out to solve. Message volume alone could rise without anyone becoming more independent.

The thresholds below are starting hypotheses, to be recalibrated after the pilot. They are not results.

| Metric | What it tells us | Starting hypothesis |
| --- | --- | --- |
| Sessions until a user sends a 5-word message unaided | Learnability | ≤ 5 sessions |
| Task success: find contact, read, reply | Usability | Most users, unaided |
| Device vs phone app: preference and error rate | Which form factor to invest in | No assumption; this is what the pilot decides |
| Users still active after 30 days | Retention | Majority of pilot users |
| Message delivery time, device to screen | Guardrail | Under 1 second |

## What I learned

We validated feasibility and assumed desirability. The decisions that shaped the product (Morse, custom hardware, English first) came from what we could build, not from users. Next time I would test the riskiest assumption with real users before building anything.

## Sources

- MORSAL graduation project report, Helwan University, June 2025: costs, test figures, requirements
- [Perkins School for the Blind: overview of Braille devices](https://www.perkins.org/resource/overview-braille-devices/)
- [World Federation of the DeafBlind: global report, 2018](https://wfdb.eu/?p=2291)
- [International Disability Alliance: WFDB global report summary](https://www.internationaldisabilityalliance.org/wfdb-global-report)
- [Egypt Today: CAPMAS population figure, Sept 2026](https://www.egypttoday.com/Article/1/149579/Egypt%E2%80%99s-domestic-population-reaches-109-5-Million-CAPMAS)
- Tan, H.Z. et al. (1997), "Reception of Morse code through motional, vibrotactile, and auditory stimulation", *Perception & Psychophysics* 59, 1004–1017. [doi:10.3758/BF03205516](https://doi.org/10.3758/BF03205516)
