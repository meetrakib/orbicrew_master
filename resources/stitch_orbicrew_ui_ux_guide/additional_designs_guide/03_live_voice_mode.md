# Brief: Live voice mode (chat GUI)

## Where this lives
Extends `orbicrew_message_agents` (the "Message your AI team..." task chat). That mockup already has a mic icon in the input bar, but it's only for one-shot dictation (record → transcribe into the text box). This brief is for a new, separate **continuous/live voice mode** — think ChatGPT's or Gemini's "voice mode": always-listening, interruptible, spoken back in real time, no typing.

## Why it's needed
Product decision: live conversational voice should be available on the plain chat GUI (Standard Dashboard), not only inside the Orbit View animated-office avatar. Distinct from the existing one-shot mic button, which stays as-is for people who just want to dictate a message.

## Entry point
Add a second icon next to the existing mic icon in the message input bar (e.g., a "waveform" or "phone/headset"-style icon) labeled on hover as "Live voice mode."

## Screen/overlay to design
A full-screen (or large centered modal) takeover, replacing the chat thread view while active:

1. **Center stage**: a large circular audio-reactive visual (waveform, pulsing orb, or similar — matches the brand's violet accent) that visibly animates while the agent is speaking, and shows a distinct "listening" state while the user is speaking.
2. **Status text** under/near the visual: short states like "Listening…", "Thinking…", "Speaking…"
3. **Live captions** (optional but recommended): a single line or two of live transcript text of what's currently being said, so the interaction isn't audio-only for accessibility.
4. **Controls** (bottom bar, minimal):
   - Mute/unmute mic button.
   - Interrupt affordance — clear that the user can start talking at any time to cut off the agent's spoken response (barge-in), not a separate button necessarily, but the state should reflect it (e.g., agent's waveform stops instantly on user speech).
   - End call / exit button — returns to the normal typed chat thread, preserving the conversation.
5. **Which agent is speaking**: show the active agent's name/avatar (small, near the top) — relevant since a live voice session could involve the Master Agent, and it should be clear which agent is on the "line."

## Style
Darker/more focused surface than the rest of the app is acceptable here (similar precedent: full-screen voice modes typically dim surrounding chrome) — but keep the brand's Deep Violet accent as the primary color for the active/speaking state, and keep typography consistent with the rest of the product (Bricolage Grotesque / Hanken Grotesk). Should work in both light and dark theme.
