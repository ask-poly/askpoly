# AskPoly AI - API Documentation

## About AskPoly

AskPoly is an AI-powered sentiment analysis platform that transforms real-time social media conversations into actionable product insights. By analyzing millions of discussions across Reddit, X (Twitter), and YouTube, AskPoly helps businesses, researchers, and consumers understand authentic public opinion about products, brands, and market trends.

Our platform uses advanced natural language processing and machine learning to:
- **Analyze sentiment** across multiple social platforms in real-time
- **Compare products** based on genuine user feedback and experiences
- **Recommend products** that match specific needs based on community discussions
- **Provide deep insights** into market trends and competitive dynamics

Whether you're a product manager tracking feature reception, a consumer researching your next purchase, or a market analyst understanding industry trends, AskPoly delivers data-driven insights from the world's largest focus group - social media.

---

## Getting Started

### API Keys

To use AskPoly APIs, you need an API key generated from your account dashboard:

1. Log into your AskPoly account at https://askpoly.ai
2. Navigate to **Settings > API Keys**
3. Click **Generate New API Key**
4. Copy your API key (format: `apk_live_xxxxx` for production or `apk_test_xxxxx` for testing)
5. Store it securely - you won't be able to see it again

### Base URL
```
https://api.askpoly.ai/api/v1/frontend
```

### Authentication

Include your API key in the Authorization header for all requests:
```
Authorization: Bearer apk_live_your_api_key_here
```

Example:
```bash
curl -H "Authorization: Bearer apk_live_abc123xyz789" \
     https://api.askpoly.ai/api/v1/frontend/ask
```

---

## API Health Check

Before integrating with AskPoly APIs, always verify the service is operational:

### Main Health Check Endpoint
```
GET https://api.askpoly.ai/health
```

**No authentication required**

#### Success Response (200 OK)
```json
{
  "status": "healthy",
  "timestamp": "2025-09-17T10:30:00Z",
  "version": "1.1.0",
  "services": {
    "database": "connected",
    "redis": "connected",
    "neo4j": "connected",
    "ml_models": "loaded"
  }
}
```

#### Error Response (503 Service Unavailable)
```json
{
  "status": "unhealthy",
  "timestamp": "2025-09-17T10:30:00Z",
  "services": {
    "database": "connected",
    "redis": "error: connection refused",
    "neo4j": "connected",
    "ml_models": "loaded"
  }
}
```

### Simple Health Check
```
GET https://api.askpoly.ai/api/v1/health/simple
```

Returns plain text response:
- `OK` - All services operational (HTTP 200)
- `ERROR` - Service issues detected (HTTP 503)

### Implementation Example

```javascript
async function checkAPIHealth() {
  try {
    const response = await fetch('https://api.askpoly.ai/health');
    const health = await response.json();

    if (health.status === 'healthy') {
      console.log('✅ AskPoly API is operational');
      return true;
    } else {
      console.warn('⚠️ AskPoly API has issues:', health.services);
      return false;
    }
  } catch (error) {
    console.error('❌ Cannot reach AskPoly API:', error);
    return false;
  }
}

// Check health before making API calls
if (await checkAPIHealth()) {
  // Proceed with API calls
} else {
  // Implement fallback or retry logic
}
```

---

## Available APIs

AskPoly provides four specialized APIs, each designed for different use cases:

| API | Purpose | Credit Cost | Response Time |
|-----|---------|-------------|---------------|
| **ASK** | Quick sentiment analysis and insights about any product | 1 credit | 2-5 seconds |
| **COMPARE** | Side-by-side comparison of two products | 2 credits | 3-6 seconds |
| **RECOMMEND** | AI-powered product recommendations based on needs | 2 credits | 4-7 seconds |
| **THINK** | Deep market analysis and comprehensive reports | 3 credits | 15-25 seconds |

---

## 1. ASK API - Quick Product Insights

