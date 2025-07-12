# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AINewsAggregator is an intelligent news aggregation system that collects, analyzes, and delivers personalized AI industry news through email digests and a Discord bot with RAG capabilities.

## Project Structure

```
AINewsAggregator/
├── src/
│   ├── collectors/      # RSS feeds, API integrations
│   ├── processors/      # Deduplication, scoring, categorization
│   ├── services/        # Email, database, AI integrations
│   ├── discord_bot/     # Discord bot implementation (Phase 2)
│   └── utils/           # Common utilities
├── tests/               # Test files
├── config/              # Configuration files
├── docs/                # Documentation
└── data/                # Local data storage (created at runtime)
```

## Development Commands

### Setup
```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Copy and configure settings
cp config/config.example.yaml config/config.yaml
# Edit config/config.yaml with your API keys and preferences

# Initialize database
python scripts/init_db.py
```

### Running the Application
```bash
# Run the news collector (one-time)
python -m src.main collect

# Run the email digest generator
python -m src.main digest --type weekly

# Start the scheduler (for automated runs)
python -m src.main scheduler

# Run Discord bot (Phase 2)
python -m src.discord_bot.main
```

### Testing
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_collectors.py

# Run with coverage
pytest --cov=src tests/
```

### Development Tools
```bash
# Format code
black src/ tests/

# Lint code
flake8 src/ tests/

# Type checking
mypy src/
```

## Architecture Overview

### Phase 1: Email System
- **Collectors**: Gather news from RSS feeds, APIs (Reddit, HN, GitHub)
- **Processors**: Deduplicate, score importance, categorize content
- **Services**: Generate email content via Claude API, send via SendGrid
- **Scheduler**: Automate weekly (Saturday), daily, and breaking news alerts

### Phase 2: Discord Bot
- **Vector Store**: ChromaDB for semantic search
- **RAG Pipeline**: Retrieve relevant news for user queries
- **Commands**: /news search, /news latest, /news explain
- **Memory**: Conversation context per user

## Configuration

The system uses a YAML configuration file (config/config.yaml) with:
- Source configurations (RSS feeds, API credentials)
- Email settings (provider, schedules, templates)
- AI provider settings (Claude/OpenAI API)
- Database connection
- Discord bot settings (Phase 2)

## Key Design Decisions

1. **Modular Architecture**: Each component (collector, processor, service) is independent
2. **Importance Scoring**: Multi-factor algorithm to identify breaking news
3. **Deduplication**: Content similarity matching to avoid duplicate stories
4. **Flexible Scheduling**: Configurable delivery times and frequencies
5. **RAG Implementation**: Vector search with metadata filtering for accurate retrieval

## Common Tasks

### Adding a New RSS Feed
1. Add feed URL to config/config.yaml under sources.rss_feeds
2. Set appropriate weight and category
3. Test with: `python -m src.collectors.rss_collector test <feed_url>`

### Adjusting Importance Scoring
1. Modify weights in config/config.yaml under processing.scoring
2. Test with sample data: `python scripts/test_scoring.py`

### Debugging Email Generation
1. Set log_level to DEBUG in config
2. Use dry-run mode: `python -m src.main digest --dry-run`
3. Check generated content in logs/email_previews/

## Important Notes

- Always use environment variables for API keys (never commit them)
- The Saturday weekly digest is the default but fully configurable
- Breaking news alerts have a cooldown period to avoid spam
- Discord bot requires separate deployment consideration
- Vector embeddings are cached to reduce API costs