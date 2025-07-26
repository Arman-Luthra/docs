# Skop API Reference

## Overview

Skop is an intelligent multi-agent document discovery and extraction service that can find and extract PDF documents from websites based on natural language prompts. The API uses advanced AI agents to navigate websites, identify relevant documents, and extract them with high accuracy.

**Base URL**: `https://api.skop.dev`  
**Version**: 1.0.0  
**Service Name**: Skop

## Authentication

All API endpoints require authentication using either:
- **Bearer Token**: Include `Authorization: Bearer <your_token>` in headers
- **API Key**: Include `X-API-Key: <your_api_key>` in headers

### Required Scopes
- `scrape:create` - Required for creating new scraping jobs
- Additional scopes may be required for other operations

## Billing & Pricing

### Pricing Model
The API uses a results-based billing system where you only pay for pages that contain actual documents:

- **Firecrawl Processing**: $0.18 per page with results (high-quality scraping)
- **Fast Scraper Processing**: $0.10 per page with results (basic scraping)

### Usage-Based Billing
The API uses a simple pay-per-use model where all users are charged based on actual results:

- **No monthly limits**: Use as much as you need
- **No plan restrictions**: All features available to all users
- **Fair pricing**: Only pay for pages that contain documents
- **Transparent costs**: Clear per-page pricing with no hidden fees

### Cost Estimation
- **Single Page**: ~$0.10-0.18 (depending on scraper used)
- **Multi-Page**: Variable based on documents found (typically $1-5 per job)

## Quickstart

