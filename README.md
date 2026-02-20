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

### Base URL
```
https://api.askpoly.ai/api/v1/frontend
```

### Authentication

AskPoly supports **three authentication methods** for maximum flexibility. Choose the method that best fits your use case:

#### Method 1: JWT Token Authentication (Frontend/Web Applications)

JWT tokens are ideal for browser-based applications and frontend integrations. Tokens are obtained during user login and provide session-based authentication.

**When to use:**
- Building web applications with user sessions
- Frontend frameworks (React, Vue, Angular)
- Mobile apps with user authentication

**Header Format:**
```
Authorization: Bearer YOUR_JWT_TOKEN
```

**Example:**
```bash
curl -X POST https://api.askpoly.ai/api/v1/frontend/ask \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{"query": "iPhone 16"}'
```

---

#### Method 2: API Key Authentication (Programmatic Access)

API keys are designed for server-to-server integrations, scripts, and programmatic access. Generate API keys from your account dashboard.

**When to use:**
- Backend services and microservices
- Scripts and automation tools
- Third-party integrations
- Command-line tools

**Tier Requirements:**
- ✅ **Pro tier and above** - Can generate and use API keys
- ❌ **Free tier** - API key access not available

**Generating an API Key:**
1. Log into your AskPoly account at https://askpoly.ai
2. Navigate to **Settings > API Keys**
3. Click **Generate New API Key**
4. Copy your API key (format: `spk_live_xxxxx` for production)
5. Store it securely - you won't be able to see it again

**Header Format:**
```
X-API-Key: YOUR_API_KEY
```

**Example:**
```bash
curl -X POST https://api.askpoly.ai/api/v1/frontend/ask \
  -H "X-API-Key: spk_live_23aeKPBqhXwzyj3fY94_o1Ai7FzIe6zZ" \
  -H "Content-Type: application/json" \
  -d '{"query": "iPhone 16"}'
```

**Python Example:**
```python
import requests

API_KEY = "spk_live_your_api_key_here"

response = requests.post(
    "https://api.askpoly.ai/api/v1/frontend/ask",
    headers={
        "X-API-Key": API_KEY,
        "Content-Type": "application/json"
    },
    json={"query": "iPhone 16"}
)

print(response.json())
```

---

#### Method 3: MCP Service-Key Authentication (AI Agent Integrations)

MCP (Model Context Protocol) service keys enable service-to-service authentication for AI tools such as ChatGPT MCP servers, Cursor, and third-party AI agents. MCP requests bypass credit consumption entirely, making this ideal for platform integrations.

**When to use:**
- ChatGPT MCP server integrations
- Cursor and other AI coding assistants
- Third-party AI agents calling AskPoly on behalf of users
- Any service-to-service integration where credits are managed externally

**Three Required Headers:**

All three headers must be present together. If the service key is provided but either identity or request ID is missing, the API returns `400 Bad Request`.

| Header | Description | Example |
|--------|-------------|---------|
| `X-ASKPOLY-SERVICE-KEY` | Service key provided by AskPoly | `sk_mcp_your_service_key` |
| `X-ASKPOLY-MCP-IDENTITY` | Caller identity with prefix `cgpt:` or `anon:` (max 128 chars, regex `^[a-zA-Z0-9:_-]+$`) | `cgpt:user123` or `anon:session456` |
| `X-ASKPOLY-REQUEST-ID` | Unique UUID for request tracing | `550e8400-e29b-41d4-a716-446655440000` |

**Authentication Priority:**

When multiple authentication methods are present, AskPoly checks them in this order:
1. **MCP Service Key** (`X-ASKPOLY-SERVICE-KEY`) - checked first
2. **JWT Token** (`Authorization: Bearer ...`) - checked second
3. **API Key** (`X-API-Key`) - checked last

**Credit Bypass:**

MCP requests have zero credit consumption. The response metadata includes:
- `billing_source: "mcp_trial"` - indicates MCP billing path
- `mcp_identity` - the caller identity passed in the request
- `mcp_request_id` - the request ID for tracing

