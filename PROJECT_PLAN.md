# AI News Aggregator Project Plan

## Project Overview

An intelligent AI news aggregation system that collects, analyzes, and delivers personalized AI industry news through email digests and an interactive Discord bot.

## Project Goals

1. **Automate AI news collection** from multiple reliable sources
2. **Provide intelligent filtering** to surface important developments
3. **Deliver customizable notifications** based on user preferences
4. **Enable interactive querying** of news through a RAG-powered Discord bot

## Phase 0: Newsletter Synthesizer MVP

### Timeline: 1 weekend

### Core Concept
Subscribe to multiple AI newsletters using a dedicated email address, then use an LLM to synthesize them into a single weekly digest sent to your main email.

### Implementation Steps

#### 1. Email Setup
- **Create dedicated email**: `ainews@yourdomain.com`
- **Subscribe to key newsletters**:
  - The Batch by DeepLearning.AI (Weekly, Saturdays)
  - Import AI by Jack Clark (Weekly, Mondays)
  - The Neuron (Daily)
  - TLDR AI (Daily)
  - Anthropic Blog
  - OpenAI Blog
  - Google AI Blog
  - Hugging Face Newsletter

#### 2. Simple Python Script
```python
# newsletter_synthesizer.py
import os
from datetime import datetime, timedelta
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build
import anthropic
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from bs4 import BeautifulSoup

def fetch_newsletters(gmail_service, days_back=7):
    """Fetch newsletters from the past week"""
    query = f'after:{(datetime.now() - timedelta(days=days_back)).strftime("%Y/%m/%d")}'
    results = gmail_service.users().messages().list(userId='me', q=query).execute()
    
    newsletters = []
    for msg in results.get('messages', []):
        message = gmail_service.users().messages().get(userId='me', id=msg['id']).execute()
        newsletters.append(extract_content(message))
    
    return newsletters

def synthesize_content(newsletters, claude_client):
    """Use Claude to synthesize newsletters into key points"""
    prompt = """
    Synthesize the following AI newsletters into a concise weekly digest.
    
    Format:
    1. Executive Summary (3-5 most important developments)
    2. Research Breakthroughs
    3. Product Launches & Updates
    4. Industry News
    5. Interesting Projects/Tools
    
    Keep each point brief but informative. Include source attribution.
    
    Newsletters:
    {content}
    """
    
    response = claude_client.messages.create(
        model="claude-3-opus-20240229",
        messages=[{"role": "user", "content": prompt.format(content=newsletters)}]
    )
    
    return response.content

def send_digest(content, recipient_email):
    """Send the synthesized digest"""
    msg = MIMEMultipart()
    msg['Subject'] = f'AI News Digest - Week of {datetime.now().strftime("%B %d, %Y")}'
    msg['From'] = os.environ['SENDER_EMAIL']
    msg['To'] = recipient_email
    
    msg.attach(MIMEText(content, 'html'))
    
    # Send email via SMTP
    with smtplib.SMTP_SSL('smtp.gmail.com', 465) as server:
        server.login(os.environ['SENDER_EMAIL'], os.environ['SENDER_PASSWORD'])
        server.send_message(msg)

# Weekly cron job
if __name__ == "__main__":
    gmail = build_gmail_service()  # Implementation depends on auth method
    claude = anthropic.Anthropic(api_key=os.environ['ANTHROPIC_API_KEY'])
    
    newsletters = fetch_newsletters(gmail)
    digest = synthesize_content(newsletters, claude)
    send_digest(digest, os.environ['RECIPIENT_EMAIL'])
```

#### 3. Deployment Options
- **Option A**: GitHub Actions (free, runs weekly)
- **Option B**: AWS Lambda + EventBridge (minimal cost)
- **Option C**: Local cron job
- **Option D**: Render.com cron job (free tier)

### Success Criteria
- Successfully consolidates 5+ newsletters into 1 digest
- Reduces reading time from 30+ minutes to 5-10 minutes
- Maintains important information without loss
- Costs < $5/month in API calls

### Decision Gate
After running for 4 weeks, evaluate:
- Is the synthesis quality good enough?
- Am I missing important news?
- Is the time savings worth the setup?
- Should I proceed to Phase 1 (full aggregator)?

## Phase 1: Email Notification System

### Timeline: 2 weeks

### Core Features

#### 1. Data Collection Pipeline
- **RSS Feed Integration**
  - OpenAI Blog
  - Anthropic Blog
  - Google AI Blog
  - Meta AI Blog
  - Hugging Face Blog
  - AI research labs (DeepMind, FAIR, etc.)
  
