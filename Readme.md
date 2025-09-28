# Social Media Content Factory

An automated n8n workflow for generating, approving, and publishing AI-powered social media content across multiple platforms.

## Overview

The Social Media Content Factory is a comprehensive automation workflow that streamlines the entire social media content creation process. It generates platform-specific content using AI, fetches relevant images, requires human approval, and publishes across LinkedIn, Instagram, Facebook, X (Twitter), TikTok, Threads, and YouTube Shorts.

## Workflow Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         WORKFLOW DIAGRAM                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐                                               │
│  │Google Sheets │                                               │
│  │   Trigger    │                                               │
│  └──────┬───────┘                                               │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────────────────────────┐                          │
│  │   AI Content Generation (GPT-4)   │◄────┐                    │
│  │  - Platform-specific posts        │     │                    │
│  │  - Hashtags & CTAs               │     │                    │
│  │  - Image suggestions             │  ┌──┴─────────┐          │
│  └──────┬───────────────────────────┘  │  SerpAPI   │          │
│         │                               │ Web Search │          │
│         ▼                               └────────────┘          │
│  ┌──────────────────────────────────┐                          │
│  │    Email for Human Approval      │                          │
│  │  - HTML formatted preview        │                          │
│  │  - Approve/Reject buttons        │                          │
│  └──────┬───────────────────────────┘                          │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────┐     ┌─────────────────────────┐              │
│  │  Approved?   │────►│    Pexels API Search    │              │
│  └──────┬───────┘ NO  │  - Fetch relevant images│              │
│         │YES          └───────┬─────────────────┘              │
│         ▼                     ▼                                 │
│  ┌──────────────────────────────────┐                          │
│  │     Image Processing Pipeline     │                          │
│  │  1. Download selected image       │                          │
│  │  2. Upload to imgbb hosting       │                          │
│  │  3. Generate public URL           │                          │
│  └──────┬───────────────────────────┘                          │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────────────────────────────────────┐              │
│  │         Platform Publishing Matrix           │              │
│  │                                               │              │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │              │
│  │  │Instagram │  │ Facebook │  │ LinkedIn │  │              │
│  │  └──────────┘  └──────────┘  └──────────┘  │              │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │              │
│  │  │    X     │  │  TikTok  │  │ Threads  │  │              │
│  │  └──────────┘  └──────────┘  └──────────┘  │              │
│  │         ┌──────────────────┐                │              │
│  │         │  YouTube Shorts  │                │              │
│  │         └──────────────────┘                │              │
│  └──────┬───────────────────────────────────────┘              │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────────────────────────┐                          │
│  │    Results Email Notification    │                          │
│  │  - Publishing status per platform│                          │
│  │  - Links to published content    │                          │
│  └──────────────────────────────────┘                          │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

## Features

### Core Capabilities
- **Multi-Platform Support**: Generates tailored content for 7 major social media platforms
- **AI-Powered Content Creation**: Uses GPT-4 for intelligent content generation
- **Web Research Integration**: Leverages SerpAPI for real-time information gathering
- **Automated Image Sourcing**: Integrates with Pexels API for relevant visual content
- **Human-in-the-Loop Approval**: Email-based approval system before publishing
- **Batch Processing**: Processes multiple content requests from Google Sheets
- **Status Tracking**: Complete audit trail of content creation and publishing

### Platform-Specific Features
Each platform receives customized content including:
- Optimized post text/captions
- Platform-appropriate hashtags
- Relevant emojis (where applicable)
- Call-to-action statements
- Image/video suggestions
- Character limit compliance (X/Twitter)

## Prerequisites

### Required Services and API Keys

1. **OpenAI API Key**
   - Required for GPT-4 content generation
   - Sign up at: https://platform.openai.com/

2. **Google Sheets API**
   - For trigger and data input
   - Setup OAuth2 credentials

3. **Pexels API Key**
   - For image sourcing
   - Get your API key at: https://www.pexels.com/api/

4. **SerpAPI Key**
   - For web search capabilities
   - Register at: https://serpapi.com/

5. **imgbb API Key**
   - For image hosting
   - Sign up at: https://imgbb.com/

6. **Gmail OAuth2**
   - For sending approval and result emails
   - Configure OAuth2 credentials: https://docs.n8n.io/integrations/builtin/credentials/google/

7. **Social Media Platform Credentials**
   - Facebook Graph API
   - LinkedIn Community Management OAuth2
   - Instagram Business Account
   - Additional platform APIs as needed

