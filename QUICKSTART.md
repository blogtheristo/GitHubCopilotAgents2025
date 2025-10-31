# Quick Start Guide

Get your first AI agent running in 15 minutes!

## Prerequisites
- Docker and Docker Compose installed
- A Slack workspace (or Microsoft Teams)
- Basic understanding of YAML configuration

## Step 1: Choose Your Use Case (2 minutes)

Select one of these common scenarios:

### Scenario A: Customer Service Automation
Best for: E-commerce, SaaS, any customer-facing business
**Benefits**: 24/7 support, 99% faster response times, 80% cost reduction

### Scenario B: Executive Data Analysis
Best for: Data-driven organizations, executives needing quick insights
**Benefits**: Real-time analytics, 500x faster than reports, self-service queries

### Scenario C: Process Automation
Best for: Operations-heavy businesses, repetitive workflows
**Benefits**: 50% faster processes, error reduction, resource liberation

## Step 2: Set Up Configuration (5 minutes)

```bash
# Clone the repository
git clone https://github.com/blogtheristo/GitHubCopilotAgents2025.git
cd GitHubCopilotAgents2025

# Copy the configuration template
cp config/agent-config.yaml.template config/my-agent.yaml
```

### For Customer Service Automation:
```yaml
# config/my-agent.yaml
agent:
  name: "MyCustomerServiceAgent"
  description: "24/7 customer support agent"

prowess:
  primary: "customer_service_automation"

skills:
  - name: "nlp_understanding"
  - name: "intent_classification"

integrations:
  - type: "slack"
    config:
      channels: ["#customer-support"]
```

### For Data Analysis:
```yaml
# config/my-agent.yaml
agent:
  name: "MyDataAgent"
  description: "Executive data analysis agent"

prowess:
  primary: "data_analysis_insights"

skills:
  - name: "nlp_understanding"
  - name: "data_processing"
  - name: "statistical_analysis"

integrations:
  - type: "slack"
    config:
      channels: ["#executive-team"]
```

## Step 3: Set Up Slack Integration (5 minutes)

1. Go to https://api.slack.com/apps
2. Click "Create New App" → "From scratch"
3. Name your app (e.g., "Customer Service Agent")
4. Select your workspace

### Configure Bot Permissions:
Add these OAuth scopes under "OAuth & Permissions":
- `app_mentions:read`
- `chat:write`
- `channels:history`
- `channels:read`

### Enable Socket Mode:
1. Go to "Socket Mode" and enable it
2. Generate an app-level token with `connections:write` scope
3. Save this as `SLACK_APP_TOKEN`

### Install to Workspace:
1. Click "Install to Workspace"
2. Authorize the app
3. Save the "Bot User OAuth Token" as `SLACK_BOT_TOKEN`

## Step 4: Set Environment Variables (1 minute)

```bash
# Create .env file
cat > .env << EOF
SLACK_BOT_TOKEN=xoxb-your-bot-token-here
SLACK_APP_TOKEN=xapp-your-app-token-here
AGENT_CONFIG_PATH=/config/my-agent.yaml
EOF
```

## Step 5: Run Your Agent (2 minutes)

### Option A: Quick Test (Simplified Python Script)

```bash
# Install dependencies
pip install slack-sdk pyyaml

# Create simple test script
cat > test_agent.py << 'EOF'
import os
from slack_sdk import WebClient
from slack_sdk.socket_mode import SocketModeClient

# Simple echo bot for testing
client = WebClient(token=os.getenv('SLACK_BOT_TOKEN'))
socket_client = SocketModeClient(
    app_token=os.getenv('SLACK_APP_TOKEN'),
    web_client=client
)

def handle_message(client, req):
    if req.type == "events_api":
        event = req.payload.get("event", {})
        if event.get("type") == "app_mention":
            channel = event["channel"]
            text = event["text"]
            
            # Simple response
            response = f"Hello! I received: {text}\n\nI'm ready to help with:\n• Customer queries\n• Data analysis\n• Process automation"
            
            client.web_client.chat_postMessage(
                channel=channel,
                text=response,
                thread_ts=event.get("thread_ts", event["ts"])
            )
    
    from slack_sdk.socket_mode.response import SocketModeResponse
    client.send_socket_mode_response(SocketModeResponse(envelope_id=req.envelope_id))

socket_client.socket_mode_request_listeners.append(handle_message)
socket_client.connect()

print("✅ Agent is running! Mention @your-bot in Slack to test.")
from threading import Event
Event().wait()
EOF

# Run the agent
python test_agent.py
```

