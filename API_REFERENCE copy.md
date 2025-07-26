# Skop PDF Scraper API Reference

**Base URL:** `https://api.skop.dev`

## Authentication

All endpoints require authentication   key in the Authorization header:

```javascript
const headers = {
  'Authorization': 'Bearer sk-your-api-key-here',
  'Content-Type': 'application/json'
}
```

**API Key Formats Supported:**
- `sk-xxxxxxxxxxxxx` (recommended)
- `sk_xxxxxxxxxxxxx` (also supported)

## API Endpoints

### 1. Health Check

**GET** `/health/`

Check if the API is healthy and operational.

```javascript
const response = await fetch('https://api.skop.dev/health/')
```

**Response:**
```json
{
  "status": "healthy",
  "service": "skop", 
  "version": "1.0.0",
  "timestamp": "2025-07-24T20:50:00Z",
  "dependency_status": "operational"
}
```

### 2. Create Scraping Job

**POST** `/scrape/`

Start a new document scraping job.

**Request Body:**
```json
{
  "website": "https://example.com",
  "prompt": "Find all board meeting minutes from 2025",
  "parameters": {
    "single_page": false,
    "timeout": 1800,
    "confidence_threshold": 0.7,
    "file_type": "document",
    "max_file_size_mb": 100
  }
}
```

**JavaScript Example:**
```javascript
const createJob = async () => {
  const response = await fetch('https://api.skop.dev/scrape/', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer sk-your-api-key',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      website: 'https://example.com',
      prompt: 'Find meeting minutes for 2025',
      parameters: {
        single_page: true,
        timeout: 1800,
        confidence_threshold: 0.7,
        file_type: 'document',
        max_file_size_mb: 100
      }
    })
  })
  
  return await response.json()
}
```

**Response (201 Created):**
```json
{
  "job_id": "job_4fc79a89797e",
  "status": "pending",
  "message": "Job created successfully and queued for processing",
  "estimated_completion": "2025-07-24T21:00:00Z",
  "created_at": "2025-07-24T20:50:00Z"
}
```

### 3. Check Job Status

**GET** `/scrape/status/{job_id}`

Get the current status of a scraping job.

**JavaScript Example:**
```javascript
const checkStatus = async (jobId) => {
  const response = await fetch(`https://api.skop.dev/scrape/status/${jobId}`, {
    headers: {
      'Authorization': 'Bearer sk-your-api-key'
    }
  })
  
  return await response.json()
}
```

**Response:**
```json
{
  "job_id": "job_4fc79a89797e",
  "status": "completed",
  "message": "Job completed successfully",
  "progress": null,
  "created_at": "2025-07-24T20:50:02Z",
  "started_at": "2025-07-24T20:50:02Z",
  "estimated_completion": "2025-07-24T20:55:02Z",
  "completed_at": "2025-07-24T20:50:53Z",
  "current_agent": null,
  "agents_completed": [],
  "total_pages_crawled": 1,
  "total_documents_found": 10,
  "errors": [],
  "warnings": [],
  "is_active": false,
  "has_errors": false
}
```

### 4. Get Job Results

**GET** `/scrape/results/{job_id}`

Get the final results and extracted documents from a completed job.

**JavaScript Example:**
```javascript
const getResults = async (jobId) => {
  const response = await fetch(`https://api.skop.dev/scrape/results/${jobId}`, {
    headers: {
      'Authorization': 'Bearer sk-your-api-key'
    }
  })
  
  return await response.json()
}
```

**Response:**
```json
{
  "job_id": "job_4fc79a89797e",
  "status": "completed",
  "message": "Job completed successfully. Found 10 document(s).",
  "documents": [
    {
      "name": "June 23, 2025 minutes",
      "url": "https://example.com/docs/minutes.pdf",
      "source_page": "https://example.com/meetings",
      "document_type": "pdf",
      "confidence_score": 0.65,
      "file_size_mb": 0.59,
      "extracted_at": "2025-07-24T20:50:53Z"
    }
  ],
  "metadata": {
    "job_id": "job_4fc79a89797e",
    "created_at": "2025-07-24T20:50:02Z",
    "completed_at": "2025-07-24T20:50:53Z",
    "original_website": "https://example.com",
    "original_prompt": "find meeting minutes for 2025",
    "parameters": {
      "single_page": true,
      "timeout": 1800,
      "confidence_threshold": 0.7,
      "file_type": "document"
    }
  },
  "total_documents_found": 10,
  "total_documents_downloaded": 10,
  "total_pages_crawled": 1,
  "success_rate": 1.0,
  "cost": 0.18,
  "total_runtime_seconds": 50.89,
  "runtime_minutes": 0.85,
  "errors": [],
  "warnings": [],
  "has_errors": false
}
```

### 5. Cancel Job

**DELETE** `/scrape/{job_id}`

Cancel a running job.

**JavaScript Example:**
```javascript
const cancelJob = async (jobId) => {
  const response = await fetch(`https://api.skop.dev/scrape/${jobId}`, {
    method: 'DELETE',
    headers: {
      'Authorization': 'Bearer sk-your-api-key'
    }
  })
  
  return await response.json()
}
```

## Request Parameters

### ScrapeRequest

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `website` | string (URL) | ✅ | Starting URL to scrape |
| `prompt` | string | ✅ | Description of documents to find (10-500 chars) |
| `parameters` | object | ❌ | Additional scraping configuration |

### ScrapeParameters

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `single_page` | boolean | `false` | Only scrape the provided URL (no navigation) |
| `timeout` | integer | `1800` | Job timeout in seconds (60-3600) |
| `confidence_threshold` | float | `0.1` | Minimum AI confidence score (0.0-1.0) |
| `file_type` | string | `"document"` | Type of files to extract |
| `max_file_size_mb` | integer | `100` | Max file size to download (1-500 MB) |

### File Types

- `"document"` - All document types (PDF, DOC, DOCX, TXT, etc.)

## Job Status Values

| Status | Description |
|--------|-------------|
| `pending` | Job queued, waiting to start |
| `in_progress` | Job is currently running |
| `completed` | Job finished successfully |
| `failed` | Job encountered an error |
| `cancelled` | Job was cancelled by user |

## Error Responses

All endpoints return errors in this format:

```json
{
  "error": true,
  "message": "Invalid API key",
  "status_code": 401,
  "path": "/scrape/",
  "timestamp": "2025-07-24T20:50:00Z"
}
```

### Common Error Codes

| Code | Description |
|------|-------------|
| `400` | Bad Request - Invalid parameters |
| `401` | Unauthorized - Invalid/missing API key |
| `402` | Payment Required - Insufficient credits |
| `403` | Forbidden - Insufficient permissions |
| `404` | Not Found - Job/resource not found |
| `429` | Too Many Requests - Rate limit exceeded |
| `500` | Internal Server Error |

## Rate Limits

- **General endpoints**: 100 requests/hour
- **Health check**: 300 requests/hour
- **Status check**: 200 requests/hour

Rate limit headers are included in responses:
- `X-Rate-Limit-Remaining`
- `X-Rate-Limit-Reset`

## TypeScript Types

```typescript
interface ScrapeRequest {
  website: string
  prompt: string
  parameters?: ScrapeParameters
}