## Installation

### Step 1: Import Workflow
1. Open your n8n instance
2. Navigate to Workflows
3. Click "Import"
4. Upload the provided JSON workflow file

### Step 2: Configure Credentials
Update all credential nodes with your API keys:

1. **OpenAI Nodes**
   - Navigate to each OpenAI node
   - Add your API key to credentials

2. **Google Sheets Trigger**
   - Configure OAuth2 authentication
   - Link to your input spreadsheet

3. **Pexels Search Images**
   - Add your Pexels API key in the Authorization header

4. **SerpAPI Tool**
   - Configure with your SerpAPI key

5. **Image Hosting (imgbb)**
   - Add your imgbb API key

6. **Gmail Nodes**
   - Configure OAuth2 for both approval and results emails
   - Update recipient email addresses

7. **Social Media Platforms**
   - Configure each platform's API credentials
   - Update page/account IDs as needed

### Step 3: Google Sheets Setup
Create an input spreadsheet with the following columns:
- Post ID
- Platform
- Scheduled Date
- Post Type
- Topic
- Key Message
- Target Audience
- Value Proposition
- CTA (Call to Action)
- Tone
- Graphic Style
- Asset Link
- Draft Content
- Status
- Note metric

### Step 4: Customize Prompts
Modify the AI prompts in the "Social Media Content Factory" node to match your brand voice and requirements.

## Usage

### Basic Workflow

1. **Add Content Request**
   - Enter new row in Google Sheets with content requirements
   - Workflow triggers automatically on new row addition

2. **Content Generation**
   - AI generates platform-specific content
   - Web search enriches content with current information
   - System suggests relevant images

3. **Approval Process**
   - HTML-formatted email sent for review
   - Click approve/reject in email
   - 45-minute timeout for approval response

4. **Image Processing**
   - Upon approval, workflow searches Pexels
   - Downloads and hosts selected image
   - Generates public URL for posting

5. **Publishing**
   - Content published to selected platforms
   - Each platform receives tailored version
   - Error handling for failed posts

6. **Results Notification**
   - Summary email with publishing status
   - Links to published content
   - Error reports if any posts failed

## Configuration Options

### Timeout Settings
- Default approval timeout: 45 minutes
- Configurable in "Gmail User for Approval" node

### Image Settings
- Default image count: 10 results from Pexels
- Orientation: Landscape (configurable)
- Hosting expiration: Permanent (0)

### Content Parameters
- Modify brand colors, fonts, and tone in prompts
- Adjust hashtag counts per platform
- Configure character limits

## Troubleshooting

### Common Issues

1. **Workflow Not Triggering**
   - Verify Google Sheets permissions
   - Check trigger configuration
   - Ensure proper OAuth2 setup

2. **Image Upload Failures**
   - Verify imgbb API key validity
   - Check file size limits
   - Ensure proper image format

3. **Platform Publishing Errors**
   - Verify API credentials are current
   - Check platform-specific requirements
   - Review rate limiting policies

4. **Approval Email Not Received**
   - Check spam/junk folders
   - Verify Gmail OAuth2 configuration
   - Confirm recipient email address

### Error Handling
The workflow includes built-in error handling:
- Continues execution on platform publishing failures
- Logs errors in results email
- Provides fallback for missing images

## Best Practices

1. **Content Planning**
   - Batch content requests for efficiency
   - Include detailed briefs in spreadsheet
   - Plan content calendar in advance

2. **Quality Control**
   - Review AI-generated content before approval
   - Verify image relevance
   - Check platform-specific formatting

3. **Performance Optimization**
   - Monitor API usage and costs
   - Implement rate limiting if needed
   - Archive completed content regularly

4. **Security**
   - Rotate API keys periodically
   - Use environment variables for sensitive data
   - Implement access controls on Google Sheets

## Limitations

- Platform API rate limits apply
- Image hosting dependent on imgbb service
- Approval timeout cannot exceed email service limits
- Some platforms may require additional setup for full automation

## Support

For issues or questions:
1. Check the troubleshooting section
2. Review n8n documentation: https://docs.n8n.io/
3. Consult platform-specific API documentation
4. Contact your system administrator

## License

This workflow is provided as-is for use within your organization. Modify and distribute according to your internal policies.

## Version History

- v1.0: Initial release with 7 platform support
- Features AI content generation, image sourcing, and approval workflow

---

Built with n8n - The workflow automation platform