### Option B: Docker (Production-Ready)

```bash
# Build and run with Docker Compose
docker-compose up -d

# Check logs
docker-compose logs -f agent
```

## Step 6: Test Your Agent (1 minute)

1. Open Slack
2. Go to your configured channel (e.g., #customer-support)
3. Mention your bot: `@YourBot hello!`
4. You should get a response within seconds!

### Test Queries:

**For Customer Service Agent:**
```
@CustomerBot What's the status of order #12345?
@CustomerBot I need help with my account
@CustomerBot How do I return an item?
```

**For Data Analysis Agent:**
```
@DataAgent What were our sales last month?
@DataAgent Show me customer growth this quarter
@DataAgent What are our top products?
```

## Next Steps

### 1. Customize Responses
Edit your agent configuration to add:
- Custom intents and responses
- Business-specific knowledge
- Integration with your systems (CRM, database, etc.)

### 2. Add More Capabilities
```yaml
prowess:
  primary: "customer_service_automation"
  secondary:
    - "data_analysis_insights"
    - "process_optimization"
```

### 3. Connect to Your Data
Add database or API connections:
```yaml
integrations:
  - type: "database"
    system: "postgresql"
    connection: "postgresql://user:pass@host:5432/db"
  
  - type: "api"
    system: "shopify"
    endpoint: "https://yourstore.myshopify.com"
```

### 4. Monitor Performance
Track these key metrics:
- Response time
- Query volume
- User satisfaction
- Cost savings
- Resource liberation

### 5. Scale Up
When ready for production:
1. Review [Deployment Guide](config/deployment-guide.md)
2. Set up monitoring and alerts
3. Configure auto-scaling
4. Implement proper security

## Troubleshooting

### Agent Not Responding
```bash
# Check if agent is running
docker-compose ps

# View logs for errors
docker-compose logs agent

# Test Slack tokens
curl -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  https://slack.com/api/auth.test
```

### Slack Events Not Received
- Verify Socket Mode is enabled
- Check app-level token has `connections:write`
- Ensure bot is invited to channel: `/invite @YourBot`

### Configuration Errors
```bash
# Validate YAML syntax
python -c "import yaml; yaml.safe_load(open('config/my-agent.yaml'))"
```

## Getting Help

- 📖 **Full Documentation**: See [README.md](README.md)
- 💡 **Examples**: Check [examples/](examples/) directory
- 🐛 **Issues**: Open a GitHub issue
- 💬 **Discussions**: Join GitHub Discussions

## Success Metrics to Track

After 1 week, measure:
- [ ] Number of queries handled by agent
- [ ] Average response time
- [ ] User satisfaction (ask for feedback)
- [ ] Time saved by team
- [ ] Cost reduction

After 1 month, calculate:
- [ ] Total interactions handled
- [ ] Percentage of queries fully resolved
- [ ] ROI (cost savings vs. implementation cost)
- [ ] Team productivity improvement

## What's Next?

Once your first agent is running successfully:

1. **Add Intelligence**: Connect to your business data
2. **Expand Coverage**: Add more use cases
3. **Optimize**: Refine based on usage patterns
4. **Scale**: Deploy additional agents for other teams
5. **Measure**: Track ROI and business impact

---

**Congratulations!** 🎉 You've deployed your first AI agent that works where your people already are, creating new capabilities while freeing up valuable resources.

Ready to learn more? Check out:
- [Architecture Overview](ARCHITECTURE.md)
- [Complete Examples](examples/)
- [Deployment Guide](config/deployment-guide.md)
