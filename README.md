# TrendScoop AI – Hyperlocal Civic News Digest

**Version:** 2.5  
**Owner:** Luv Saxena  
**Stack:** n8n, Tavily, Groq (Llama 3.3), Google Sheets, Gmail

TrendScoop AI is a production-grade hyperlocal civic news agent that sends a daily 8:00 AM email briefing for Mira Bhayandar residents. It searches local news sources, intelligently ranks stories by civic impact, generates a professional newsroom-style digest using a guarded LLM, and delivers personalized HTML emails to subscribers from a Google Sheet – all fully automated.

⚠️ **Commercial product** – contact Luv Saxena for licensing, multi-city deployments, or custom adaptations.

---

## What it does (end-to-end)

1. **Daily trigger** at 8:00 AM Asia/Kolkata via n8n Schedule Trigger
2. **Tavily search** for Mira Bhayandar civic news (MBMC, roads, infrastructure, traffic, events) from trusted domains like Times of India, Mid-Day, Indian Express
3. **Article pipeline**: Split results → Filter junk URLs → Scrape full HTML → Extract `<p>` paragraphs → Clean into `clean_article` text
4. **De-duplication**: Load StoryLog from Google Sheets → Code node excludes previously sent URLs
5. **Impact ranking**: Custom Code node scores stories by local relevance (Mira Bhayandar/MBMC keywords, ₹ amounts, infrastructure/urgency signals) → Sort → Limit to top 3
6. **LLM summarization**: Aggregate stories into `story_bundle` → Strict anti-hallucination prompt → Groq Llama 3.3 generates unified "Headline + Briefing" → Fallback template for no-stories days
7. **Email delivery**: Parse LLM output → Render personalized responsive HTML → Load subscriber list (Email/Name/Locale) → Send via Gmail
8. **Full logging**: DigestLogs (run metadata) + append sent URLs to StoryLog

---

## Architecture diagram

Schedule (8AM)
↓
Tavily Search → Split → Filter URLs → Scrape HTML → Extract/Clean
↓
Google Sheets StoryLog → Dedupe Code Node
↓
Impact Score → Sort → Limit Top 3 → Aggregate Bundle
↓
Anti-Hallucination Prompt → Groq Llama 3.3 → LLM Gate + Fallback
↓
HTML Email Template → Load Subscribers → Gmail Send → Log Everything

---

## Technical highlights

### 🛡️ **Production reliability**
- **De-duplication**: Never sends the same URL twice (StoryLog sheet tracking)
- **Fallbacks**: "No major updates today" template when no stories pass filters or LLM fails
- **Error gates**: Multiple If nodes check article count, LLM output, email send status
- **Observability**: DigestLogs captures execution_id, date, locale, story_count, fallback_used, headline

### 🎯 **Smart ranking**
Custom `impact_score` algorithm in Code node:
Mira Bhayandar (+5) | MBMC (+4) | Mumbai (+1) | ₹/crore/lakh (+2) |
road/bridge/metro/water (+2) | fire/accident/protest (+3) | short text (-2)

### 🤖 **Anti-hallucination LLM**
Strict prompt engineering:
"You MUST ONLY use information explicitly provided... do NOT invent, assume, speculate...
FORMAT EXACTLY AS: Headline: <summary> | Briefing: 4–6 paragraphs"


---

## Tech stack

| Category | Tools |
|----------|-------|
| **Orchestration** | n8n (Schedule Trigger, Code nodes, If gates, Merge) |
| **Search** | Tavily n8n node (news-focused, domain whitelist) |
| **Scraping** | HTTP Request (browser headers) + HTML Extract |
| **LLM** | Groq `llama-3.3-70b-versatile` via n8n LangChain |
| **Data** | Google Sheets (Subscribers A:C, StoryLog, DigestLogs) |
| **Email** | Gmail OAuth (personalized HTML newsletters) |

---

## Files

- `TrendScoop-AI-v2.5.json` – Complete n8n workflow export (import-ready)

## Setup (for licensees)

1. **n8n**: Import JSON → configure credentials:
   - Tavily API key
   - Groq API key  
   - Google Sheets OAuth
   - Gmail OAuth
2. **Google Sheets**: Create doc with tabs:
   - `Subscribers` (A:C = Email, Name, Locale)
   - `StoryLog` (A:B = URL, Date)
   - `DigestLogs` (A:F = execution_id, run_date, locale, story_count, used_fallback, headline)
3. **Update Sheet IDs** in 4 Google Sheets nodes
4. **Activate workflow** → first run tomorrow 8AM

---

## Commercial roadmap

- ✅ **v2.5**: Mira Bhayandar production (de-dupe, ranking, logging, multi-sub)
- 🔄 **v3.0**: Multi-city support (parameterized locale/search)
- 🔄 **v3.5**: Telegram/WhatsApp + public webhook
- 🔄 **v4.0**: Subscription landing page + Stripe

**Contact**: Luv Saxena (LinkedIn/GitHub) for custom deployments, white-labeling, or city-specific adaptations.

---

**👨‍💻 Built by Luv Saxena**  
**📍 Mumbai, Maharashtra, India**  
**📅 February 2026**  
**💼 AI/ML Engineer | No-code Automation Specialist**  
**🔗 [LinkedIn](https://linkedin.com/in/luvsaxena279) | [GitHub](https://github.com/luvsaxena279)**  
**✉️ luvs99@gmail.com**  
**💰 Commercial licensing available – DM for pricing**
