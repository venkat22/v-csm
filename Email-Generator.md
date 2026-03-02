# GPT-4 Email Generator with GPT-4o Critic Check

A serverless email generation service powered by OpenAI's GPT-4 for intelligent email composition, validated by GPT-4o for quality assurance. The service is hosted on AWS Lambda and exposed through AWS API Gateway.

## Features

- **GPT-4 Email Generation**: Leverage advanced AI to generate professional, contextual emails
- **GPT-4o Quality Validation**: Automated critic check using GPT-4o to ensure email quality, tone, and compliance
- **Serverless Architecture**: Deploy on AWS Lambda for scalability and cost-efficiency
- **API Gateway Integration**: RESTful API endpoints for seamless integration
- **Error Handling**: Comprehensive error handling and validation
- **Logging & Monitoring**: CloudWatch integration for tracking and debugging

## Architecture

```
┌─────────────────┐
│   API Client    │
└────────┬────────┘
         │ HTTP Request
         ▼
┌─────────────────────────────────────────┐
│      AWS API Gateway                    │
│  (HTTP to Lambda Bridge)                │
└────────┬────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│   AWS Lambda Function                   │
│  ┌─────────────────────────────────────┐│
│  │ 1. Parse Request                    ││
│  │ 2. Call GPT-4 for Email Generation  ││
│  │ 3. Call GPT-4o for Critic Check     ││
│  │ 4. Return Validated Email           ││
│  └─────────────────────────────────────┘│
└────────┬────────────────────────────────┘
         │
         ├─────────────────┬───────────────┤
         ▼                 ▼               ▼
    ┌────────────┐   ┌─────────────┐  ┌──────────┐
    │  OpenAI   │   │ CloudWatch  │  │ DynamoDB │
    │  API      │   │  Logs       │  │ (optional)
    │(GPT-4/4o) │   │             │  │          │
    └────────────┘   └─────────────┘  └──────────┘
```

## Prerequisites

- AWS Account with appropriate permissions (Lambda, API Gateway, IAM)
- OpenAI API Key (GPT-4 and GPT-4o access)
- Python 3.11+ (for Lambda runtime)
- AWS CLI configured locally
- Serverless Framework or AWS SAM (for deployment)

## Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/venkat22/v-csm.git
cd v-csm
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Environment Configuration

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_api_key_here
OPENAI_MODEL_GPT4=gpt-4
OPENAI_MODEL_GPT4O=gpt-4o
AWS_REGION=us-east-1
LOG_LEVEL=INFO
```

### 4. Deploy to AWS Lambda

#### Using AWS SAM:

```bash
sam build
sam deploy --guided
```

#### Using Serverless Framework:

```bash
serverless deploy
```

## API Endpoints

### Generate Email with Critic Check

**Endpoint:** `POST /email/generate`

**Request Body:**
```json
{
  "subject": "Meeting Follow-up",
  "recipient_name": "John Doe",
  "context": "Discuss Q2 project timeline",
  "tone": "professional",
  "length": "medium"
}
```

**Parameters:**
- `subject` (string, required): Email subject line
- `recipient_name` (string, required): Name of email recipient
- `context` (string, required): Context/purpose of the email
- `tone` (string, optional): Email tone - "professional", "friendly", "formal", "casual" (default: "professional")
- `length` (string, optional): Email length - "short", "medium", "long" (default: "medium")

**Response (Success - 200):**
```json
{
  "status": "success",
  "generated_email": "Dear John,\n\nFollowing up on our recent discussion regarding the Q2 project timeline...",
  "critic_feedback": {
    "quality_score": 8.5,
    "tone_match": true,
    "improvements": [
      "Consider adding a specific call-to-action",
      "The closing could be more personalized"
    ],
    "approved": true
  },
  "timestamp": "2026-03-02T10:30:45Z"
}
```

**Response (Error - 400/500):**
```json
{
  "status": "error",
  "error_code": "INVALID_REQUEST",
  "message": "Missing required field: subject",
  "timestamp": "2026-03-02T10:30:45Z"
}
```

## Usage Examples

### Python Client

```python
import requests
import json

API_ENDPOINT = "https://your-api-id.execute-api.us-east-1.amazonaws.com/prod/email/generate"

payload = {
    "subject": "Project Update",
    "recipient_name": "Sarah Smith",
    "context": "Update on the new feature development",
    "tone": "professional",
    "length": "medium"
}

headers = {
    "Content-Type": "application/json"
}

response = requests.post(API_ENDPOINT, json=payload, headers=headers)
result = response.json()

print("Generated Email:")
print(result["generated_email"])
print("\nCritic Feedback:")
print(json.dumps(result["critic_feedback"], indent=2))
```

### cURL

```bash
curl -X POST https://your-api-id.execute-api.us-east-1.amazonaws.com/prod/email/generate \ 
  -H "Content-Type: application/json" \ 
  -d '{
    "subject": "Project Update",
    "recipient_name": "Sarah Smith",
    "context": "Update on the new feature development",
    "tone": "professional",
    "length": "medium"
  }'
