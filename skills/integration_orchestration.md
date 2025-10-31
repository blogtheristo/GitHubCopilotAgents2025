# Skill: Integration & Orchestration

## Overview
Core skill enabling agents to connect, coordinate, and orchestrate actions across multiple systems and platforms seamlessly.

## Capabilities
- Multi-system integration
- Workflow orchestration
- Event-driven automation
- API management
- Data synchronization
- Error handling and retry logic

## Technical Components

### 1. Integration Framework
```python
# integration_manager.py
from typing import Dict, List, Any, Optional
import requests
from abc import ABC, abstractmethod

class IntegrationBase(ABC):
    """Base class for all integrations"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.setup()
    
    @abstractmethod
    def setup(self):
        """Initialize integration"""
        pass
    
    @abstractmethod
    def test_connection(self) -> bool:
        """Test if integration is working"""
        pass
    
    @abstractmethod
    def execute(self, action: str, params: Dict[str, Any]) -> Any:
        """Execute an action on the integrated system"""
        pass

class CRMIntegration(IntegrationBase):
    """CRM system integration (Salesforce, HubSpot, etc.)"""
    
    def setup(self):
        self.api_url = self.config['api_url']
        self.api_key = self.config['api_key']
        self.headers = {'Authorization': f'Bearer {self.api_key}'}
    
    def test_connection(self) -> bool:
        try:
            response = requests.get(f"{self.api_url}/health", headers=self.headers)
            return response.status_code == 200
        except Exception as e:
            print(f"Connection test failed: {e}")
            return False
    
    def execute(self, action: str, params: Dict[str, Any]) -> Any:
        """Execute CRM action"""
        if action == "get_customer":
            return self.get_customer(params['customer_id'])
        elif action == "update_customer":
            return self.update_customer(params['customer_id'], params['data'])
        elif action == "create_ticket":
            return self.create_ticket(params)
        else:
            raise ValueError(f"Unknown action: {action}")
    
    def get_customer(self, customer_id: str) -> Dict[str, Any]:
        response = requests.get(
            f"{self.api_url}/customers/{customer_id}",
            headers=self.headers
        )
        return response.json()
    
    def update_customer(self, customer_id: str, data: Dict[str, Any]) -> Dict[str, Any]:
        response = requests.put(
            f"{self.api_url}/customers/{customer_id}",
            headers=self.headers,
            json=data
        )
        return response.json()
    
    def create_ticket(self, params: Dict[str, Any]) -> Dict[str, Any]:
        response = requests.post(
            f"{self.api_url}/tickets",
            headers=self.headers,
            json=params
        )
        return response.json()

class IntegrationManager:
    """Manages all system integrations"""
    
    def __init__(self):
        self.integrations: Dict[str, IntegrationBase] = {}
    
    def register_integration(self, name: str, integration: IntegrationBase):
        """Register a new integration"""
        self.integrations[name] = integration
    
    def get_integration(self, name: str) -> Optional[IntegrationBase]:
        """Get an integration by name"""
        return self.integrations.get(name)
    
    def test_all_integrations(self) -> Dict[str, bool]:
        """Test all registered integrations"""
        results = {}
        for name, integration in self.integrations.items():
            results[name] = integration.test_connection()
        return results
```

### 2. Workflow Orchestration
```python
# workflow_orchestrator.py
from typing import List, Dict, Any, Callable
from dataclasses import dataclass
from enum import Enum

class StepStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    COMPLETED = "completed"
    FAILED = "failed"
    SKIPPED = "skipped"

@dataclass
class WorkflowStep:
    """Represents a single step in a workflow"""
    name: str
    action: Callable
    params: Dict[str, Any]
    retry_count: int = 3
    timeout: int = 60
    on_failure: str = "stop"  # "stop", "continue", or "retry"
    dependencies: List[str] = None
    
    def __post_init__(self):
        if self.dependencies is None:
            self.dependencies = []

class WorkflowOrchestrator:
    """Orchestrates multi-step workflows across integrations"""
    
    def __init__(self, integration_manager: IntegrationManager):
        self.integration_manager = integration_manager
        self.workflows: Dict[str, List[WorkflowStep]] = {}
    
    def define_workflow(self, name: str, steps: List[WorkflowStep]):
        """Define a new workflow"""
        self.workflows[name] = steps
    
    def execute_workflow(self, name: str, context: Dict[str, Any] = None) -> Dict[str, Any]:
        """Execute a workflow"""
        if name not in self.workflows:
            raise ValueError(f"Workflow '{name}' not found")
        
        steps = self.workflows[name]
        context = context or {}
        results = {}
        
        for step in steps:
            # Check dependencies
            if not self._check_dependencies(step, results):
                print(f"Skipping step '{step.name}' - dependencies not met")
                results[step.name] = {'status': StepStatus.SKIPPED}
                continue
            
            # Execute step
            print(f"Executing step: {step.name}")
            try:
                result = self._execute_step(step, context, results)
                results[step.name] = {
                    'status': StepStatus.COMPLETED,
                    'result': result
                }
            except Exception as e:
                print(f"Step '{step.name}' failed: {e}")
                results[step.name] = {
                    'status': StepStatus.FAILED,
                    'error': str(e)
                }
                
                if step.on_failure == "stop":
                    break
                elif step.on_failure == "retry":
                    # Implement retry logic
                    pass
        
        return results
    
    def _check_dependencies(self, step: WorkflowStep, results: Dict[str, Any]) -> bool:
        """Check if step dependencies are met"""
        for dep in step.dependencies:
            if dep not in results:
                return False
            if results[dep]['status'] != StepStatus.COMPLETED:
                return False
        return True
    
    def _execute_step(self, step: WorkflowStep, context: Dict[str, Any], 
                     previous_results: Dict[str, Any]) -> Any:
        """Execute a single workflow step"""
        # Merge context with previous results
        params = {**context, **step.params}
        
        # Add results from previous steps
        params['previous_results'] = previous_results
        
        # Execute the action
        return step.action(**params)

# Example workflow definition
def create_customer_onboarding_workflow(orchestrator: WorkflowOrchestrator):
    """Define a customer onboarding workflow"""
    
    def create_account(customer_data, **kwargs):
        crm = orchestrator.integration_manager.get_integration('crm')
        return crm.execute('create_customer', {'data': customer_data})
    
    def send_welcome_email(previous_results, **kwargs):
        customer = previous_results['create_account']['result']
        email_service = orchestrator.integration_manager.get_integration('email')
        return email_service.execute('send_email', {
            'to': customer['email'],
            'template': 'welcome',
            'data': customer
        })
    
    def setup_tools_access(previous_results, **kwargs):
        customer = previous_results['create_account']['result']
        tools = orchestrator.integration_manager.get_integration('slack')
        return tools.execute('invite_user', {
            'email': customer['email'],
            'channels': ['#general', '#support']
        })
    
    steps = [
        WorkflowStep(
            name='create_account',
            action=create_account,
            params={},
            retry_count=3
        ),
        WorkflowStep(
            name='send_welcome_email',
            action=send_welcome_email,
            params={},
            dependencies=['create_account']
        ),
        WorkflowStep(
            name='setup_tools_access',
            action=setup_tools_access,
            params={},
            dependencies=['create_account']
        )
    ]
    
    orchestrator.define_workflow('customer_onboarding', steps)
```