**Example (curl):**
```bash
curl -X POST https://api.askpoly.ai/api/v1/frontend/ask \
  -H "X-ASKPOLY-SERVICE-KEY: sk_mcp_your_service_key" \
  -H "X-ASKPOLY-MCP-IDENTITY: cgpt:user123" \
  -H "X-ASKPOLY-REQUEST-ID: 550e8400-e29b-41d4-a716-446655440000" \
  -H "Content-Type: application/json" \
  -d '{"query": "iPhone 16"}'
```

**Example (Python):**
```python
import requests
import uuid

SERVICE_KEY = "sk_mcp_your_service_key"

response = requests.post(
    "https://api.askpoly.ai/api/v1/frontend/ask",
    headers={
        "X-ASKPOLY-SERVICE-KEY": SERVICE_KEY,
        "X-ASKPOLY-MCP-IDENTITY": "cgpt:user123",
        "X-ASKPOLY-REQUEST-ID": str(uuid.uuid4()),
        "Content-Type": "application/json"
    },
    json={"query": "iPhone 16"}
)

data = response.json()
print(f"Billing source: {data['metadata']['billing_source']}")  # "mcp_trial"
print(f"Credits used: {data['credits_used']}")  # 0
```

**Error Responses:**

| Scenario | Status | Message |
|----------|--------|---------|
| Service key valid but identity header missing | 400 | `MCP service-key requests require X-ASKPOLY-MCP-IDENTITY and X-ASKPOLY-REQUEST-ID headers` |
| Service key valid but identity format invalid | 400 | `Invalid X-ASKPOLY-MCP-IDENTITY format. Must start with 'cgpt:' or 'anon:'` |
| Identity exceeds 128 characters | 400 | `X-ASKPOLY-MCP-IDENTITY exceeds maximum length (128 chars)` |
| Identity contains invalid characters | 400 | `X-ASKPOLY-MCP-IDENTITY contains invalid characters` |
| Invalid service key | 401 | `Not authenticated` |
| MCP_SERVICE_KEY not configured on server | 401 | `Not authenticated` |

**Supported Endpoints:**

All four query APIs support MCP service-key authentication:
- `POST /api/v1/frontend/ask` (0 credits via MCP)
- `POST /api/v1/frontend/compare` (0 credits via MCP)
- `POST /api/v1/frontend/recommend` (0 credits via MCP)
- `POST /api/v1/frontend/think` (0 credits via MCP)

---

### Important Notes

- **Same Endpoints, Same Responses**: All four query APIs (ASK, COMPARE, RECOMMEND, THINK) work identically with all authentication methods
- **Same Credit Usage**: Credit deduction is identical for JWT and API key methods
- **MCP Service Key**: For AI agent integrations (ChatGPT MCP, Cursor, third-party agents), use the MCP service key method. All three headers (`X-ASKPOLY-SERVICE-KEY`, `X-ASKPOLY-MCP-IDENTITY`, `X-ASKPOLY-REQUEST-ID`) are required together. MCP requests bypass credit consumption.
- **Choose One Method**: Use JWT token, API key, or MCP service key in a single request - not multiple methods simultaneously
- **Security**: Never expose API keys or service keys in client-side code or public repositories
- **Tier Check**: API keys automatically enforce tier restrictions (free tier users cannot use API keys)

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

| API | Purpose | Credit Cost | Typical Response Time | Recommended Timeout |
|-----|---------|-------------|----------------------|---------------------|
| **ASK** | Quick sentiment analysis and insights about any product | 1 credit | 15-20 seconds | 30 seconds |
| **COMPARE** | Side-by-side comparison of two products | 2 credits | 40-50 seconds | 90 seconds |
| **RECOMMEND** | AI-powered product recommendations based on needs | 2 credits | 8-12 seconds | 60 seconds |
| **THINK** | Deep market analysis and comprehensive reports | 3 credits | 50-60 seconds | 90 seconds |