### What It Does
The ASK API is your go-to endpoint for quick sentiment analysis about any product or brand. It analyzes the last 30 days of social media discussions to provide:
- Overall sentiment scores and distribution
- Key themes in user discussions
- Actual user quotes and opinions
- Trending indicators and momentum
- Platform-specific sentiment breakdown

Perfect for: Product managers monitoring launches, marketers tracking brand perception, or consumers researching purchases.

### Endpoint
`POST /api/v1/frontend/ask`

### Request
```json
{
  "query": "Apple Vision Pro"
}
```

### Parameters
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| query | string | Yes | Product or brand name to analyze (1-500 characters) |

### Response Structure
```json
{
  "success": true,
  "query_id": "550e8400-e29b-41d4-a716-446655440000",
  "query_text": "Apple Vision Pro",
  "query_language": "en",
  "query_timestamp": "2025-09-17T10:30:00Z",

  "sentiment_summary": {
    "overall_sentiment": "positive",  // "positive", "negative", "neutral", or "mixed"
    "positive_percentage": 65.5,       // Percentage of positive discussions
    "negative_percentage": 15.3,       // Percentage of negative discussions
    "neutral_percentage": 19.2,        // Percentage of neutral discussions
    "confidence_score": 0.92,          // AI confidence in analysis (0-1)
    "total_analyzed": 1250            // Number of posts analyzed
  },

  "volume_metrics": {
    "total_posts": 1250,              // Total discussions found
    "unique_authors": 892,            // Unique users discussing
    "engagement_total": 45670,        // Total likes, comments, shares
    "platform_breakdown": {
      "reddit": {
        "posts": 650,
        "engagement": 23000,
        "sentiment": {
          "positive": 68.0,
          "negative": 12.0,
          "neutral": 20.0
        }
      },
      "x": { /* Same structure */ },
      "youtube": { /* Same structure */ }
    },
    "time_range": "last_30_days"
  },

  "poly_index": 78,  // Proprietary sentiment score (0-100)

  "trend": {
    "direction": "increasing",    // "increasing", "decreasing", or "stable"
    "change_percentage": 12.5,    // Change from previous period
    "momentum": "positive"         // Overall momentum indicator
  },

  "what_people_love": [
    "Display quality and immersion",
    "Spatial computing capabilities",
    "Hand tracking accuracy"
  ],

  "top_concerns": [
    "High price point",
    "Limited battery life",
    "Weight during extended use"
  ],

  "quote_samples": [
    {
      "platform": "reddit",
      "source": "r/AppleVisionPro",
      "content": "After a month of daily use, the Vision Pro has completely changed...",
      "sentiment": "positive",
      "engagement_metrics": {
        "likes": 234,
        "comments": 45
      },
      "created_at": "2025-09-15T14:30:00Z"
    }
  ],

  "credits_used": 1,
  "credits_remaining": 99,
  "metadata": {
    "processing_time_ms": 2150,
    "data_sources_queried": ["reddit", "x", "youtube"],
    "cache_hit": false
  }
}
```

### Example Implementation

```python
import requests

def analyze_product(product_name, api_key):
    """Get quick insights about a product"""

    response = requests.post(
        'https://api.askpoly.ai/api/v1/frontend/ask',
        headers={'Authorization': f'Bearer {api_key}'},
        json={'query': product_name}
    )

    if response.status_code == 200:
        data = response.json()
        print(f"Product: {product_name}")
        print(f"Sentiment: {data['sentiment_summary']['overall_sentiment']}")
        print(f"Poly Index: {data['poly_index']}/100")
        print(f"Trend: {data['trend']['direction']}")
        return data
    else:
        print(f"Error: {response.status_code}")
        return None

# Usage
result = analyze_product("Tesla Model 3", "apk_live_your_key")
```

---

## 2. COMPARE API - Product Comparison

### What It Does
The COMPARE API performs head-to-head sentiment comparison between two products. It analyzes how users perceive each product relative to the other, identifying:
- Sentiment scores and winner determination
- Key strengths and weaknesses of each product
- Common themes and differentiators
- Platform-specific performance
- Direct comparison metrics