- **API Integrations**
  - Hacker News API (filter for AI keywords)
  - Reddit API (r/MachineLearning, r/artificial, r/singularity)
  - GitHub Trending API (AI/ML repositories)
  - NewsAPI.org (technology category with AI filters)
  - Product Hunt API (AI tools category)

- **Web Scraping Fallbacks**
  - BeautifulSoup for sites without APIs
  - Respect robots.txt and rate limits
  - Cache results to minimize requests

#### 2. Content Processing
- **Deduplication System**
  - Similarity matching using TF-IDF or sentence embeddings
  - URL normalization
  - Title/content fuzzy matching
  
- **Importance Scoring Algorithm**
  ```
  Score = (source_weight × 0.3) + 
          (engagement_score × 0.2) + 
          (keyword_relevance × 0.2) + 
          (recency × 0.2) + 
          (category_importance × 0.1)
  ```
  
- **Categorization**
  - Research Papers
  - Product Launches
  - Industry News
  - Open Source Projects
  - Tutorials/Guides
  - Company Updates

#### 3. Email Generation
- **LLM Integration**
  - Claude API for content synthesis
  - Structured prompts for consistent formatting
  - Fact preservation from original sources
  
- **Email Templates**
  - Weekly Digest (default: Saturday 9 AM, configurable)
  - Daily Brief (optional, configurable time)
  - Breaking News Alert (importance score > 8.5)
  
- **Content Structure**
  ```
  1. Executive Summary (3-5 key points)
  2. Top Stories (with "why it matters" context)
  3. Product Launches
  4. Research Highlights
  5. Community Discussions
  6. Upcoming Events
  7. Quick Links (all other relevant items)
  ```

#### 4. Configuration System
- **User Preferences**
  - Delivery schedule (day/time)
  - Email frequency (weekly/daily/breaking)
  - Category preferences
  - Importance threshold
  - Content length preference
  
- **Technical Configuration**
  - API keys management
  - Source feed URLs
  - Email service settings
  - Database connection

### Technical Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Data Sources                         │
├─────────────┬────────────┬────────────┬───────────────┤
│   RSS Feeds │    APIs    │ Web Scrape │  User Sources │
└──────┬──────┴─────┬──────┴─────┬──────┴───────┬───────┘
       │            │            │              │
       └────────────┴────────────┴──────────────┘
                           │
                    ┌──────▼──────┐
                    │  Collector   │
                    │   Service    │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Processing  │
                    │   Pipeline   │
                    └──────┬──────┘
                           │
                ┌──────────┴──────────┐
                │                     │
         ┌──────▼──────┐      ┌──────▼──────┐
         │  Database   │      │    Email    │
         │ PostgreSQL  │      │   Service   │
         └─────────────┘      └─────────────┘
```

### Database Schema

```sql
-- News items table
CREATE TABLE news_items (
    id UUID PRIMARY KEY,
    title TEXT NOT NULL,
    url TEXT UNIQUE NOT NULL,
    source TEXT NOT NULL,
    category TEXT,
    content TEXT,
    summary TEXT,
    published_at TIMESTAMP,
    collected_at TIMESTAMP DEFAULT NOW(),
    importance_score FLOAT,
    engagement_metrics JSONB,
    tags TEXT[]
);

-- User preferences
CREATE TABLE user_preferences (
    user_id UUID PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    delivery_schedule JSONB,
    category_preferences TEXT[],
    importance_threshold FLOAT DEFAULT 7.0,
    timezone TEXT DEFAULT 'UTC'
);

