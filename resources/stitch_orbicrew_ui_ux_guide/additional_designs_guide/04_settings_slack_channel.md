# Brief: Settings → Connected Channels — add Slack

## Where this lives
Same page as `orbicrew_settings` → "Connected Channels" tab (Telegram / Discord / WhatsApp rows). This is a small addition, not a new screen.

## Why it's needed
Product decision: Slack should be a chat channel adapter like Telegram/Discord/WhatsApp (agents message users directly in Slack), not just a background tool integration. The existing settings mockup already has the right pattern for exactly this — it's just missing a Slack row.

## What to add
A fourth row in the "Connected Channels" list, identical in style to the existing three (icon in a bordered square, name + status dot + status text, "Configure"/"Disconnect" when connected or a single "Connect" button when not):

- **Icon**: Slack's recognizable mark (simplified/monochrome to match the other channel icons' line-art style).
- **Name**: "Slack"
- **Default state**: "Not Connected" (same gray dot + "Connect" button treatment as the WhatsApp row today).

No other changes needed — reuse the row component exactly as designed for Telegram/Discord/WhatsApp.