> **MCP Service-Key Note:** All four APIs support MCP service-key authentication with **zero credit cost**. When authenticated via MCP service key, the `credits_used` field returns `0` and `billing_source` is set to `"mcp_trial"` in response metadata. See [Method 3: MCP Service-Key Authentication](#method-3-mcp-service-key-authentication-ai-agent-integrations) for details.

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

  "resolved_entities": [    // NEW: Canonical product metadata (October 2025)
    {
      "name": "Apple Vision Pro",
      "type": "product",
      "brand": "Apple",
      "product_name": "Vision Pro",
      "model": null,
      "category": "Electronics",
      "source": "database-catalog",
      "confidence": 0.95
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

### Resolved Entities Field (NEW - October 2025)

The `resolved_entities` field provides canonical product metadata extracted from the query. This enables:

- **Product Identification**: Exact brand, product name, and model detection
- **Category Classification**: Automatic product category assignment
- **Confidence Scoring**: Reliability metric for entity resolution
- **Data Source Tracking**: Origin of product information (database-catalog, semantic-extraction, etc.)

**Use Cases**:
- Display product details in UI without re-parsing query text
- Filter and categorize results by product metadata
- Track which products users are searching for
- Implement product-specific features based on category

**Field Structure**:
```typescript
interface ResolvedEntity {
  name: string;           // Full product name (e.g., "Apple Vision Pro")
  type: string;           // Entity type (always "product" currently)
  brand: string;          // Brand name (e.g., "Apple")
  product_name: string;   // Product without brand (e.g., "Vision Pro")
  model: string | null;   // Model identifier if applicable (e.g., "Pro", "15")
  category: string;       // Product category (e.g., "Electronics", "Appliances")
  source: string;         // Data source (e.g., "database-catalog")
  confidence: number;     // Confidence score 0.0-1.0
}
```

**Universal Category Support** (October 2025):
The system now supports **ANY product category** without code changes:
- Technology: iPhone, Galaxy, Pixel, MacBook
- Appliances: Washing machines, refrigerators, dishwashers
- Fashion: Levi's jeans, Nike sneakers, Crocs
- Food: Oreo Thins, Lay's chips, Ben & Jerry's
- Toys: Fisher-Price, LEGO, Hot Wheels
- Automotive: Tesla Model 3, Ford F-150, Toyota Camry
- Furniture: IKEA desks, Herman Miller chairs
- **Future products automatically supported**

### Optional Advanced Response Fields

The ASK API includes additional optional fields that provide enhanced error handling, performance metrics, and AI feedback:

| Field | Type | Description |
|-------|------|-------------|
| `error_code` | string (optional) | Standardized error code for programmatic error handling (e.g., "INSUFFICIENT_DATA", "RATE_LIMIT") |
| `details` | object (optional) | Additional error or debug information for troubleshooting |
| `poly_response` | object (optional) | Poly AI feedback system data including quality scores and model metadata |
| `data_sources` | object (optional) | Breakdown of data sources used (e.g., `{"reddit": 45, "x": 23, "youtube": 12}`) |
| `processing_time_ms` | integer (optional) | Total request processing time in milliseconds for performance monitoring |

**Example with advanced fields:**
```json
{
  "success": true,
  "query_id": "550e8400-e29b-41d4-a716-446655440000",
  "sentiment_summary": { ... },
  "poly_index": 78,

  "data_sources": {
    "reddit": 650,
    "x": 400,
    "youtube": 200
  },
  "processing_time_ms": 2150,
  "poly_response": {
    "quality_score": 0.95,
    "model_version": "gpt-4"
  }
}
```

### Example Implementation

```python
import requests

def analyze_product(product_name, api_key):
    """Get quick insights about a product with canonical metadata"""

    response = requests.post(
        'https://api.askpoly.ai/api/v1/frontend/ask',
        headers={
            'X-API-Key': api_key,
            'Content-Type': 'application/json'
        },
        json={'query': product_name},
        timeout=30  # ASK API typically takes 15-20 seconds
    )

    if response.status_code == 200:
        data = response.json()
        print(f"Product: {product_name}")
        print(f"Sentiment: {data['sentiment_summary']['overall_sentiment']}")
        print(f"Poly Index: {data['poly_index']}/100")

        # NEW: Access canonical product metadata
        if data.get('resolved_entities'):
            entity = data['resolved_entities'][0]
            print(f"Brand: {entity['brand']}")
            print(f"Category: {entity['category']}")
            print(f"Confidence: {entity['confidence']:.2f}")
        print(f"Trend: {data['trend']['direction']}")

        # Optional: Monitor performance
        if data.get('processing_time_ms'):
            print(f"Processing time: {data['processing_time_ms']}ms")

        return data
    else:
        print(f"Error: {response.status_code}")
        return None

# Usage
result = analyze_product("Tesla Model 3", "spk_live_your_key")
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
| session_id | string | No | Optional session ID for tracking related queries |
| impression_id | string | No | Optional impression tracking ID for analytics |

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

### Optional Advanced Response Fields

The COMPARE API includes additional optional fields for enhanced functionality:

| Field | Type | Description |
|-------|------|-------------|
| `error_code` | string (optional) | Standardized error code for programmatic error handling |
| `details` | object (optional) | Additional error or debug information |
| `poly_response` | object (optional) | Poly AI feedback system data including quality scores and model metadata |
| `processing_time_ms` | integer (optional) | Total request processing time in milliseconds |

**Example with advanced fields:**
```json
{
  "success": true,
  "comparison_id": "660e8400-e29b-41d4-a716-446655440000",
  "products": [...],
  "insights": {...},

  "processing_time_ms": 3450,
  "poly_response": {
    "quality_score": 0.92,
    "model_version": "gpt-4"
  }
}
```

### Example Implementation

```javascript
async function compareProducts(product1, product2, apiKey) {
  const response = await fetch('https://api.askpoly.ai/api/v1/frontend/compare', {
    method: 'POST',
    headers: {
      'X-API-Key': apiKey,
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
| answers | object | No | Answers to previous clarifying questions (simple key-value pairs) |
| structured_answers | array | No | Structured clarification responses with detailed format |
| skip_clarification | boolean | No | Skip clarification questions and return recommendations immediately (default: false) |

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

  "next_questions": [  // Enhanced - intelligent clarification questions
    {
      "id": "motivation",
      "question": "What's driving your laptop search?",
      "options": ["current laptop too slow", "need better performance for work", "upgrading old device", "first-time buyer"],
      "type": "single_choice"
    },
    {
      "id": "budget_comfort",
      "question": "What's your budget comfort zone?",
      "options": ["student-friendly under $800", "professional $1200-1800", "high-end $2000+"],
      "type": "single_choice"
    }
  ],

  "credits_used": 2,
  "processing_time_ms": 4200
}
```

### Optional Advanced Response Fields

The RECOMMEND API includes additional optional fields for enhanced functionality:

| Field | Type | Description |
|-------|------|-------------|
| `error_code` | string (optional) | Standardized error code for programmatic error handling |
| `details` | object (optional) | Additional error or debug information |
| `poly_response` | object (optional) | Poly AI feedback system data including quality scores and model metadata |
| `processing_time_ms` | integer (optional) | Total request processing time in milliseconds |

**Advanced Request Example:**
```json
{
  "query": "best gaming laptop",
  "skip_clarification": true,
  "structured_answers": [
    {
      "question_id": "budget",
      "answer": "under $1500"
    }
  ]
}
```

**Example response with advanced fields:**
```json
{
  "success": true,
  "session_id": "770e8400-e29b-41d4-a716-446655440000",
  "recommendations": [...],

  "processing_time_ms": 4200,
  "poly_response": {
    "quality_score": 0.89,
    "model_version": "gpt-4"
  }
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
            headers={
                'X-API-Key': self.api_key,
                'Content-Type': 'application/json'
            },
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
session = RecommendationSession('spk_live_your_key')

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
| session_id | string | No | Optional session ID for tracking related queries |

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

### Optional Advanced Response Fields

The THINK API includes additional optional fields for enhanced functionality:

| Field | Type | Description |
|-------|------|-------------|
| `error_code` | string (optional) | Standardized error code for programmatic error handling |
| `details` | object (optional) | Additional error or debug information |
| `poly_response` | object (optional) | Poly AI feedback system data including quality scores and model metadata |

**Example response with advanced fields:**
```json
{
  "success": true,
  "analysis_id": "880e8400-e29b-41d4-a716-446655440000",
  "comprehensive_report": {...},
  "data_analyzed": {...},
  "processing_time_seconds": 18.5,
  "credits_used": 3,

  "poly_response": {
    "quality_score": 0.94,
    "model_version": "gpt-4",
    "reasoning_depth": "comprehensive"
  }
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
        'X-API-Key': apiKey,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ query }),
      timeout: 90000  // 90 second timeout (THINK API typically takes 50-60s)
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
  'spk_live_your_key'
);
```

---

## Backend Management APIs

**New in January 2025**: Complete backend management capabilities for enhanced user experience and application integration.

### Authentication & Session Management

#### Enhanced Login with Remember Me
```
POST https://api.askpoly.ai/api/v1/auth/login
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "password123",
  "remember_me": true
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "expires_in": 1209600,
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "tier": "pro",
    "role": "admin",
    "display_name": "John Doe"
  }
}
```

**Remember Me Options:**
- `remember_me: true` → Access token: 14 days, Refresh token: 30 days
- `remember_me: false` → Access token: 1 day, Refresh token: 7 days

#### Token Refresh with Rotation
```
POST https://api.askpoly.ai/api/v1/auth/refresh
```

**Request Body:**
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response:** Returns new access and refresh tokens (old refresh token becomes invalid)

#### Multi-Device Session Management
```
GET https://api.askpoly.ai/api/v1/auth/sessions
```

**Response:**
```json
{
  "sessions": [
    {
      "session_id": "550e8400-e29b-41d4-a716-446655440000",
      "device_name": "Chrome on MacOS",
      "device_type": "browser",
      "ip_address": "192.168.1.1",
      "created_at": "2025-01-20T10:30:00Z",
      "last_active": "2025-01-20T15:45:00Z",
      "is_current": true,
      "location": "San Francisco, CA"
    }
  ],
  "total_count": 1
}
```

#### Revoke Session
```
DELETE https://api.askpoly.ai/api/v1/auth/sessions/{session_id}
```

#### Enhanced Logout
```
POST https://api.askpoly.ai/api/v1/auth/logout
```

**Request Body (Optional):**
```json
{
  "logout_all": true
}
```

### Query History Management

#### List Query History with Advanced Filtering
```
GET https://api.askpoly.ai/api/v1/query/history
```

**Query Parameters:**
- `limit`: Number of items (default: 50, max: 100)
- `offset`: Pagination offset
- `query_type`: Filter by ask/compare/recommend/think
- `search`: Full-text search
- `is_favorite`: Filter favorites only
- `session_id`: Filter by session/workspace
- `start_date`/`end_date`: Date range filtering

**Response:**
```json
{
  "items": [
    {
      "id": "123e4567-e89b-12d3-a456-426614174000",
      "query_type": "ask",
      "query_text": "What do people think about iPhone 15?",
      "display_label": "iPhone 15 sentiment analysis",
      "session_id": "550e8400-e29b-41d4-a716-446655440000",
      "created_at": "2025-01-20T14:30:00Z",
      "updated_at": "2025-01-20T14:30:05Z",
      "is_favorite": false,
      "credits_used": 2,
      "response_time_ms": 2500,
      "summary": {
        "sentiment_score": 0.75,
        "mention_count": 156,
        "key_insight": "Generally positive reception with concerns about price",
        "poly_index": 8.2
      },
      "metadata": {
        "source_count": 45,
        "platforms": ["reddit", "x", "youtube"],
        "language": "en"
      },
      "response_data": {
        "poly_response": "Complete AI response preserved...",
        "trending_topics": ["battery life", "camera quality", "price"],
        "sources": [...]
      }
    }
  ],
  "pagination": {
    "current_page": 1,
    "total_pages": 5,
    "page_size": 20,
    "total_items": 95,
    "has_next": true,
    "has_previous": false
  }
}
```

#### Update Query History
```
PATCH https://api.askpoly.ai/api/v1/query/history/{query_id}
```

**Request Body:**
```json
{
  "display_label": "Custom query name",
  "is_favorite": true
}
```

#### Delete Query History
```
DELETE https://api.askpoly.ai/api/v1/query/history/{query_id}
```

Performs soft delete with audit trail preservation.

#### Create Query History Entry
```
POST https://api.askpoly.ai/api/v1/query/history
```

**Request Body:**
```json
{
  "query_type": "ask",
  "query_text": "What do people think about iPhone 15?",
  "display_label": "iPhone 15 sentiment analysis",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "credits_used": 2,
  "response_time_ms": 2500,
  "summary": { "sentiment_score": 0.75, "mention_count": 156 },
  "metadata": { "source_count": 45, "platforms": ["reddit", "x"] },
  "response_data": { "poly_response": "Complete analysis..." }
}
```

### Profile & Preferences Management

#### Update User Profile
```
PATCH https://api.askpoly.ai/api/v1/auth/me
```

**Request Body:**
```json
{
  "display_name": "John Doe",
  "organization": "Acme Corp",
  "timezone": "America/New_York",
  "use_case": "Market research"
}
```

#### Get User Preferences
```
GET https://api.askpoly.ai/api/v1/auth/preferences
```

#### Update All Preferences
```
PUT https://api.askpoly.ai/api/v1/auth/preferences
```

#### Partial Preference Update
```
PATCH https://api.askpoly.ai/api/v1/auth/preferences
```

### Usage Analytics

#### Query Statistics
```
GET https://api.askpoly.ai/api/v1/query/stats
```

**Response:**
```json
{
  "total_queries": 150,
  "queries_by_type": {
    "ask": 75,
    "compare": 35,
    "recommend": 25,
    "think": 15
  },
  "total_credits_consumed": 285,
  "queries_last_24h": 12,
  "queries_last_7d": 47,
  "queries_last_30d": 150,
  "favorite_count": 18,
  "most_frequent_topics": [
    {"topic": "iPhone 15 vs Samsung Galaxy S24", "frequency": 8},
    {"topic": "Best gaming laptop 2025", "frequency": 5}
  ]
}
```

#### Advanced Analytics
```
GET https://api.askpoly.ai/api/v1/query/analytics?days=30
```

**Response:**
```json
{
  "usage_over_time": [
    {"date": "2025-01-25", "queries": 15, "credits": 28},
    {"date": "2025-01-24", "queries": 8, "credits": 12}
  ],
  "peak_hours": [
    {"hour": 14, "count": 25},
    {"hour": 10, "count": 18}
  ],
  "query_type_distribution": {
    "ask": 50.0,
    "compare": 23.3,
    "recommend": 16.7,
    "think": 10.0
  },
  "average_response_time_ms": 3250.5,
  "success_rate": 98.7,
  "sentiment_trends": {
    "data": [
      {"date": "2025-01-25", "positive": 65.2, "negative": 12.1}
    ],
    "trend": "positive"
  },
  "top_entities": [
    {"entity": "iPhone 15", "count": 23},
    {"entity": "Tesla Model 3", "count": 15}
  ]
}
```

**Benefits of Backend Management APIs:**
- **Enhanced Security**: Multi-device session management with revocation
- **Rich User Experience**: Complete query history with search and organization
- **Analytics Insights**: Detailed usage patterns and trends
- **Profile Customization**: Personalized user profiles and preferences
- **Session Continuity**: Extended login sessions with remember me functionality

---

## Popular Searches API

### What It Does
The Popular Searches API returns trending product searches across the AskPoly platform. Results are privacy-filtered to remove any personally identifiable information and aggregated to show community-level trends.

**No authentication required** - this is a public endpoint.

### Endpoint
```
GET /api/v1/public/popular-searches
```

### Example Request
```bash
curl https://api.askpoly.ai/api/v1/public/popular-searches
```

### Example Response
```json
{
  "success": true,
  "popular_searches": [
    {
      "query": "iPhone 16",
      "search_count": 245,
      "category": "Electronics"
    },
    {
      "query": "Tesla Model 3",
      "search_count": 189,
      "category": "Automotive"
    }
  ],
  "updated_at": "2026-02-20T10:00:00Z"
}
```

### Use Cases
- Display trending searches on your application's landing page
- Suggest popular queries to new users
- Monitor market interest trends across product categories

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
            raise AuthenticationError('Invalid API key or JWT token')

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
                headers={
                    'X-API-Key': self.api_key,
                    'Content-Type': 'application/json'
                },
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
| PAYG | 20 | Pay-per-use | 2 |
| Pro | 60 | 500 | 5 |
| Business | 120 | 1000 | 10 |
| Enterprise | 300 | Unlimited | 20 |
| MCP Service | 60 | Unlimited (bypass) | 5 |

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

### 5. Set Appropriate Request Timeouts

AskPoly APIs perform deep analysis across millions of social media posts, which requires processing time. Always set appropriate timeouts to avoid premature failures:

**Recommended Timeouts by API:**
```python
TIMEOUT_CONFIG = {
    'ask': 30,      # ASK API: 15-20s typical, 30s timeout
    'compare': 90,  # COMPARE API: 40-50s typical, 90s timeout
    'recommend': 60, # RECOMMEND API: 8-12s typical, 60s timeout
    'think': 90     # THINK API: 50-60s typical, 90s timeout
}

def make_api_request(endpoint, payload, api_key):
    """Make API request with appropriate timeout"""
    timeout = TIMEOUT_CONFIG.get(endpoint, 60)

    response = requests.post(
        f'https://api.askpoly.ai/api/v1/frontend/{endpoint}',
        headers={'X-API-Key': api_key, 'Content-Type': 'application/json'},
        json=payload,
        timeout=timeout
    )
    return response.json()
```

**Why Response Times Vary:**
- Data volume: More social media posts = longer processing
- Query complexity: Detailed comparisons take more time
- Real-time analysis: Fresh data requires live processing
- AI reasoning: Deep analysis requires extensive LLM processing

**Handling Timeouts:**
```python
import time

def api_call_with_retry(endpoint, payload, api_key, max_retries=2):
    """Retry API calls on timeout"""
    for attempt in range(max_retries + 1):
        try:
            return make_api_request(endpoint, payload, api_key)
        except requests.Timeout:
            if attempt < max_retries:
                wait_time = (attempt + 1) * 5  # 5s, 10s
                print(f'Timeout occurred. Retrying in {wait_time}s...')
                time.sleep(wait_time)
            else:
                raise TimeoutError(f'{endpoint.upper()} API timed out after {max_retries} retries')
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

### v2.11.0 (February 2026)
- BYOK (Bring Your Own Key) Upload-Post integration
- Social sharing with user's own API key support

### v2.10.0 (February 2026)
- Compare API Phase 2: word boundary matching for entity resolution
- Confidence gating for comparison results
- Entity resolution cache aliasing for faster lookups

### v2.9.9 (January 2026)
- **MCP Service-Key Credit Bypass** for all query APIs (Ask, Compare, Recommend, Think)
- Service-to-service authentication via `X-ASKPOLY-SERVICE-KEY` header
- Zero credit consumption for MCP-authenticated requests
- `billing_source: "mcp_trial"` in response metadata

### v2.9.0 (January 2026)
- AI/GPT products enrichment system with custom JSON support
- 110+ products from OpenAI, Anthropic, Google, Meta, and more

### v2.8.0 (January 2026)
- Relevant hashtags added to social share posts

### v2.7.0 (January 2026)
- Sentiment pipeline fix achieving 100% coverage for sentiment scores

### v2.6.0 (December 2025)
- Infographic generation with logo support
- Product abbreviation handling improvements

### v2.5.0 (November 2025)
- Amazon Reviews real-time integration

### v2.4.0 (November 2025)
- Popular Searches public API (`GET /api/v1/public/popular-searches`)
- No authentication required, privacy-filtered results

### v2.3.0 (November 2025)
- Hybrid authentication (API key + JWT) for all query endpoints
- Share-to-Earn v2.2 security fix

### v2.2.0 (October 2025)
- Universal entity resolution supporting any product category
- Trends rollup system for historical sentiment tracking

### v2.1.0 (October 2025)
- `resolved_entities` field added to all API responses
- Brand relationships and canonical product metadata

### v1.1.0 (September 2025)
- Enhanced ASK API with structured JSON responses
- Improved THINK API performance by 85%
- Added health check endpoints
- Fixed sentiment calculation accuracy

### v1.0.0 (August 2025)
- Initial release with four core APIs
- Launch of credit-based pricing model