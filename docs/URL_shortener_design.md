1. High-Level Architecture
Components:
•	Client: Initiates requests to shorten URLs or access analytics.
•	API Gateway: Routes requests to respective services.
•	URL Shortening Service: Handles POST requests to shorten URLs.
•	URL Redirection Handler: Redirects requests based on short URL codes.
•	Analytics Service: Collects and serves analytics data.
•	NoSQL DB: Persistent store for URL mappings.
•	Cache: Fast-access store for frequently accessed short URLs.
•	In-memory Store: Temporary store for real-time analytics.

Data Flow:
•	Client sends POST to shorten URL → API Gateway → URL Shortening Service → Stores in NoSQL DB and Cache.
•	Client accesses short URL (GET) → API Gateway → Redirection Handler → Redirects using Cache (fallback to DB if cache miss).
•	Client requests analytics (GET) → API Gateway → Analytics Service → In-memory analytics data served (flushed periodically to DB).

2. Detailed Component Design
API Gateway
•	Routes requests to appropriate microservices.
•	Handles throttling, rate limiting, and basic auth.

URL Shortening Service
•	Accepts original URL.
•	Generates a unique hash or slug.
•	Stores mapping in NoSQL DB.
•	Updates cache for quick access.

URL Redirection Request Handler
•	Receives short URL code.
•	Checks cache for mapping.
•	If not found, queries NoSQL DB.
•	Responds with HTTP 301/302 redirect.
•	Sends access logs to analytics service.

Analytics Service
•	Collects hits, location, device info.
•	Stores data in in-memory DB.
•	Periodically flushes to persistent storage (NoSQL).

3. Databases and Caching
NoSQL DB (e.g., DynamoDB, MongoDB)
•	Stores: { short_code, original_url, metadata }
•	Indexed for fast lookups.
Cache (e.g., Redis)
•	Stores most accessed short_code → original_url.
•	TTL based eviction policy.
In-Memory Store
•	Used for real-time analytics aggregation.
•	Periodically flushed to NoSQL DB.

4.Scalability and Reliability
•	Horizontal scaling of all services.
•	Caching to reduce DB load.
•	DB replication and partitioning.
•	Circuit breakers and retries on failures.

5. API Design

POST /shorten
Request:
{
  "url": "https://example.com/very-long-url"
}

Response:
{
  "short_url": "https://short.ly/abc123"
}

GET /{short_code}
• Redirects to original URL.

GET /analytics/{short_code}
Response:
{
  "clicks": 1234,
  "browsers": {
    "Chrome": 800,
    "Firefox": 300
  },
  "countries": {
    "US": 900,
    "IN": 334
  }
}

6. Periodic Tasks
•	Flush in-memory analytics to NoSQL.
•	Clean up expired URLs.

7. Future Improvements
•	Add user accounts and history.
•	Expiry dates for short URLs.
•	Custom aliases for short URLs.
•	QR code generation.