Perfect for: Competitive analysis, purchase decisions between alternatives, or market positioning studies.

### Endpoint
`POST /api/v1/frontend/compare`

### Request
```json
{
  "product1": "iPhone 15 Pro",
  "product2": "Samsung Galaxy S24"
}
```

### Parameters
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| product1 | string | Yes | First product name (1-200 characters) |
| product2 | string | Yes | Second product name (1-200 characters) |

### Response Structure
```json
{
  "success": true,
  "comparison_id": "660e8400-e29b-41d4-a716-446655440000",

  "products": [
    {
      "name": "iPhone 15 Pro",
      "sentiment_score": 0.75,
      "total_mentions": 3456,
      "sentiment_distribution": {
        "positive": 68.5,
        "negative": 12.3,
        "neutral": 19.2
      },
      "key_strengths": [
        "Camera quality in low light",
        "iOS ecosystem integration",
        "Build quality and design"
      ],
      "key_weaknesses": [
        "Price premium",
        "Limited customization"
      ],
      "trending_score": 0.82
    },
    {
      "name": "Samsung Galaxy S24",
      // Same structure as above
    }
  ],

  "insights": {
    "winner": "iPhone 15 Pro",
    "winner_margin": 3.0,
    "key_differentiators": [
      "iPhone leads in overall sentiment by 3 percentage points",
      "Samsung has stronger YouTube presence",
      "iPhone shows higher engagement rates"
    ],
    "recommendation": "iPhone 15 Pro edges ahead with slightly better sentiment...",
    "common_themes": [
      "Camera quality discussions",
      "Price-to-value debates"
    ]
  },

  "comparison_summary": {
    "head_to_head": {
      "sentiment_winner": "iPhone 15 Pro",
      "volume_winner": "iPhone 15 Pro",
      "growth_winner": "Samsung Galaxy S24",
      "engagement_winner": "iPhone 15 Pro"
    }
  },

  "credits_used": 2,
  "processing_time_ms": 3450
}
```

### Example Implementation

```javascript
async function compareProducts(product1, product2, apiKey) {
  const response = await fetch('https://api.askpoly.ai/api/v1/frontend/compare', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ product1, product2 })
  });

  const result = await response.json();

  if (result.success) {
    console.log(`Winner: ${result.insights.winner}`);
    console.log(`Margin: ${result.insights.winner_margin}%`);

    // Display key differentiators
    result.insights.key_differentiators.forEach(diff => {
      console.log(`• ${diff}`);
    });
  }

  return result;
}
```

---

## 3. RECOMMEND API - Product Recommendations

### What It Does
The RECOMMEND API provides intelligent product recommendations based on natural language queries. It understands context, requirements, and preferences to suggest products that best match user needs based on actual user experiences and discussions. Features include:
- Natural language understanding of requirements
- Multi-turn conversation support for refinement
- Category auto-detection
- Sentiment-backed recommendations
- Alternative suggestions

Perfect for: E-commerce integrations, buying guides, personalized shopping assistants, or research tools.

### Endpoint
`POST /api/v1/frontend/recommend`

### Request
```json
{
  "query": "best laptop for video editing under $2000",
  "category": "laptops",           // Optional - auto-detected
  "brand_preference": "Apple",     // Optional
  "session_id": "previous-session", // Optional - for follow-ups
  "answers": {                     // Optional - previous Q&A
    "priority": "performance",
    "portability": "not important"
  }
}
```

### Parameters
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| query | string | Yes | Natural language description of needs (5-500 chars) |
| category | string | No | Product category (auto-detected if not provided) |
| brand_preference | string | No | Preferred brand filter |
| session_id | string | No | Previous session ID for multi-turn conversation |
| answers | object | No | Answers to previous clarifying questions |

