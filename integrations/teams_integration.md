# Integration: Microsoft Teams

## Overview
Integration enabling agents to operate within Microsoft Teams, providing AI capabilities directly in the platform where enterprise teams collaborate.

## Capabilities
- Respond to @mentions and chats
- Proactive messaging and notifications
- Adaptive cards for rich interactions
- Meeting integration and transcription
- Task and workflow automation
- SharePoint and OneDrive integration

## Implementation

### 1. Bot Framework Setup
```python
# Microsoft Teams Bot using Bot Framework
from botbuilder.core import ActivityHandler, TurnContext, MessageFactory
from botbuilder.schema import ChannelAccount, Activity, ActivityTypes

class TeamsAgentBot(ActivityHandler):
    """Teams integration for AI agents"""
    
    async def on_message_activity(self, turn_context: TurnContext):
        """Handle incoming messages"""
        text = turn_context.activity.text.strip()
        
        # Check if bot was mentioned
        if self.is_mentioned(turn_context.activity):
            response = await self.process_mention(text, turn_context)
        else:
            response = await self.process_message(text, turn_context)
        
        # Send response
        await turn_context.send_activity(MessageFactory.text(response))
    
    def is_mentioned(self, activity: Activity) -> bool:
        """Check if bot was mentioned in the message"""
        if activity.entities:
            for entity in activity.entities:
                if entity.type == "mention" and entity.mentioned.id == activity.recipient.id:
                    return True
        return False
    
    async def process_mention(self, text: str, turn_context: TurnContext) -> str:
        """Process bot mention and generate response"""
        # Remove bot mention from text
        clean_text = self.remove_mention(text, turn_context)
        
        # Process using agent capabilities
        return await self.generate_agent_response(clean_text, turn_context)
    
    async def generate_agent_response(self, text: str, turn_context: TurnContext) -> str:
        """Generate response using agent skills"""
        # This would integrate with your agent's capabilities
        # Placeholder response
        return f"I received your request: {text}"
    
    def remove_mention(self, text: str, turn_context: TurnContext) -> str:
        """Remove bot mention from message text"""
        # Implementation to clean up mention tags
        return text
```

### 2. Adaptive Cards for Rich Interactions
```python
# Create adaptive card for interactive responses
def create_adaptive_card(title: str, data: dict) -> dict:
    """
    Create an adaptive card for rich formatting
    
    Args:
        title: Card title
        data: Data to display in card
        
    Returns:
        Adaptive card JSON
    """
    return {
        "type": "AdaptiveCard",
        "body": [
            {
                "type": "TextBlock",
                "size": "Large",
                "weight": "Bolder",
                "text": title
            },
            {
                "type": "FactSet",
                "facts": [
                    {"title": key, "value": str(value)}
                    for key, value in data.items()
                ]
            },
            {
                "type": "ActionSet",
                "actions": [
                    {
                        "type": "Action.OpenUrl",
                        "title": "View Details",
                        "url": "https://example.com/details"
                    },
                    {
                        "type": "Action.Submit",
                        "title": "Refresh",
                        "data": {"action": "refresh"}
                    }
                ]
            }
        ],
        "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
        "version": "1.4"
    }

# Send adaptive card
async def send_adaptive_card(turn_context: TurnContext, card_data: dict):
    """Send an adaptive card to Teams"""
    card_attachment = {
        "contentType": "application/vnd.microsoft.card.adaptive",
        "content": card_data
    }
    
    message = Activity(
        type=ActivityTypes.message,
        attachments=[card_attachment]
    )
    
    await turn_context.send_activity(message)
```

### 3. Proactive Messaging
```python
# Send proactive messages to Teams channels
from botbuilder.core import BotFrameworkAdapter, ConversationReference

async def send_proactive_message(
    adapter: BotFrameworkAdapter,
    conversation_ref: ConversationReference,
    message: str
):
    """
    Send proactive message to Teams channel or user
    
    Args:
        adapter: Bot framework adapter
        conversation_ref: Reference to conversation
        message: Message to send
    """
    
    async def callback(turn_context: TurnContext):
        await turn_context.send_activity(message)
    
    await adapter.continue_conversation(
        conversation_ref,
        callback,
        app_id="your-app-id"
    )

# Example: Send alert
async def send_alert_to_teams(
    adapter: BotFrameworkAdapter,
    channel_ref: ConversationReference,
    alert_data: dict
):
    """Send formatted alert to Teams channel"""
    card = create_adaptive_card(
        title=f"🚨 {alert_data['title']}",
        data={
            "Severity": alert_data['severity'],
            "Time": alert_data['timestamp'],
            "Description": alert_data['description']
        }
    )
    
    async def callback(turn_context: TurnContext):
        await send_adaptive_card(turn_context, card)
    
    await adapter.continue_conversation(channel_ref, callback)
```