-- Delivery logs
CREATE TABLE delivery_logs (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES user_preferences(user_id),
    delivered_at TIMESTAMP DEFAULT NOW(),
    delivery_type TEXT,
    items_count INT,
    status TEXT
);
```

## Phase 2: RAG-Enabled Discord Bot

### Timeline: 2 weeks

### Core Features

#### 1. Vector Database Setup
- **Technology**: ChromaDB or Qdrant
- **Embedding Model**: OpenAI text-embedding-ada-002 or BGE-large
- **Index Structure**:
  - News content embeddings
  - Metadata filtering (date, category, source)
  - Hybrid search (semantic + keyword)

#### 2. Discord Bot Implementation
- **Framework**: discord.py or interactions.py
- **Commands**:
  - `/news latest` - Get latest news (customizable count)
  - `/news search [query]` - Semantic search through news
  - `/news category [type]` - Filter by category
  - `/news explain [topic]` - Get detailed explanation
  - `/news subscribe [preferences]` - Set personal preferences
  - `/news summary [timeframe]` - Get time-based summary
  
- **Interactive Features**:
  - Conversation memory per user
  - Follow-up questions
  - Source citations
  - Reaction-based feedback

#### 3. RAG Pipeline
```
User Query → Embedding → Vector Search → Rerank → 
Context Building → LLM Generation → Response Formatting
```

- **Retrieval Strategy**:
  - Top-k semantic search (k=10)
  - MMR (Maximum Marginal Relevance) for diversity
  - Recency boost for recent news
  - Category filtering when specified

- **Context Management**:
  - 4K token context window
  - Smart truncation
  - Source attribution
  - Fact verification

### Deployment Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Discord Bot                        │
├─────────────────────┬───────────────────────────────┤
│   Event Handlers    │        Commands               │
└──────────┬──────────┴────────────┬─────────────────┘
           │                       │
    ┌──────▼──────┐         ┌──────▼──────┐
    │   Message   │         │     RAG     │
    │   Router    │         │   Pipeline  │
    └──────┬──────┘         └──────┬──────┘
           │                       │
           └───────────┬───────────┘
                       │
                ┌──────▼──────┐
                │   Services   │
                ├──────────────┤
                │ Vector Store │
                │   LLM API    │
                │  Database    │
                └──────────────┘
```

## Implementation Milestones

### Phase 0: Newsletter Synthesizer
- [ ] Create dedicated email account
- [ ] Subscribe to newsletters
- [ ] Write synthesis script
- [ ] Deploy weekly cron job
- [ ] Run for 4 weeks and evaluate

### Week 1: Foundation
- [ ] Set up project structure
- [ ] Configure development environment
- [ ] Implement RSS feed parser
- [ ] Create database schema
- [ ] Basic data collection

### Week 2: Processing & Delivery
- [ ] Implement deduplication
- [ ] Build importance scoring
- [ ] Create email templates
- [ ] Integrate Claude API
- [ ] Test email delivery

### Week 3: Discord Bot
- [ ] Set up Discord bot
- [ ] Implement basic commands
- [ ] Configure vector database
- [ ] Create embedding pipeline

### Week 4: RAG Integration
- [ ] Build RAG pipeline
- [ ] Add conversation memory
- [ ] Implement all commands
- [ ] Testing and refinement

## Configuration File Structure

```yaml
# config.yaml
app:
  name: "AI News Aggregator"
  version: "1.0.0"

# Phase 0 config
newsletter_synthesis:
  source_email: "ainews@yourdomain.com"
  recipient_email: "you@yourdomain.com"
  schedule: "0 9 * * SAT"  # Every Saturday at 9 AM
  
sources:
  rss_feeds:
    - url: "https://openai.com/blog/rss.xml"
      weight: 0.9
    - url: "https://www.anthropic.com/index/rss.xml"
      weight: 0.9
  
  apis:
    hackernews:
      enabled: true
      keywords: ["AI", "machine learning", "GPT", "LLM"]
    reddit:
      subreddits: ["MachineLearning", "artificial"]
      
email:
  provider: "sendgrid"
  from_address: "ainews@yourdomain.com"
  
  schedules:
    weekly:
      enabled: true
      day: "saturday"  # configurable
      time: "09:00"
      timezone: "America/New_York"
    
    daily:
      enabled: false
      time: "08:00"
    
    breaking:
      enabled: true
      importance_threshold: 8.5

ai:
  provider: "anthropic"
  model: "claude-3-opus-20240229"
  
database:
  type: "postgresql"
  connection_string: "${DATABASE_URL}"
  
discord:
  bot_token: "${DISCORD_BOT_TOKEN}"
  command_prefix: "/"
  
vector_store:
  provider: "chromadb"
  collection_name: "ai_news"
  embedding_model: "text-embedding-ada-002"
```

## Success Metrics

1. **Data Quality**
   - < 5% duplicate stories
   - > 90% relevant content
   - < 24 hour lag for major news

2. **User Engagement**
   - Email open rate > 40%
   - Bot query response time < 2s
   - User retention > 80% monthly

3. **System Reliability**
   - 99% uptime
   - Successful delivery rate > 95%
   - Error rate < 1%

## Future Enhancements

1. **Web Dashboard**
   - Visual analytics
   - Preference management
   - Historical browsing

2. **Advanced Features**
   - Sentiment analysis
   - Trend detection
   - Personalized recommendations
   - Multi-language support

3. **Additional Integrations**
   - Slack bot
   - Telegram bot
   - RSS feed generation
   - Podcast generation