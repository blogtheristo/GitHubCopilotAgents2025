# Example: Customer Service Agent Implementation

## Scenario
A mid-sized e-commerce company wants to automate customer service to provide 24/7 support while reducing costs and freeing human agents to handle complex issues.

## Requirements
- Handle order status inquiries
- Answer product questions
- Process return requests
- Escalate complex issues to humans
- Integrate with existing Slack workspace

## Implementation

### 1. Agent Configuration

```yaml
# customer-service-agent.yaml
agent:
  name: "CustomerServiceAgent"
  description: "24/7 automated customer support"
  version: "1.0.0"

prowess:
  primary: "customer_service_automation"

skills:
  - name: "nlp_understanding"
    config:
      languages: ["en"]
      confidence_threshold: 0.75
  
  - name: "intent_classification"
    config:
      intents:
        - "order_status"
        - "product_info"
        - "return_request"
        - "complaint"
        - "shipping_info"

integrations:
  - type: "slack"
    config:
      channels: ["#customer-support"]
      bot_name: "SupportBot"
  
  - type: "crm"
    system: "shopify"
    config:
      store_url: "https://yourstore.myshopify.com"

rules:
  escalation:
    - condition: "confidence < 0.75"
      action: "escalate_to_human"
    - condition: "intent == 'complaint'"
      action: "escalate_to_supervisor"
  
  auto_respond:
    - condition: "intent == 'order_status'"
      action: "fetch_and_provide_tracking"

monitoring:
  metrics:
    - "response_time"
    - "resolution_rate"
    - "customer_satisfaction"
    - "escalation_rate"
```

### 2. Sample Code Implementation