interface ScrapeParameters {
  single_page?: boolean
  timeout?: number
  confidence_threshold?: number
  file_type?: FileType
  max_file_size_mb?: number
}

type FileType = 'document'

type JobStatus = 'pending' | 'in_progress' | 'completed' | 'failed' | 'cancelled'

interface JobCreateResponse {
  job_id: string
  status: JobStatus
  message: string
  estimated_completion?: string
  created_at: string
}

interface JobStatusResponse {
  job_id: string
  status: JobStatus
  message: string
  created_at: string
  started_at?: string
  completed_at?: string
  estimated_completion?: string
  total_pages_crawled: number
  total_documents_found: number
  is_active: boolean
  has_errors: boolean
  errors: JobError[]
  warnings: string[]
}

interface JobResultsResponse {
  job_id: string
  status: JobStatus
  message: string
  documents: ExtractedDocument[]
  total_documents_found: number
  total_documents_downloaded: number
  total_pages_crawled: number
  success_rate: number
  cost: number
  total_runtime_seconds: number
  runtime_minutes: number
  has_errors: boolean
  errors: JobError[]
  warnings: string[]
}

interface ExtractedDocument {
  name: string
  url: string
  source_page: string
  document_type: string
  confidence_score: number
  file_size_mb?: number
  extracted_at: string
}

interface JobError {
  error_id: string
  agent_name: string
  error_type: string
  error_message: string
  page_url?: string
  timestamp: string
  is_recoverable: boolean
}
```

## Complete Example: Polling Workflow

```javascript
const scrapeDocuments = async (website, prompt, options = {}) => {
  const apiKey = 'sk-your-api-key'
  const baseURL = 'https://api.skop.dev'
  
  // 1. Create job
  const job = await fetch(`${baseURL}/scrape/`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      website,
      prompt,
      parameters: {
        single_page: false,
        confidence_threshold: 0.7,
        file_type: 'document',
        ...options
      }
    })
  }).then(r => r.json())
  
  console.log('Job created:', job.job_id)
  
  // 2. Poll for completion
  const pollStatus = async () => {
    const status = await fetch(`${baseURL}/scrape/status/${job.job_id}`, {
      headers: { 'Authorization': `Bearer ${apiKey}` }
    }).then(r => r.json())
    
    console.log('Status:', status.status, `(${status.total_documents_found} docs found)`)
    
    if (status.is_active) {
      // Still running, poll again in 5 seconds
      setTimeout(pollStatus, 5000)
    } else {
      // Job completed, get results
      const results = await fetch(`${baseURL}/scrape/results/${job.job_id}`, {
        headers: { 'Authorization': `Bearer ${apiKey}` }
      }).then(r => r.json())
      
      console.log('Results:', results)
      return results
    }
  }
  
  return pollStatus()
}

// Usage
scrapeDocuments('https://example.com', 'find meeting minutes for 2025')
  .then(results => console.log(`Found ${results.documents.length} documents`))
```

## Billing

- **Single page jobs**: $0.18 (pages with results only)
- **Multi-page jobs**: $0.25 (pages with results only)
- No charge for pages without relevant documents
- Real-time credit deduction during job execution 