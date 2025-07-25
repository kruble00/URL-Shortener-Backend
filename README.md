# URL Shortener Backend (AWS Lambda)

A serverless URL shortening service backend built with AWS Lambda, DynamoDB, and API Gateway. This repository contains the backend API that powers the URL shortener application.

## 🏗️ Architecture

- **AWS Lambda**: Serverless compute for API endpoints
- **DynamoDB**: NoSQL database for URL mappings
- **API Gateway**: REST API management and routing
- **Serverless Framework**: Infrastructure as Code deployment

## 📋 Features

- ✅ Create shortened URLs with 6-character random codes
- ✅ User-specific URL management (isolated by Clerk user ID)
- ✅ URL deletion functionality
- ✅ Redirect service for shortened URLs
- ✅ CORS enabled for frontend integration
- ✅ Collision detection for short codes
- ✅ Input validation and error handling

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ installed
- AWS CLI configured with appropriate permissions
- Serverless Framework installed globally

### Installation

1. **Clone the repository**
```bash
git clone <repository-url>
cd url-shortener-lambda
```

2. **Install dependencies**
```bash
npm install
```

3. **Configure environment variables**
```bash
# Optional: Set custom domain name
export DOMAIN_NAME=your-domain.com
```

4. **Deploy to AWS**
```bash
npm run deploy
```

The deployment will output your API Gateway URL - save this for frontend configuration.

## 📡 API Endpoints

### POST /shorten
Create a new shortened URL.

**Request:**
```json
{
  "originalUrl": "https://example.com/very-long-url",
  "userId": "user_clerk_id_here"
}
```

**Response:**
```json
{
  "shortCode": "abc123",
  "shortUrl": "https://your-domain.com/r/abc123"
}
```

### GET /urls/{userId}
Retrieve all URLs for a specific user.

**Response:**
```json
{
  "urls": [
    {
      "userId": "user_clerk_id",
      "shortCode": "abc123",
      "originalUrl": "https://example.com/very-long-url",
      "createdAt": 1640995200000
    }
  ]
}
```

### DELETE /urls/{userId}/{shortCode}
Delete a specific URL mapping.

**Response:**
```json
{
  "success": true
}
```

### GET /r/{shortCode}
Redirect to the original URL (returns HTTP 301).

## 🗄️ Database Schema

### DynamoDB Table: `url-mappings`

| Attribute | Type | Description |
|-----------|------|-------------|
| `userId` (PK) | String | Clerk user identifier |
| `shortCode` (SK) | String | 6-character random string |
| `originalUrl` | String | The full URL to redirect to |
| `createdAt` | Number | Unix timestamp |

### Access Patterns
- **Get user URLs**: Query by `userId`
- **Get specific mapping**: GetItem by `userId` + `shortCode`
- **Delete mapping**: DeleteItem by `userId` + `shortCode`
- **Global redirect lookup**: Scan by `shortCode` (for redirect service)

## ⚙️ Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DYNAMODB_TABLE_NAME` | DynamoDB table name | Auto-generated |
| `DOMAIN_NAME` | Domain for short URLs | `short.ly` |

### AWS Permissions

The Lambda function requires the following DynamoDB permissions:
- `dynamodb:Query`
- `dynamodb:Scan`
- `dynamodb:GetItem`
- `dynamodb:PutItem`
- `dynamodb:DeleteItem`

## 🔧 Development

### Local Testing

```bash
# Install Serverless Framework plugins for local development
npm install -g serverless-offline

# Run locally
serverless offline
```

### Available Scripts

```bash
npm run deploy          # Deploy to AWS
npm run remove          # Remove AWS resources
npm run logs            # View Lambda logs
```

### Code Structure

```
src/
└── index.js           # Main Lambda handler (95 lines)
```

The main handler contains all API logic including:
- URL shortening with collision detection
- CRUD operations for URL mappings
- Redirect functionality
- CORS handling
- Error management

## 🚀 Deployment

### Using Serverless Framework

```bash
# Deploy to dev stage (default)
serverless deploy

# Deploy to production
serverless deploy --stage prod

# Deploy to custom region
serverless deploy --region eu-west-1
```

### CI/CD Integration

For automated deployments, add these steps to your CI pipeline:

```yaml
- name: Deploy Lambda
  run: |
    npm install
    npx serverless deploy --stage prod
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## 📊 Monitoring

### CloudWatch Metrics
- Function duration and memory usage
- Error rates and invocation counts
- DynamoDB read/write capacity

### Logging
All function logs are available in CloudWatch:
```bash
# View logs
serverless logs -f api

# Tail logs in real-time
serverless logs -f api --tail
```

## 💰 Cost Estimation

### Monthly Costs (Estimated)
- **Lambda**: $0-2 (1M requests in free tier)
- **DynamoDB**: $1-5 (depending on usage)
- **API Gateway**: $1-3 (per 1M requests)

**Total**: ~$2-10/month for moderate usage

## 🔒 Security

### Current Implementation
- Input validation for URLs
- User data isolation via Clerk user IDs
- CORS configuration for frontend security
- HTTPS enforcement through API Gateway

### Recommended Enhancements
- API rate limiting
- URL validation against malicious sites
- WAF rules for additional protection

## 🐛 Troubleshooting

### Common Issues

**Permission Denied Errors**
```bash
# Verify AWS credentials
aws sts get-caller-identity

# Check IAM permissions for DynamoDB
```

**CORS Issues**
- Verify frontend domain in CORS headers
- Check API Gateway CORS configuration

**Function Timeout**
- Check CloudWatch logs for errors
- Consider increasing timeout in `serverless.yml`

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## 🔗 Related Repositories

- **Frontend**: [url-shortener-frontend](../url-shortener-frontend) - Next.js web application