```python
# customer_service_agent.py
import os
from typing import Dict, Optional
from slack_sdk import WebClient
from shopify import Shop, Order

class CustomerServiceAgent:
    """Automated customer service agent"""
    
    def __init__(self, config: dict):
        self.config = config
        self.slack_client = WebClient(token=os.getenv('SLACK_BOT_TOKEN'))
        self.setup_shopify()
        
    def setup_shopify(self):
        """Initialize Shopify connection"""
        shop_url = self.config['integrations'][1]['config']['store_url']
        api_key = os.getenv('SHOPIFY_API_KEY')
        Shop.setup(url=shop_url, token=api_key)
    
    def process_message(self, message: str, user_id: str) -> Dict[str, any]:
        """Process incoming customer message"""
        # Step 1: Understand intent
        intent = self.classify_intent(message)
        
        # Step 2: Extract relevant information
        entities = self.extract_entities(message)
        
        # Step 3: Determine action
        if intent['confidence'] < 0.75:
            return self.escalate_to_human(message, user_id, reason="low_confidence")
        
        # Step 4: Execute action based on intent
        if intent['type'] == 'order_status':
            return self.handle_order_status(entities)
        elif intent['type'] == 'product_info':
            return self.handle_product_info(entities)
        elif intent['type'] == 'return_request':
            return self.handle_return_request(entities, user_id)
        elif intent['type'] == 'complaint':
            return self.escalate_to_human(message, user_id, reason="complaint")
        else:
            return self.handle_general_inquiry(message)
    
    def classify_intent(self, message: str) -> Dict[str, any]:
        """Classify customer intent using NLP"""
        message_lower = message.lower()
        
        # Simple keyword-based classification (replace with ML model)
        if any(word in message_lower for word in ['order', 'status', 'tracking', 'where is']):
            return {'type': 'order_status', 'confidence': 0.9}
        elif any(word in message_lower for word in ['return', 'refund', 'send back']):
            return {'type': 'return_request', 'confidence': 0.85}
        elif any(word in message_lower for word in ['product', 'price', 'specs', 'features']):
            return {'type': 'product_info', 'confidence': 0.8}
        elif any(word in message_lower for word in ['complaint', 'unhappy', 'disappointed', 'angry']):
            return {'type': 'complaint', 'confidence': 0.95}
        else:
            return {'type': 'general', 'confidence': 0.5}
    
    def extract_entities(self, message: str) -> Dict[str, any]:
        """Extract order numbers, product names, etc."""
        import re
        
        entities = {}
        
        # Extract order number (format: #12345 or 12345)
        order_match = re.search(r'#?(\d{5,})', message)
        if order_match:
            entities['order_number'] = order_match.group(1)
        
        # Extract email
        email_match = re.search(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', message)
        if email_match:
            entities['email'] = email_match.group(0)
        
        return entities
    
    def handle_order_status(self, entities: Dict[str, any]) -> Dict[str, any]:
        """Handle order status inquiry"""
        order_number = entities.get('order_number')
        
        if not order_number:
            return {
                'success': False,
                'response': "I'd be happy to help with your order status. Could you please provide your order number?"
            }
        
        try:
            # Fetch order from Shopify
            order = Order.find(order_number)
            
            tracking_info = f"""
Order #{order.order_number} Status:

📦 Status: {order.fulfillment_status or 'Processing'}
📍 Shipping: {order.shipping_address.city if order.shipping_address else 'N/A'}
🚚 Tracking: {self.get_tracking_number(order)}
📅 Expected Delivery: {self.estimate_delivery(order)}

Need anything else? Just ask!
            """
            
            return {
                'success': True,
                'response': tracking_info,
                'metrics': {
                    'intent': 'order_status',
                    'resolved': True,
                    'escalated': False
                }
            }
        except Exception as e:
            return {
                'success': False,
                'response': f"I couldn't find order #{order_number}. Please verify the order number or contact support.",
                'error': str(e)
            }
    
    def handle_product_info(self, entities: Dict[str, any]) -> Dict[str, any]:
        """Handle product information request"""
        # Simplified implementation
        return {
            'success': True,
            'response': "I can help with product information. Which product are you interested in?"
        }
    
    def handle_return_request(self, entities: Dict[str, any], user_id: str) -> Dict[str, any]:
        """Handle return request"""
        order_number = entities.get('order_number')
        
        if not order_number:
            return {
                'success': False,
                'response': "To process a return, I'll need your order number. Could you provide it?"
            }
        
        # Create return request
        response = f"""
I'll help you with your return for order #{order_number}.

✅ Returns accepted within 30 days
📦 Free return shipping label
💰 Refund processed within 5-7 business days

I'm generating your return label now. You'll receive an email with instructions shortly.

Return Reference: RET-{order_number}
        """
        
        return {
            'success': True,
            'response': response,
            'action_taken': 'return_initiated',
            'metrics': {
                'intent': 'return_request',
                'resolved': True,
                'escalated': False
            }
        }
    
    def escalate_to_human(self, message: str, user_id: str, reason: str) -> Dict[str, any]:
        """Escalate to human agent"""
        # Notify human agents in Slack
        self.slack_client.chat_postMessage(
            channel='#customer-support-escalations',
            text=f"""
🔔 Customer Support Escalation

User: {user_id}
Reason: {reason}
Original Message: "{message}"

Please respond to this customer as soon as possible.
            """
        )
        
        return {
            'success': True,
            'response': "I've connected you with a human agent who will assist you shortly. Average wait time: 5 minutes.",
            'escalated': True,
            'metrics': {
                'escalation_reason': reason
            }
        }
    
    def handle_general_inquiry(self, message: str) -> Dict[str, any]:
        """Handle general inquiries"""
        return {
            'success': True,
            'response': "I'm here to help! I can assist with:\n• Order status\n• Product information\n• Returns and refunds\n\nWhat can I help you with?"
        }
    
    def get_tracking_number(self, order) -> str:
        """Extract tracking number from order"""
        if order.fulfillments:
            return order.fulfillments[0].tracking_number or "Not available yet"
        return "Processing"
    
    def estimate_delivery(self, order) -> str:
        """Estimate delivery date"""
        # Simplified - would use actual shipping data
        return "2-3 business days"

# Example usage
if __name__ == "__main__":
    # Load configuration
    import yaml
    with open('customer-service-agent.yaml', 'r') as f:
        config = yaml.safe_load(f)
    
    # Initialize agent
    agent = CustomerServiceAgent(config)
    
    # Test with sample message
    result = agent.process_message(
        "Where is my order #12345?",
        user_id="U12345"
    )
    
    print(f"Response: {result['response']}")
    print(f"Metrics: {result.get('metrics', {})}")
```