```

## Lambda Function Structure

```
lambda_function.py
├── handler()              # API Gateway entry point
├── generate_email()       # GPT-4 email generation
├── critic_check()         # GPT-4o validation
├── validate_request()     # Input validation
├── format_response()      # Response formatting
└── log_execution()        # CloudWatch logging
```

## Configuration & Customization

### Prompts

Edit the system prompts for GPT-4 and GPT-4o in `config/prompts.json`:

```json
{
  "gpt4_email_prompt": "You are a professional email writer...",
  "gpt4o_critic_prompt": "You are an expert email critic..."
}
```

### Lambda Settings

Configure in `serverless.yml` or `template.yaml`:
- **Memory:** 256-512 MB (recommended: 512 MB for optimal performance)
- **Timeout:** 30 seconds
- **Ephemeral Storage:** 512 MB
- **Environment Variables:** Loaded from `.env`

## Cost Estimation

### AWS Costs (per million invocations):
- Lambda: ~$0.20 (512 MB, 10 sec average)
- API Gateway: ~$3.50
- CloudWatch: ~$0.50
- **Total: ~$4.20/million requests**

### OpenAI API Costs (approximate):
- GPT-4: ~$0.03 per 1K tokens
- GPT-4o: ~$0.015 per 1K tokens
- **Average per email: $0.05-$0.10**

## Monitoring & Debugging

### CloudWatch Logs

View logs for a specific Lambda invocation:

```bash
aws logs tail /aws/lambda/email-generator --follow
```

### Metrics

Monitor key metrics in CloudWatch Dashboard:
- Invocation count
- Average duration
- Error rate
- Throttling events

### Error Codes

| Code | Description | Solution |
|------|-------------|----------|
| `INVALID_REQUEST` | Missing or malformed request body | Verify JSON format and required fields |
| `API_KEY_ERROR` | OpenAI API key invalid/missing | Check AWS Secrets Manager configuration |
| `GENERATION_ERROR` | GPT-4 email generation failed | Check OpenAI API status and quota |
| `VALIDATION_ERROR` | GPT-4o critic check failed | Review email content for compliance issues |
| `RATE_LIMIT` | API rate limit exceeded | Implement exponential backoff |

## Best Practices

1. **API Security:**
   - Use API Gateway API Key for rate limiting
   - Implement AWS WAF for DDoS protection
   - Store OpenAI API key in AWS Secrets Manager

2. **Performance:**
   - Cache common prompts in Lambda memory
   - Use connection pooling for API calls
   - Implement async processing for batch emails

3. **Error Handling:**
   - Implement retry logic with exponential backoff
   - Log all errors to CloudWatch
   - Return meaningful error messages to clients

4. **Cost Optimization:**
   - Use Lambda provisioned concurrency for predictable traffic
   - Implement request throttling
   - Monitor and optimize token usage

## Troubleshooting

### Lambda Cold Start

**Issue:** First invocation takes longer than expected

**Solution:** 
- Increase provisioned concurrency
- Use Lambda@Edge for warming up
- Pre-compile dependencies

### API Timeouts

**Issue:** Requests timing out after 30 seconds

**Solution:**
- Reduce GPT model complexity if possible
- Implement async processing
- Increase Lambda timeout to 60 seconds

### OpenAI Rate Limiting

**Issue:** Frequent 429 rate limit errors

**Solution:**
- Implement exponential backoff
- Use OpenAI usage quota monitoring
- Distribute requests across multiple API keys

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Testing

Run tests locally:

```bash
# Unit tests
python -m pytest tests/unit/

# Integration tests
python -m pytest tests/integration/

# Load tests
python -m locust -f tests/load/locustfile.py
```

## Security Considerations

- Store sensitive data (API keys, secrets) in AWS Secrets Manager
- Use IAM roles with least privilege principles
- Enable VPC endpoint for PrivateLink connections
- Implement request signing with SigV4 for enhanced security
- Enable API Gateway request/response logging

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support & Contact

For issues, questions, or suggestions:
- GitHub Issues: [Create an Issue](https://github.com/venkat22/v-csm/issues)
- Email: venkat22@example.com
- Documentation: [Full Docs](https://github.com/venkat22/v-csm/wiki)

## Roadmap

- [ ] Support for multiple languages
- [ ] Custom prompt templates via API
- [ ] Batch email processing
- [ ] Email template library
- [ ] Analytics dashboard
- [ ] A/B testing capabilities
- [ ] Integration with email service providers (SendGrid, SES)

## Changelog

### v1.0.0 (2026-03-02)
- Initial release with GPT-4 generation and GPT-4o critic check
- AWS Lambda and API Gateway integration
- CloudWatch monitoring and logging

---

**Last Updated:** 2026-03-02 17:42:21
**Maintainer:** venkat22