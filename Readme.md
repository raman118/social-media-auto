# Social Media Factory v2.0 - Complete Documentation

## Table of Contents
1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Workflow Diagram](#workflow-diagram)
4. [Setup Guide](#setup-guide)
5. [Node Configuration](#node-configuration)
6. [Troubleshooting](#troubleshooting)

---

## Overview

### Purpose
Automates end-to-end social media content creation, compliance checking, approval, and multi-platform publishing for Advino Healthcare.

### Platforms Supported
- Instagram (Feed Posts, Carousels, Stories)
- Facebook (Page Posts)
- LinkedIn (Organization Posts)
- X/Twitter (Basic support)

### Content Types
- **Static**: Single image posts
- **Carousel**: Multi-slide presentations (3 slides)
- **Reel**: Short-form video content

### Key Features
- AI-powered content generation with GPT-4o-mini
- Automated hashtag research via SerpAPI
- Compliance validation (MCI healthcare guidelines)
- Multi-source asset generation (Pexels, Templated.io, OpenAI)
- Email-based approval workflow
- Automated scheduling and publishing
- Performance tracking and reporting

---

## System Architecture

### High-Level Flow
```
START (Daily 4 AM or New Row Added)
   │
   ▼
[Read Google Sheets Calendar]
   │
   ▼
[Filter Today's Posts] (scheduled_date == today AND status == "Scheduled")
   │
   ▼
[Normalize Data] (Handle column name variations)
   │
   ▼
[AI Content Generator]
   ├─ SerpAPI (Hashtag Research)
   ├─ GPT-4o-mini (Content Generation)
   └─ JSON Parser (Structure Validation)
   │
   ▼
[Compliance Validator]
   ├─ Check prohibited claims
   ├─ Verify disclaimers
   └─ Validate completeness
   │
   ├─ PASS ──→ [Route by Content Type]
   │              │
   │              ├─ Static ──→ [Pexels Search] ──→ [Download] ──→ [Upload ImgBB]
   │              ├─ Carousel → [Templated.io API]
   │              └─ Reel ────→ [OpenAI DALL-E]
   │              │
   │              ▼
   │           [Merge Assets]
   │              │
   │              ▼
   │           [Send Approval Email] (Wait 2 hours)
   │              │
   │              ├─ APPROVED ──→ [Publish to Platforms]
   │              │                  ├─ Instagram (2-step: Create + Publish)
   │              │                  ├─ Facebook (Single POST)
   │              │                  └─ LinkedIn (ugcPosts API)
   │              │                  │
   │              │                  ▼
   │              │               [Update Sheet Status]
   │              │                  │
   │              │                  ▼
   │              │               [Aggregate Results]
   │              │                  │
   │              │                  ▼
   │              │               [Send Daily Report]
   │              │
   │              └─ REJECTED ──→ [Mark as Rejected] ──→ END
   │
   └─ FAIL ──→ [Send Compliance Alert] ──→ END
```

### Detailed Architecture Diagram
```
┌───────────────────────────────────────────────────────────┐
│                    TRIGGER LAYER                          │
│  ┌──────────────┐           ┌──────────────┐             │
│  │  Schedule    │           │  Sheet Row   │             │
│  │  (4 AM)      │           │  Trigger     │             │
│  └──────┬───────┘           └──────┬───────┘             │
└─────────┼──────────────────────────┼───────────────────────┘
          └──────────┬───────────────┘
                     │
┌────────────────────▼──────────────────────────────────────┐
│                  DATA LAYER                                │
│  Read Sheets → Filter Today → Normalize → Pass to AI      │
└────────────────────┬──────────────────────────────────────┘
                     │
┌────────────────────▼──────────────────────────────────────┐
│                CONTENT GENERATION LAYER                    │
│  ┌─────────────────────────────────────────────────┐      │
│  │  AI Agent (GPT-4o-mini + SerpAPI + Parser)     │      │
│  │  Generates: Captions, Hashtags, Image Prompts  │      │
│  └─────────────────────┬───────────────────────────┘      │
└────────────────────────┼──────────────────────────────────┘
                         │
┌────────────────────────▼──────────────────────────────────┐
│              COMPLIANCE & VALIDATION LAYER                 │
│  Validate → Pass/Fail → Alert if Failed                   │
└────────────────────────┬──────────────────────────────────┘
                         │
┌────────────────────────▼──────────────────────────────────┐
│                 ASSET GENERATION LAYER                     │
│  Switch by Type:                                           │
│  ├─ Static: Pexels → Download → ImgBB                    │
│  ├─ Carousel: Templated.io API                           │
│  └─ Reel: OpenAI DALL-E → ImgBB                          │
└────────────────────────┬──────────────────────────────────┘
                         │
┌────────────────────────▼──────────────────────────────────┐
│                  APPROVAL LAYER                            │
│  Send Email → Wait for Response → Route                   │
└────────────────────────┬──────────────────────────────────┘
                         │
┌────────────────────────▼──────────────────────────────────┐
│               PUBLISHING LAYER (Parallel)                  │
│  ├─ Instagram: Create Media → Publish                     │
│  ├─ Facebook: POST /photos                                │
│  └─ LinkedIn: POST /ugcPosts                              │
└────────────────────────┬──────────────────────────────────┘
                         │
┌────────────────────────▼──────────────────────────────────┐
│                REPORTING LAYER                             │
│  Update Sheet → Aggregate → Generate Report → Send Email  │
└────────────────────────────────────────────────────────────┘
```

---

## Workflow Diagram

### Node Sequence
```
Node  | Name                          | Type          | Key Function
------|-------------------------------|---------------|---------------------------
1     | Daily Scheduler               | Trigger       | Runs at 4 AM daily
2     | New Row Trigger               | Trigger       | On sheet row add
3     | Read Calendar Sheet           | Google Sheets | Fetch all rows
4     | Filter Today's Posts          | IF            | Date + status filter
5     | Normalize Sheet Data          | Code          | Standardize columns
6     | AI Content Generator          | LangChain     | Generate content
7     | SerpAPI Tool                  | Tool          | Hashtag research
8     | GPT-4o-mini Model             | LLM           | Language model
9     | Content Structure Parser      | Parser        | JSON validation
10    | Compliance Validator          | Code          | Check guidelines
11    | Compliance Passed?            | IF            | Route decision
12    | Handle Compliance Failure     | Code          | Error handling
13    | Send Compliance Alert         | Gmail         | Alert email
14    | Route by Content Type         | Switch        | Static/Carousel/Reel
15    | Search Pexels Images          | HTTP          | Image search
16    | Extract Image Data            | Set           | Parse response
17    | Download Image                | HTTP          | Get binary
18    | Upload to ImgBB               | HTTP          | Host image
19    | Generate Carousel             | HTTP          | Templated.io
20    | Generate AI Image             | OpenAI        | DALL-E fallback
21    | Merge All Assets              | Merge         | Combine paths
22    | Send for Approval             | Gmail         | Approval email
23    | Is Approved?                  | IF            | Check response
24    | Handle Rejection              | Code          | Rejection logic
25    | Mark as Rejected              | Google Sheets | Update status
26    | Instagram - Create Media      | HTTP          | Graph API
27    | Instagram - Publish           | Facebook API  | Publish post
28    | Facebook - Post               | Facebook API  | Direct post
29    | LinkedIn - Post               | LinkedIn      | Organization post
30    | Merge Platform Results        | Merge         | Combine responses
31    | Update Sheet Status           | Google Sheets | Write results
32    | Aggregate Daily Results       | Aggregate     | Collect posts
33    | Prepare Results Email         | LangChain     | HTML report
34    | GPT Model for Reports         | LLM           | Email formatting
35    | Send Results Email            | Gmail         | Final report
```

---

## Setup Guide

### Prerequisites
1. n8n instance (self-hosted or cloud)
2. Google Account with Sheets API access
3. API Keys needed:
   - OpenAI API key
   - SerpAPI key 
   - Pexels API key
   - ImgBB API key
   - Templated.io API key (optional)
4. Social Media Access:
   - Facebook Page ID + Access Token
   - Instagram Business Account ID
   - LinkedIn Organization ID + OAuth
5. Gmail for approvals/reports

### Step 1: Create Google Sheet
Create sheet with these columns:
```
Post ID | Scheduled Date | Platform | Content Type | Topic | Key Message | 
CTA | Tone | Target Audience | Graphic Style | Asset Link | Asset Source | 
Video Description | Status | Note Metric | Priority
```

Example row:
```
POST_001 | 2025-10-01 | Instagram | Carousel | Navratri Elderly Care | 
Stay safe during festivities | Call: +91 96 2445 2445 | Empathetic | 
Seniors in Gujarat | Canva Blue | https://pexels.com/... | Pexels | 
N/A | Scheduled | 8% engagement | High
```

### Step 2: Import Workflow
1. Copy JSON from artifact
2. n8n: Create Workflow → Import from JSON
3. Replace placeholders:
   - `YOUR_SHEET_ID` → Your Google Sheet ID
   - `YOUR_OPENAI_CREDS` → OpenAI credential name
   - `YOUR_SERP_CREDS` → SerpAPI credential
   - `YOUR_PEXELS_API_KEY` → From pexels.com/api
   - `YOUR_IMGBB_API_KEY` → From imgbb.com
   - `YOUR_FB_GRAPH_CREDS` → Facebook credential
   - `YOUR_LINKEDIN_CREDS` → LinkedIn credential
   - `YOUR_APPROVAL_EMAIL` → Your email
   - `YOUR_REPORT_EMAIL` → Team email

### Step 3: Configure Credentials

#### Google Sheets OAuth2
```
1. Google Cloud Console → Create OAuth 2.0 Client
2. Redirect URI: https://your-n8n.com/rest/oauth2-credential/callback
3. Copy Client ID + Secret to n8n
4. Authorize and test
```

#### Facebook Graph API
```
1. developers.facebook.com → Create App
2. Add Instagram Graph API
3. Generate token with permissions:
   - pages_read_engagement
   - pages_manage_posts
   - instagram_basic
   - instagram_content_publish
4. Get Page ID from Page Settings
5. Get Instagram Business ID:
   GET /{page-id}?fields=instagram_business_account
```

#### LinkedIn OAuth2
```
1. linkedin.com/developers → Create App
2. Add redirect URI
3. Request Community Management API access
4. Get Organization ID from:
   GET /v2/organizationalEntityAcls?q=roleAssignee
```

### Step 4: Test Nodes
Test in order:
1. Read Calendar Sheet → Should return rows
2. Filter Today's Posts → Set row date to today
3. AI Content Generator → Verify JSON output
4. Compliance Validator → Check error detection
5. Asset Generation → Test each branch
6. Approval Email → Send to yourself
7. Publishing → Start with Instagram

### Step 5: Activate
```
Daily Scheduler settings:
- Trigger at: 4:00 AM
- Timezone: Asia/Kolkata
- Active: YES
```

---

## Node Configuration

### AI Content Generator Prompt
```
Generate comprehensive social media content for ALL platforms.

# Post Details
- Post ID: {{ $json.post_id }}
- Content Type: {{ $json.content_type }}
- Theme: {{ $json.theme_topic }}
- Key Message: {{ $json.key_message }}

# Tasks
1. Use SerpAPI to find 15-20 trending hashtags
2. Generate platform-specific content:
   - Instagram: Caption (150-200 words), 20-30 hashtags
   - Facebook: Post (200-300 words), poll, 10-15 hashtags
   - LinkedIn: Professional (250-350 words), 10-15 hashtags
   - X: Tweet (280 chars), thread, 3-5 hashtags
3. Create image prompts (Static: 1, Carousel: 3, Reel: video script)
4. Include compliance disclaimer
5. Format with Social Media Content Tool
```

### Compliance Validator Logic
```javascript
// Prohibited claims
const prohibited = ['cure', 'guaranteed', '100% effective', 'miracle'];
prohibited.forEach(claim => {
  if (content.includes(claim)) {
    validation.errors.push(`Prohibited: "${claim}"`);
  }
});

// Disclaimer check
let hasDisclaimer = false;
for (const platform in content.platform_posts) {
  if (content.platform_posts[platform].caption?.includes('consult')) {
    hasDisclaimer = true;
  }
}
if (!hasDisclaimer) {
  validation.warnings.push('Add healthcare disclaimer');
}

// Platform completeness
['Instagram', 'Facebook', 'LinkedIn', 'X'].forEach(platform => {
  if (!content.platform_posts[platform]) {
    validation.errors.push(`Missing ${platform} content`);
  }
});

// Result
validation.passed = validation.errors.length === 0;
```

### Instagram Publishing (2-Step)
```javascript
// Step 1: Create Media Container
POST /v20.0/{instagram-id}/media
Params: { image_url, caption }
Response: { id: "creation_id" }

// Step 2: Publish
POST /v20.0/{instagram-id}/media_publish
Params: { creation_id }
Response: { id: "post_id" }

// For Carousels: Create 3 containers, then carousel container, then publish
```

### Facebook Publishing (1-Step)
```javascript
POST /v20.0/{page-id}/photos
Params: {
  message: "Caption with hashtags",
  url: "https://i.ibb.co/image.jpg",
  published: true
}
Response: { id: "post_id" }
```

### LinkedIn Publishing
```javascript
POST /v2/ugcPosts
Body: {
  author: "urn:li:organization:123456",
  lifecycleState: "PUBLISHED",
  specificContent: {
    "com.linkedin.ugc.ShareContent": {
      shareCommentary: { text: "Post text" },
      shareMediaCategory: "IMAGE",
      media: [{ media: "urn:li:digitalmediaAsset:..." }]
    }
  }
}
```

---

## Troubleshooting

### Common Issues

#### Issue: No items processed
```
Cause: Date format mismatch
Fix: Ensure sheet uses YYYY-MM-DD format
Test: Set Scheduled Date to today manually
```

#### Issue: Compliance failures
```
Cause: AI generated prohibited terms
Fix: Strengthen AI prompt:
"CRITICAL: Never use: cure, guaranteed, miracle.
ALWAYS include: Consult healthcare professional."
```

#### Issue: Asset generation fails
```
Cause: Pexels returns no results
Fix: Add fallback in Merge Assets:
if (!assets.image_url) {
  assets.image_url = "https://i.ibb.co/default-advino.jpg";
}
```

#### Issue: Instagram "Invalid image URL"
```
Cause: ImgBB URL not direct link
Fix: Ensure URL ends in .jpg or .png
Test: Open URL in browser - should show image directly
```

#### Issue: "Expression error: Cannot read property"
```
Cause: Missing JSON field
Fix: Use safe navigation:
{{ $json.field?.subfield || 'default' }}
Example: {{ $json.platform_posts?.Instagram?.caption || '' }}
```

#### Issue: API timeout
```
Cause: Slow API response
Fix: Increase timeout in node settings:
{
  "options": {
    "timeout": 60000
  }
}
```

#### Issue: Sheet update fails
```
Cause: Invalid row_number
Fix: Add error handling:
{
  "retryOnFail": true,
  "maxTries": 3,
  "continueOnFail": true
}
```

### Performance Metrics
```
Single Post Execution:
- Content Generation: 30-60s
- Asset Generation: 15-30s
- Publishing: 10-20s
Total: ~1-2 minutes (excluding approval)

Daily Batch (5 posts):
- Sequential: ~10 minutes
- With approval: Variable (user-dependent)
```

### API Rate Limits
```
Service          | Free Tier        | Recommendation
-----------------|------------------|------------------
OpenAI           | N/A              | Paid ($10/mo min)
SerpAPI          | 100/month        | Start free
Pexels           | 200/hour         | Free sufficient
ImgBB            | Unlimited        | Free forever
Templated.io     | 100/month        | Start free
Instagram API    | 200 calls/hour   | Sufficient
Facebook API     | Rate limited     | Use page token
LinkedIn API     | Throttled        | Request quota
```

### Error Handling Strategy
```
Critical Nodes (Publishing):
- retryOnFail: true
- maxTries: 2
- waitBetweenTries: 5000ms

Non-Critical (Image search):
- continueOnFail: true
- Log error, continue workflow

Compliance Failures:
- Stop workflow
- Send alert email
- Require manual review
```

---

## Best Practices

1. **Test incrementally**: Test each node before running full workflow
2. **Monitor daily**: Check execution history for failures
3. **Update prompts**: Refine AI prompts based on output quality
4. **Cache assets**: Reuse Pexels images for similar themes
5. **Set alerts**: Email notifications for critical failures
6. **Review compliance**: Weekly audit of compliance failures
7. **Track performance**: Analyze which content types perform best
8. **Backup credentials**: Store API keys in secure password manager
9. **Document changes**: Keep log of workflow modifications
10. **Regular maintenance**: Update API credentials before expiry

---

## Support & Resources

- **n8n Documentation**: https://docs.n8n.io
- **OpenAI API Docs**: https://platform.openai.com/docs
- **Facebook Graph API**: https://developers.facebook.com/docs/graph-api
- **Instagram API**: https://developers.facebook.com/docs/instagram-api
- **LinkedIn API**: https://docs.microsoft.com/en-us/linkedin
- **Pexels API**: https://www.pexels.com/api/documentation
- **SerpAPI Docs**: https://serpapi.com/search-api

For workflow-specific issues, review execution logs in n8n and check node error messages.