### Response Structure
```json
{
  "success": true,
  "session_id": "770e8400-e29b-41d4-a716-446655440000",

  "recommendations": [
    {
      "rank": 1,
      "product": "MacBook Pro 14-inch M3 Pro",
      "category": "laptops",
      "brand": "Apple",
      "sentiment_score": 0.88,
      "confidence": 0.92,
      "total_mentions": 2340,
      "reasoning": "Top choice for video editing with excellent performance...",
      "pros": [
        "Exceptional performance for video editing",
        "Outstanding battery life",
        "Professional-grade display"
      ],
      "cons": [
        "Higher price point",
        "Limited port selection"
      ],
      "price_range": "$1,999",
      "alternatives": ["MacBook Pro 16-inch", "Mac Studio"],
      "user_sentiment": {
        "positive": 82.5,
        "negative": 7.3,
        "neutral": 10.2
      }
    }
    // Additional recommendations...
  ],

  "reasoning": {
    "status": "complete",
    "message": "Found 3 highly-rated laptops matching your criteria",
    "factors_considered": [
      "Video editing performance",
      "Price within budget",
      "User sentiment from creators"
    ]
  },

  "next_questions": [  // Optional - for refinement
    {
      "id": "screen_size",
      "question": "Do you prefer 14-inch portable or 16-inch larger display?",
      "options": ["14-inch portable", "16-inch larger", "No preference"],
      "type": "single_choice"
    }
  ],

  "credits_used": 2,
  "processing_time_ms": 4200
}
```

### Multi-Turn Conversation Example

```python
class RecommendationSession:
    def __init__(self, api_key):
        self.api_key = api_key
        self.session_id = None
        self.base_url = 'https://api.askpoly.ai/api/v1/frontend/recommend'

    def get_recommendations(self, query, answers=None):
        """Get or refine recommendations"""

        payload = {'query': query}
        if self.session_id:
            payload['session_id'] = self.session_id
        if answers:
            payload['answers'] = answers

        response = requests.post(
            self.base_url,
            headers={'Authorization': f'Bearer {self.api_key}'},
            json=payload
        )

        result = response.json()
        self.session_id = result.get('session_id')

        return result

    def answer_questions(self, answers):
        """Refine recommendations with answers"""
        return self.get_recommendations(
            query=None,  # Use original query from session
            answers=answers
        )

# Usage
session = RecommendationSession('apk_live_your_key')

# Initial recommendation
result = session.get_recommendations('gaming laptop')

# Answer follow-up questions if any
if result.get('next_questions'):
    refined = session.answer_questions({
        'budget': 'under $1500',
        'graphics': 'RTX 4060 or better'
    })
```

---

## 4. THINK API - Deep Analysis

### What It Does
The THINK API performs comprehensive, AI-powered analysis of complex queries about products, markets, or trends. Unlike other endpoints, THINK uses all available historical data and advanced reasoning to provide:
- Executive summaries and key findings
- Detailed market analysis
- Trend evolution and projections
- Competitive dynamics
- Data-backed recommendations
- Sentiment evolution over time

Perfect for: Market research, investment analysis, strategic planning, trend reports, or any scenario requiring deep insights.

### Endpoint
`POST /api/v1/frontend/think`

### Request
```json
{
  "query": "How is the electric vehicle market evolving and what are the key competitive dynamics between Tesla, Rivian, and traditional automakers?"
}
```

### Parameters
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| query | string | Yes | Complex question or analysis request (10-1000 characters) |