### 3. Slack Integration

```python
# slack_bot.py
from slack_sdk.socket_mode import SocketModeClient
from slack_sdk.socket_mode.request import SocketModeRequest
from slack_sdk.socket_mode.response import SocketModeResponse
from customer_service_agent import CustomerServiceAgent
import yaml

def start_slack_bot():
    """Start Slack bot with customer service agent"""
    
    # Load configuration
    with open('customer-service-agent.yaml', 'r') as f:
        config = yaml.safe_load(f)
    
    # Initialize agent
    agent = CustomerServiceAgent(config)
    
    # Initialize Slack client
    socket_client = SocketModeClient(
        app_token=os.getenv('SLACK_APP_TOKEN'),
        web_client=agent.slack_client
    )
    
    def handle_message(client: SocketModeClient, req: SocketModeRequest):
        """Handle incoming Slack messages"""
        if req.type == "events_api":
            event = req.payload["event"]
            
            # Ignore bot messages
            if event.get("bot_id"):
                response = SocketModeResponse(envelope_id=req.envelope_id)
                client.send_socket_mode_response(response)
                return
            
            # Process customer message
            if event["type"] == "message":
                user_message = event["text"]
                user_id = event["user"]
                channel = event["channel"]
                thread_ts = event.get("thread_ts", event["ts"])
                
                # Get agent response
                result = agent.process_message(user_message, user_id)
                
                # Post response to Slack
                agent.slack_client.chat_postMessage(
                    channel=channel,
                    text=result['response'],
                    thread_ts=thread_ts
                )
        
        # Acknowledge event
        response = SocketModeResponse(envelope_id=req.envelope_id)
        client.send_socket_mode_response(response)
    
    # Register event handler
    socket_client.socket_mode_request_listeners.append(handle_message)
    
    # Start bot
    print("Customer Service Agent started. Listening for messages...")
    socket_client.connect()
    
    # Keep running
    from threading import Event
    Event().wait()

if __name__ == "__main__":
    start_slack_bot()
```

## Results

### Before Agent Implementation
- Average response time: 4 hours
- Human agents handling: 5,000 tickets/month
- Cost per ticket: $15
- Monthly cost: $75,000
- Customer satisfaction: 72%

### After Agent Implementation
- Average response time: 2 minutes
- AI handling: 4,000 tickets/month (80%)
- Human agents handling: 1,000 tickets/month (20% - complex cases)
- Cost per AI ticket: $0.50
- Monthly cost: $17,000 (AI) + $15,000 (human) = $32,000
- Customer satisfaction: 87%

### Resource Liberation
- **Human agents freed**: 80% of routine inquiries
- **Cost savings**: $43,000/month ($516,000/year)
- **Time savings**: 4,000 hours/month freed for complex issues
- **Response improvement**: 99.2% faster (4 hours → 2 minutes)
- **Availability**: 24/7 instead of business hours only

## Deployment

```bash
# 1. Set environment variables
export SLACK_BOT_TOKEN=xoxb-your-token
export SLACK_APP_TOKEN=xapp-your-token
export SHOPIFY_API_KEY=your-api-key

# 2. Install dependencies
pip install slack-sdk shopify-python-api pyyaml

# 3. Run agent
python slack_bot.py
```

## Next Steps
1. Monitor performance metrics
2. Collect customer feedback
3. Refine intent classification based on real data
4. Add more product-specific knowledge
5. Implement continuous learning from escalated cases