## Configuration

### Bot Registration (Azure)
```json
{
  "name": "BusinessAgentBot",
  "description": "AI-powered business automation agent",
  "messaging_endpoint": "https://your-bot.azurewebsites.net/api/messages",
  "app_id": "your-app-id",
  "app_password": "your-app-password",
  "channels": ["msteams"],
  "scopes": ["team", "personal", "groupchat"]
}
```

### Teams App Manifest
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/teams/v1.16/MicrosoftTeams.schema.json",
  "manifestVersion": "1.16",
  "version": "1.0.0",
  "id": "your-app-id",
  "packageName": "com.yourcompany.businessagent",
  "developer": {
    "name": "Your Company",
    "websiteUrl": "https://yourcompany.com",
    "privacyUrl": "https://yourcompany.com/privacy",
    "termsOfUseUrl": "https://yourcompany.com/terms"
  },
  "name": {
    "short": "Business Agent",
    "full": "AI Business Automation Agent"
  },
  "description": {
    "short": "AI-powered business automation",
    "full": "Comprehensive AI agent for business process automation, data analysis, and decision support"
  },
  "icons": {
    "outline": "outline.png",
    "color": "color.png"
  },
  "accentColor": "#0078D4",
  "bots": [
    {
      "botId": "your-bot-id",
      "scopes": ["personal", "team", "groupchat"],
      "supportsFiles": true,
      "isNotificationOnly": false,
      "commandLists": [
        {
          "scopes": ["personal", "team", "groupchat"],
          "commands": [
            {
              "title": "analyze",
              "description": "Analyze business data"
            },
            {
              "title": "report",
              "description": "Generate reports"
            },
            {
              "title": "help",
              "description": "Show available commands"
            }
          ]
        }
      ]
    }
  ],
  "permissions": ["identity", "messageTeamMembers"],
  "validDomains": ["yourcompany.com"]
}
```

## Use Cases

### 1. Meeting Assistant
```
During Meeting:
User: @agent summarize key points
Agent: [Adaptive Card]
       📋 Meeting Summary
       • Project timeline approved: Q1 2026 launch
       • Budget: $500K allocated
       • Action items: 3 assigned
       • Next meeting: Dec 15
       [View Full Notes] [Export to OneNote]
```

### 2. Data Dashboard
```
User: @agent show sales dashboard
Agent: [Adaptive Card with charts]
       📊 Sales Dashboard - October 2025
       Revenue: $1.2M (+15% MoM)
       New Customers: 145 (+22%)
       Top Product: Widget Pro
       [Refresh] [View Detailed Report]
```

### 3. Workflow Automation
```
Agent: ✅ Approval Request
       Invoice #INV-2025-10-450
       Vendor: Acme Corp
       Amount: $12,500
       Category: Office Supplies
       [Approve] [Reject] [View Details]
```

## Best Practices
- Use adaptive cards for rich, interactive experiences
- Leverage Teams activity feed for important notifications
- Integrate with Microsoft Graph for enhanced capabilities
- Support both 1:1 chats and channel conversations
- Provide clear command documentation
- Handle message edits and deletions appropriately

## Advanced Features

### Integration with Microsoft Graph
```python
# Access user calendar, email, files, etc.
from msgraph.core import GraphClient

def get_user_calendar(user_id: str, access_token: str):
    """Get user's calendar events using Microsoft Graph"""
    client = GraphClient(credential=access_token)
    
    events = client.get(f'/users/{user_id}/calendar/events')
    return events
```

### Meeting Transcription Analysis
```python
async def analyze_meeting_transcript(transcript: str) -> dict:
    """Analyze meeting transcript and extract insights"""
    return {
        "summary": "Meeting covered project timeline and budget",
        "action_items": [
            {"task": "Update project plan", "assignee": "John"},
            {"task": "Prepare budget report", "assignee": "Sarah"}
        ],
        "decisions": ["Approved Q1 2026 launch date"],
        "key_topics": ["Timeline", "Budget", "Resources"]
    }
```

## Security Considerations
- Use Azure Active Directory for authentication
- Implement role-based access control
- Encrypt data in transit and at rest
- Comply with Microsoft Teams app security requirements
- Regular security audits and updates