### Response Structure
```json
{
  "success": true,
  "analysis_id": "880e8400-e29b-41d4-a716-446655440000",
  "query": "How is the electric vehicle market evolving...",

  "comprehensive_report": {
    "executive_summary": "The EV market is experiencing rapid transformation...",

    "key_findings": [
      {
        "finding": "Tesla maintains 60% market share but declining from 75%",
        "confidence": 0.91,
        "supporting_data": {
          "mentions": 12450,
          "sentiment": 0.72,
          "sources": ["Industry reports", "User discussions"]
        }
      },
      {
        "finding": "Traditional automakers gaining ground",
        "confidence": 0.88,
        "supporting_data": {
          "mentions": 8900,
          "sentiment": 0.68
        }
      }
    ],

    "detailed_analysis": {
      "market_dynamics": "Comprehensive competitive landscape analysis...",
      "consumer_sentiment": "What buyers are saying about each brand...",
      "trend_analysis": "Historical and projected trends...",
      "competitive_positioning": "How each brand is positioned..."
    },

    "recommendations": [
      "Tesla should focus on quality improvements",
      "Rivian needs to accelerate production",
      "Traditional automakers should emphasize reliability"
    ],

    "market_insights": {
      "growth_rate": "28% YoY",
      "market_size": "$400B by 2030",
      "key_drivers": ["Government incentives", "Battery costs", "Infrastructure"],
      "challenges": ["Supply chain", "Charging anxiety", "Price premiums"]
    },

    "sentiment_evolution": {
      "6_months_ago": 0.65,
      "3_months_ago": 0.70,
      "current": 0.73,
      "trend": "improving",
      "drivers_of_change": ["Better charging", "More options", "Price drops"]
    }
  },

  "data_analyzed": {
    "total_posts": 45670,
    "unique_authors": 28900,
    "platforms": ["reddit", "x", "youtube"],
    "time_span": "all_available",
    "earliest_data": "2023-01-01",
    "latest_data": "2025-09-17",
    "data_quality_score": 0.92
  },

  "processing_time_seconds": 18.5,
  "credits_used": 3
}
```

### Example Implementation

```javascript
async function deepAnalysis(query, apiKey) {
  console.log('Starting deep analysis (this may take 15-25 seconds)...');

  const startTime = Date.now();

  try {
    const response = await fetch('https://api.askpoly.ai/api/v1/frontend/think', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ query }),
      timeout: 30000  // 30 second timeout
    });

    const result = await response.json();
    const elapsed = (Date.now() - startTime) / 1000;

    if (result.success) {
      console.log(`✅ Analysis complete in ${elapsed}s`);
      console.log('\nExecutive Summary:');
      console.log(result.comprehensive_report.executive_summary);

      console.log('\nKey Findings:');
      result.comprehensive_report.key_findings.forEach((finding, i) => {
        console.log(`${i + 1}. ${finding.finding} (confidence: ${finding.confidence})`);
      });

      return result;
    } else {
      console.error('Analysis failed:', result.error);
      return null;
    }
  } catch (error) {
    console.error('Request error:', error);
    return null;
  }
}

// Usage
const analysis = await deepAnalysis(
  'What are the emerging trends in AR/VR gaming and how are consumers responding?',
  'apk_live_your_key'
);
```

---

## Error Handling

### Common Error Responses

#### Insufficient Credits (402)
```json
{
  "error": "Insufficient credits",
  "credits_required": 2,
  "current_balance": 1,
  "message": "You need at least 2 credits to use the Compare feature"
}
```

#### Invalid API Key (401)
```json
{
  "error": "Authentication required",
  "message": "Invalid API key or key expired"
}
```

#### Rate Limit (429)
```json
{
  "error": "Rate limit exceeded",
  "message": "Too many requests",
  "retry_after": 60
}
```

#### Invalid Request (400)
```json
{
  "error": "Invalid request",
  "message": "Query parameter is required",
  "details": {
    "field": "query",
    "issue": "missing"
  }
}
```

### Error Handling Implementation

```python
class AskPolyClient:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = 'https://api.askpoly.ai/api/v1/frontend'

    def _handle_response(self, response):
        """Unified error handling for all API calls"""

        if response.status_code == 200:
            return response.json()

        elif response.status_code == 401:
            raise AuthenticationError('Invalid API key')

        elif response.status_code == 402:
            error_data = response.json()
            raise InsufficientCreditsError(
                f"Need {error_data['credits_required']} credits, "
                f"have {error_data['current_balance']}"
            )

        elif response.status_code == 429:
            retry_after = response.json().get('retry_after', 60)
            raise RateLimitError(f'Rate limited. Retry after {retry_after}s')

        elif response.status_code == 400:
            error_data = response.json()
            raise ValidationError(error_data.get('message', 'Invalid request'))

        elif response.status_code == 503:
            raise ServiceUnavailableError('AskPoly service temporarily unavailable')

        else:
            raise APIError(f'Unexpected error: {response.status_code}')

    def ask(self, query):
        """Call ASK API with error handling"""
        try:
            response = requests.post(
                f'{self.base_url}/ask',
                headers={'Authorization': f'Bearer {self.api_key}'},
                json={'query': query},
                timeout=10
            )
            return self._handle_response(response)

        except requests.Timeout:
            raise TimeoutError('Request timed out')
        except requests.ConnectionError:
            raise ConnectionError('Cannot connect to AskPoly API')
```