### 1. Get Your API Key
Visit [https://www.skop.dev/dashboard](https://www.skop.dev/dashboard) to create an account and get your API key.

### 2. Make Your First Request
```bash
curl -X POST "https://api.skop.dev/scrape" \
  -H "Authorization: Bearer <your_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "website": "https://example.com",
    "prompt": "Find all board meeting minutes from 2024",
    "parameters": {
      "single_page": true,
      "timeout": 1800,
      "confidence_threshold": 0.7
    }
  }'
```

### 3. Check Job Status
```bash
curl -X GET "https://api.skop.dev/scrape/status/{job_id}" \
  -H "Authorization: Bearer <your_token>"
```

### 4. Get Results
```bash
curl -X GET "https://api.skop.dev/scrape/results/{job_id}" \
  -H "Authorization: Bearer <your_token>"
```

## Rate Limits

- **General endpoints**: 100 requests per hour
- **Job creation**: 100 requests per hour
- **Status checks**: 300 requests per hour
- **Health checks**: 60 requests per minute

Rate limit headers are included in responses:
- `X-Rate-Limit-Remaining`: Remaining requests
- `X-Rate-Limit-Reset`: Reset timestamp

## Request & Response Format

All requests and responses use JSON format with `Content-Type: application/json`.

### Error Response Format
```json
{
  "error": true,
  "message": "Human-readable error message",
  "status_code": 400,
  "path": "/scrape",
  "timestamp": "2024-01-15T10:00:00Z",
  "details": {
    "field": "website",
    "reason": "URL must use http or https protocol"
  }
}
```

---

# API Endpoints

## Create Scraping Job

**`POST /scrape/`**

Creates a new document scraping job that runs asynchronously in the background.

### Request Body

```json
{
  "website": "https://example.com",
  "prompt": "Find all board meeting minutes from 2024",
  "parameters": {
    "single_page": true,
    "timeout": 1800,
    "confidence_threshold": 0.7,
    "file_type": "document",
    "max_file_size_mb": 100
  }
}
```

### Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `website` | string (URL) | ✅ | Starting URL to scrape (must be http/https) |
| `prompt` | string | ✅ | Description of documents to find (10-500 chars) |
| `parameters` | object | ❌ | Additional scraping configuration |

### Parameters Object

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `single_page` | boolean | `true` | Only scrape the provided URL (no navigation) |
| `timeout` | integer | `1800` | Max time in seconds (60-3600) |
| `confidence_threshold` | float | `0.1` | Min AI confidence score (0.0-1.0) |  
| `file_type` | string | `"document"` | Type of files to extract |
| `max_file_size_mb` | integer | `100` | Max file size in MB (1-500) |

### Response

**Status**: `201 Created`

```json
{
  "job_id": "job_123456789",
  "status": "pending",
  "message": "Job created successfully and queued for processing",
  "estimated_completion": "2024-01-15T10:30:00Z",
  "created_at": "2024-01-15T10:00:00Z"
}
```

### Error Responses

| Status | Error Code | Description |
|--------|------------|-------------|
| `400` | `validation_error` | Invalid request parameters |
| `402` | `insufficient_credits` | Not enough credits |
| `429` | `concurrency_limit_exceeded` | Too many concurrent jobs |
| `503` | `service_unavailable` | Required services not configured |

---

## Get Job Status

**`GET /scrape/status/{job_id}`**

Returns the current status and progress of a scraping job.

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `job_id` | string | Unique job identifier |

### Response

**Status**: `200 OK`

```json
{
  "job_id": "job_123456789",
  "status": "extracting",
  "message": "Currently extracting documents from discovered pages",
  "created_at": "2024-01-15T10:00:00Z",
  "started_at": "2024-01-15T10:01:00Z",
  "estimated_completion": "2024-01-15T10:30:00Z",
  "completed_at": null,
  "current_agent": "scraper",
  "agents_completed": ["coordinator", "navigator"],
  "total_pages_crawled": 3,
  "total_documents_found": 12,
  "errors": [],
  "warnings": [],
  "is_active": true,
  "has_errors": false
}
```

### Job Status Values

| Status | Description |
|--------|-------------|
| `pending` | Job queued, waiting to start |
| `in_progress` | Job is actively running |
| `completed` | Job finished successfully |
| `failed` | Job failed due to errors |
| `cancelled` | Job was cancelled by user |

### Error Responses

| Status | Description |
|--------|-------------|
| `404` | Job not found or access denied |
| `500` | Internal server error |

---

## Get Job Results

**`GET /scrape/results/{job_id}`**

Returns the final results of a completed scraping job, including all extracted documents.

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `job_id` | string | Unique job identifier |

### Response

**Status**: `200 OK`

```json
{
  "job_id": "job_123456789",
  "status": "completed",
  "message": "Job completed successfully. Found 12 document(s).",
  "documents": [
    {
      "name": "Board Meeting Minutes - January 2024",
      "url": "https://example.com/docs/minutes-jan-2024.pdf",
      "source_page": "https://example.com/meetings",
      "document_type": "pdf",
      "confidence_score": 0.95,
      "file_size_mb": 2.3,
      "extracted_at": "2024-01-15T10:25:00Z"
    }
  ],
  "metadata": {
    "job_id": "job_123456789",
    "created_at": "2024-01-15T10:00:00Z",
    "started_at": "2024-01-15T10:01:00Z",
    "completed_at": "2024-01-15T10:28:00Z",
    "estimated_completion": "2024-01-15T10:30:00Z",
    "original_website": "https://example.com",
    "original_prompt": "Find all board meeting minutes from 2024",
    "parameters": {
      "single_page": false,
      "timeout": 1800,
      "confidence_threshold": 0.7
    },
    "current_agent": null,
    "agents_completed": ["coordinator", "navigator", "scraper"],
    "total_pages_crawled": 4,
    "total_documents_found": 15,
    "errors": [],
    "warnings": ["Some documents were too large to download"],
    "final_status": "completed"
  },
  "total_documents_found": 15,
  "total_documents_downloaded": 12,
  "total_pages_crawled": 4,
  "success_rate": 0.8,
  "cost": 2.16,
  "total_runtime_seconds": 1680,
  "runtime_minutes": 28.0,
  "errors": [],
  "warnings": ["Some documents were too large to download"],
  "has_errors": false
}
```

### Document Object

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Human-readable document name |
| `url` | string | Direct URL to the document |
| `source_page` | string | Page where document was found |
| `document_type` | string | File type (pdf, doc, etc.) |
| `confidence_score` | float | AI confidence (0.0-1.0) |
| `file_size_mb` | float | File size in megabytes |
| `extracted_at` | string | When document was extracted |

### Error Responses

| Status | Description |
|--------|-------------|
| `404` | Job not found or access denied |
| `500` | Internal server error |

---

## Cancel Job

**`DELETE /scrape/{job_id}`**

Cancels a running scraping job and stops further processing.

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `job_id` | string | Unique job identifier |

### Response

**Status**: `200 OK`

```json
{
  "message": "Job job_123456789 cancelled successfully",
  "status": "cancelled"
}
```

### Error Responses

| Status | Description |
|--------|-------------|
| `404` | Job not found or access denied |
| `500` | Internal server error |

---

## Health Check

**`GET /health/`**

Returns the overall health status of the API service and its dependencies.

### Response

**Status**: `200 OK`

```json
{
  "status": "healthy",
  "service": "skop",
  "version": "1.0.0",
  "timestamp": "2024-01-15T10:00:00Z",
  "dependency_status": "operational"
}
```

### Health Status Values

| Status | Description |
|--------|-------------|
| `healthy` | All systems operational |
| `degraded` | Some non-critical issues |
| `unhealthy` | Critical issues detected |

---

## Detailed Health Checks

### Dependencies Status

**`GET /health/dependencies`**

Returns detailed status of all external dependencies.

### Configuration Check

**`GET /health/configuration`**

Returns current configuration status without exposing sensitive values.

### Readiness Check

**`GET /health/ready`**

Kubernetes-style readiness check. Returns `200` if ready, `503` if not ready.

---

## Root Endpoint

**`GET /`**

Returns basic API information and available endpoints.

### Response

```json
{
  "message": "Skop API",
  "version": "1.0.0",
  "docs": "/docs",
  "health": "/health",
  "endpoints": {
    "scrape": "/scrape",
    "status": "/scrape/status/{job_id}",
    "results": "/scrape/results/{job_id}",
    "cancel": "/scrape/{job_id}",
    "health": "/health"
  }
}
```

---

## Error Handling

### Common Error Codes

| HTTP Status | Error Type | Description |
|-------------|------------|-------------|
| `400` | Bad Request | Invalid request format or parameters |
| `401` | Unauthorized | Missing or invalid authentication |
| `402` | Payment Required | Insufficient credits or billing issue |
| `403` | Forbidden | Access denied or feature not available |
| `404` | Not Found | Resource not found |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Internal Server Error | Unexpected server error |
| `503` | Service Unavailable | Service temporarily unavailable |

### Billing-Related Errors

#### Insufficient Credits
```json
{
  "error": "insufficient_credits",
  "message": "Insufficient credits. You need $2.50 more.",
  "estimated_cost": 5.00,
  "current_credits": 2.50,
  "shortfall": 2.50,
  "suggested_action": "Add more credits to your account"
}
```

#### Concurrency Limits
```json
{
  "error": "concurrency_limit_exceeded",
  "message": "You have reached the maximum number of concurrent jobs (5). Please wait for a job to complete before starting a new one.",
  "active_jobs": 5,
  "max_concurrent": 5,
  "suggested_action": "Wait for active jobs to complete before starting a new one"
}
```

---

## SDK and Integration Examples

### cURL Examples

#### Create a single-page job (default behavior)
```bash
curl -X POST "https://api.skop.dev/scrape" \
  -H "Authorization: Bearer <your_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "website": "https://example.com/documents",
    "prompt": "Find financial reports from Q4 2024",
    "parameters": {
      "confidence_threshold": 0.8
    }
  }'
```

#### Create a multi-page job
```bash
curl -X POST "https://api.skop.dev/scrape" \
  -H "Authorization: Bearer <your_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "website": "https://company.com",
    "prompt": "Find all investor presentations and annual reports from 2024",
    "parameters": {
      "single_page": false,
      "timeout": 3600,
      "confidence_threshold": 0.7,
      "max_file_size_mb": 200
    }
  }'
```

### Python Example

```python
import requests
import time

# Configuration
API_BASE = "https://api.skop.dev"
TOKEN = "your_bearer_token"
HEADERS = {
    "Authorization": f"Bearer {TOKEN}",
    "Content-Type": "application/json"
}

# Create job
job_data = {
    "website": "https://example.com",
    "prompt": "Find all policy documents related to data privacy",
    "parameters": {
        "single_page": False,  # Set to False for multi-page scraping
        "confidence_threshold": 0.8
    }
}

response = requests.post(f"{API_BASE}/scrape", json=job_data, headers=HEADERS)
job = response.json()
job_id = job["job_id"]

print(f"Created job: {job_id}")

# Poll for completion
while True:
    status_response = requests.get(f"{API_BASE}/scrape/status/{job_id}", headers=HEADERS)
    status = status_response.json()
    
    print(f"Status: {status['status']} - {status['message']}")
    
    if status["status"] in ["completed", "failed", "cancelled"]:
        break
    
    time.sleep(10)

# Get results
if status["status"] == "completed":
    results_response = requests.get(f"{API_BASE}/scrape/results/{job_id}", headers=HEADERS)
    results = results_response.json()
    
    print(f"Found {len(results['documents'])} documents")
    print(f"Total cost: ${results['cost']:.2f}")
    
    for doc in results["documents"]:
        print(f"- {doc['name']} (confidence: {doc['confidence_score']:.2f})")
```

---

## Best Practices

### Prompt Engineering
- **Be specific**: "Find quarterly earnings reports from 2024" vs "Find reports"
- **Include timeframes**: Specify date ranges when relevant
- **Mention document types**: "board meeting minutes", "financial statements", etc.
- **Use keywords**: Include relevant industry or company-specific terms

### Configuration Tips
- **Single-page mode**: Use for targeted document extraction from known pages
- **Confidence threshold**: Lower (0.1-0.3) for broader results, higher (0.7-0.9) for precision
- **Timeout**: Allow sufficient time for complex sites (1800-3600 seconds)
- **File size limits**: Balance between document completeness and processing time

### Error Handling
- Always check job status before requesting results
- Implement retry logic for transient errors
- Handle rate limiting with exponential backoff
- Monitor your credit balance for uninterrupted service

### Cost Optimization
- Use single-page mode when you know the exact page containing documents
- Set appropriate confidence thresholds to avoid false positives
- Monitor results to understand typical costs for your use cases
- Consider batching similar requests to optimize navigation

---

## Support

For API support, billing questions, or technical issues:
- Check the `/health/` endpoint for system status
- Review error messages for specific guidance
- Monitor rate limit headers to avoid throttling
- Visit [https://www.skop.dev](https://www.skop.dev) for documentation and support
- Access your dashboard at [https://www.skop.dev/dashboard](https://www.skop.dev/dashboard)

---

*Last updated: January 2025*