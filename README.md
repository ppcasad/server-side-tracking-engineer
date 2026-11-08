# Server-Side Tracking Engineer (AI Agent Skill)

An end-to-end AI agent skill for deploying, configuring, and maintaining **Server-Side Google Tag Manager (sGTM)** on **Google Cloud Run** ($0/month scale-to-zero architecture), complete with custom first-party subdomain mapping, **GA4** server routing, and **Google Ads Enhanced Conversions** for WordPress/Elementor.

## ✨ What This Skill Covers

- **Google Cloud Run Architecture:** Ready-to-run Cloud Shell commands for both Production (`sgtm-[site]`) and Dedicated Preview (`sgtm-[site]-preview`) containers.
- **First-Party DNS Mapping:** Safe subdomain (`data.[domain]`) CNAME configuration and `/healthy` endpoint verification.
- **Web GTM Configuration:** Conversion Linker, Google Tag (`server_container_url`), custom Elementor form listener (`generate_lead` with `user_data`), and phone/email click triggers.
- **Server GTM Configuration:** GA4 event forwarding and Google Ads Enhanced Conversion tracking (`generate_lead`, `phone_click`, `email_click`).

## 🚀 Installation

Clone this repository directly into your agent's `skills` directory:

```bash
git clone https://github.com/ppcasad/server-side-tracking-engineer.git
```

Or manually download [`SKILL.md`](./SKILL.md) and place it inside:
- `~/.gemini/config/skills/server-side-tracking-engineer/SKILL.md`