### 3. Event-Driven Architecture
```python
# event_handler.py
from typing import Callable, List, Dict, Any
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Event:
    """Represents an event in the system"""
    type: str
    source: str
    data: Dict[str, Any]
    timestamp: datetime
    id: str

class EventBus:
    """Central event bus for event-driven architecture"""
    
    def __init__(self):
        self.listeners: Dict[str, List[Callable]] = {}
    
    def subscribe(self, event_type: str, handler: Callable):
        """Subscribe to an event type"""
        if event_type not in self.listeners:
            self.listeners[event_type] = []
        self.listeners[event_type].append(handler)
    
    def publish(self, event: Event):
        """Publish an event"""
        print(f"Event published: {event.type} from {event.source}")
        
        if event.type in self.listeners:
            for handler in self.listeners[event.type]:
                try:
                    handler(event)
                except Exception as e:
                    print(f"Error handling event: {e}")
    
    def unsubscribe(self, event_type: str, handler: Callable):
        """Unsubscribe from an event type"""
        if event_type in self.listeners:
            self.listeners[event_type].remove(handler)

# Example event handlers
def handle_new_customer(event: Event):
    """Handle new customer event"""
    customer_data = event.data
    print(f"New customer registered: {customer_data['email']}")
    # Trigger onboarding workflow
    # Send notification to sales team
    # Update analytics

def handle_order_placed(event: Event):
    """Handle order placed event"""
    order_data = event.data
    print(f"New order: {order_data['order_id']}")
    # Update inventory
    # Notify fulfillment team
    # Send confirmation email
```

## Integration Patterns

### REST API Integration
```python
class RESTIntegration(IntegrationBase):
    """Generic REST API integration"""
    
    def make_request(self, method: str, endpoint: str, **kwargs):
        url = f"{self.api_url}{endpoint}"
        response = requests.request(method, url, headers=self.headers, **kwargs)
        response.raise_for_status()
        return response.json()
```

### Webhook Integration
```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/webhook/<integration_name>', methods=['POST'])
def handle_webhook(integration_name: str):
    """Handle incoming webhooks"""
    data = request.json
    event = Event(
        type=f"{integration_name}_webhook",
        source=integration_name,
        data=data,
        timestamp=datetime.now(),
        id=str(uuid.uuid4())
    )
    event_bus.publish(event)
    return {'status': 'received'}, 200
```

## Use Cases
1. **Customer Onboarding**: Coordinate across CRM, email, and access management
2. **Order Fulfillment**: Integrate inventory, shipping, and notification systems
3. **Incident Response**: Orchestrate monitoring, alerting, and resolution workflows
4. **Data Synchronization**: Keep multiple systems in sync
5. **Multi-Channel Communication**: Coordinate messaging across platforms

## Best Practices
- Use retry logic with exponential backoff
- Implement circuit breakers for failing integrations
- Log all integration calls for debugging
- Use async processing for long-running operations
- Implement proper error handling and rollback
- Monitor integration health continuously
- Cache frequently accessed data
- Use webhooks instead of polling when possible

## Dependencies
- Requests library for HTTP
- Async libraries (aiohttp, asyncio)
- Message queues (RabbitMQ, Kafka)
- Workflow engines (Apache Airflow, Prefect)
- API management tools