---

## Rate Limits

| Plan | Requests/Minute | Daily Credits | Concurrent Requests |
|------|----------------|---------------|-------------------|
| Free | 10 | 10 | 1 |
| Starter | 30 | 100 | 3 |
| Pro | 60 | 500 | 5 |
| Enterprise | 300 | Unlimited | 20 |

---

## Best Practices

### 1. Always Check Service Health First
```javascript
async function initializeAPI() {
  // Check health before starting
  const isHealthy = await checkHealth();

  if (!isHealthy) {
    console.error('AskPoly API is not available');
    // Implement fallback or wait
    return false;
  }

  // Proceed with API operations
  return true;
}
```

### 2. Implement Retry Logic with Exponential Backoff
```python
import time

def retry_api_call(func, max_retries=3):
    """Retry API calls with exponential backoff"""

    for attempt in range(max_retries):
        try:
            return func()
        except RateLimitError as e:
            if attempt == max_retries - 1:
                raise
            wait_time = 2 ** attempt  # 1, 2, 4 seconds
            time.sleep(wait_time)
        except ServiceUnavailableError:
            if attempt == max_retries - 1:
                raise
            time.sleep(5)  # Wait 5 seconds for service issues

    return None
```

### 3. Cache Responses to Save Credits
```javascript
class ResponseCache {
  constructor(ttlMinutes = 60) {
    this.cache = new Map();
    this.ttl = ttlMinutes * 60 * 1000;
  }

  getCacheKey(endpoint, params) {
    return `${endpoint}:${JSON.stringify(params)}`;
  }

  get(endpoint, params) {
    const key = this.getCacheKey(endpoint, params);
    const cached = this.cache.get(key);

    if (cached && Date.now() - cached.timestamp < this.ttl) {
      console.log('Cache hit - saving credits');
      return cached.data;
    }

    return null;
  }

  set(endpoint, params, data) {
    const key = this.getCacheKey(endpoint, params);
    this.cache.set(key, {
      data: data,
      timestamp: Date.now()
    });
  }
}
```

### 4. Monitor Credit Usage
```python
def track_credit_usage(response):
    """Monitor and alert on credit usage"""

    if 'credits_remaining' in response:
        remaining = response['credits_remaining']

        # Alert on low credits
        if remaining < 10:
            send_alert(f'Low credits warning: {remaining} remaining')

        # Log usage for analytics
        log_credit_usage({
            'timestamp': datetime.now(),
            'endpoint': response.get('endpoint'),
            'credits_used': response.get('credits_used'),
            'remaining': remaining
        })
```

---

## Support

### Technical Support
- Email: support@askpoly.ai
- Response time: Within 24 hours for Pro/Enterprise

### Resources
- API Status: https://status.askpoly.ai
- Documentation: https://docs.askpoly.ai
- Community Forum: https://community.askpoly.ai

### Reporting Issues
When reporting issues, please include:
1. Your API request (without API key)
2. Response received
3. Timestamp of the request
4. Your account email

---

## Changelog

### v1.1.0 (September 2025)
- Enhanced ASK API with structured JSON responses
- Improved THINK API performance by 85%
- Added health check endpoints
- Fixed sentiment calculation accuracy

### v1.0.0 (August 2025)
- Initial release with four core APIs
- Launch of credit-based pricing model