# Integration: Slack

## Overview
Integration enabling agents to operate within Slack workspaces, where teams already collaborate. Provides seamless agent interaction without requiring users to learn new tools.

## Capabilities
- Respond to mentions and direct messages
- Monitor specific channels for keywords
- Post automated updates and alerts
- Interactive buttons and forms
- Thread-based conversations
- File sharing and analysis

## Implementation

### 1. Bot Setup
```python
# Slack Bot Configuration
import os
from slack_sdk import WebClient
from slack_sdk.socket_mode import SocketModeClient
from slack_sdk.socket_mode.request import SocketModeRequest
from slack_sdk.socket_mode.response import SocketModeResponse

class AgentSlackBot:
    """Slack integration for AI agents"""
    
    def __init__(self, bot_token: str, app_token: str):
        self.client = WebClient(token=bot_token)
        self.socket_client = SocketModeClient(
            app_token=app_token,
            web_client=self.client
        )
        
    def start(self):
        """Start listening for Slack events"""
        self.socket_client.socket_mode_request_listeners.append(self.handle_events)
        self.socket_client.connect()
        
    def handle_events(self, client: SocketModeClient, req: SocketModeRequest):
        """Process incoming Slack events"""
        if req.type == "events_api":
            event = req.payload["event"]
            
            # Handle app mentions
            if event["type"] == "app_mention":
                self.handle_mention(event)
            
            # Handle direct messages
            elif event["type"] == "message":
                self.handle_message(event)
        
        # Acknowledge the request
        response = SocketModeResponse(envelope_id=req.envelope_id)
        client.send_socket_mode_response(response)
    
    def handle_mention(self, event):
        """Respond to @mentions"""
        channel = event["channel"]
        text = event["text"]
        thread_ts = event.get("thread_ts", event["ts"])
        
        # Process user request using agent capabilities
        response = self.process_request(text)
        
        # Post response in thread
        self.client.chat_postMessage(
            channel=channel,
            text=response,
            thread_ts=thread_ts
        )
    
    def handle_message(self, event):
        """Handle direct messages"""
        # Similar to handle_mention but for DMs
        pass
    
    def process_request(self, text: str) -> str:
        """Process user request using agent skills"""
        # This would integrate with your agent's NLP and other skills
        # Placeholder response
        return f"I received your request: {text}"
```

### 2. Interactive Commands
```python
# Slash command handler
def handle_slash_command(command: str, text: str, user_id: str) -> dict:
    """
    Handle Slack slash commands
    
    Examples:
        /agent-analyze data sales last-month
        /agent-report customer-satisfaction
        /agent-help
    """
    
    if command == "/agent-help":
        return {
            "response_type": "ephemeral",
            "text": "Available commands:\n" +
                   "• /agent-analyze <topic> - Analyze data\n" +
                   "• /agent-report <type> - Generate report\n" +
                   "• /agent-help - Show this help"
        }
    
    elif command == "/agent-analyze":
        # Process analysis request
        result = perform_analysis(text)
        return {
            "response_type": "in_channel",
            "text": f"Analysis complete: {result}"
        }
    
    return {
        "response_type": "ephemeral",
        "text": "Unknown command. Use /agent-help for available commands."
    }
```

### 3. Automated Alerts
```python
# Send proactive alerts to Slack
def send_alert(channel: str, alert_data: dict):
    """
    Send formatted alert to Slack channel
    
    Args:
        channel: Slack channel ID or name
        alert_data: Dictionary containing alert information
    """
    blocks = [
        {
            "type": "header",
            "text": {
                "type": "plain_text",
                "text": f"🚨 Alert: {alert_data['title']}"
            }
        },
        {
            "type": "section",
            "fields": [
                {
                    "type": "mrkdwn",
                    "text": f"*Severity:*\n{alert_data['severity']}"
                },
                {
                    "type": "mrkdwn",
                    "text": f"*Time:*\n{alert_data['timestamp']}"
                }
            ]
        },
        {
            "type": "section",
            "text": {
                "type": "mrkdwn",
                "text": alert_data['description']
            }
        },
        {
            "type": "actions",
            "elements": [
                {
                    "type": "button",
                    "text": {"type": "plain_text", "text": "View Details"},
                    "action_id": "view_details",
                    "url": alert_data.get('details_url')
                },
                {
                    "type": "button",
                    "text": {"type": "plain_text", "text": "Acknowledge"},
                    "action_id": "acknowledge_alert",
                    "style": "primary"
                }
            ]
        }
    ]
    
    client.chat_postMessage(
        channel=channel,
        blocks=blocks
    )
```

## Configuration

### Slack App Manifest
```yaml
display_information:
  name: Business Agent
  description: AI-powered business automation agent
  background_color: "#2c2d30"

features:
  bot_user:
    display_name: Business Agent
    always_online: true
  
  slash_commands:
    - command: /agent-analyze
      description: Analyze business data
      usage_hint: "[topic] [parameters]"
    
    - command: /agent-report
      description: Generate reports
      usage_hint: "[report-type]"
    
    - command: /agent-help
      description: Show available commands

oauth_config:
  scopes:
    bot:
      - app_mentions:read
      - chat:write
      - channels:history
      - channels:read
      - files:write
      - im:history
      - im:write

settings:
  event_subscriptions:
    bot_events:
      - app_mention
      - message.im
      - file_shared
  
  interactivity:
    is_enabled: true
  
  org_deploy_enabled: false
  socket_mode_enabled: true
```

## Use Cases

### 1. Customer Support Agent
```
User: @agent What's the status of order #12345?
Agent: Order #12345 is out for delivery. Expected arrival: Today by 5 PM.
       Tracking: https://tracking.example.com/12345
```

### 2. Data Analysis Agent
```
User: /agent-analyze sales last-quarter
Agent: Q3 2025 Sales Analysis:
       📈 Total Revenue: $1.2M (+15% vs Q2)
       🏆 Top Product: Widget Pro ($450K)
       🌍 Top Region: EMEA (40% of total)
       [View detailed report]
```

### 3. Automated Alerts
```
Agent: 🚨 Alert: High Customer Churn Detected
       Severity: High
       Current Rate: 8.2% (Threshold: 5%)
       Affected Segment: Enterprise customers
       [View Details] [Acknowledge]
```

## Best Practices
- Use threads to keep conversations organized
- Provide clear, actionable responses
- Use interactive elements for complex workflows
- Respect user privacy and data security
- Implement rate limiting to avoid API limits
- Monitor and log all interactions

## Security Considerations
- Store credentials securely (use environment variables or secrets manager)
- Validate all user inputs
- Implement access controls (who can use which commands)
- Audit trail for all agent actions
- Encrypt sensitive data in transit and at rest
