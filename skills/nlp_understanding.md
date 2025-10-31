# Skill: Natural Language Processing (NLP)

## Overview
Core skill enabling agents to understand and generate human language across multiple languages and contexts.

## Capabilities
- Text understanding and interpretation
- Intent classification
- Entity extraction
- Sentiment analysis
- Language translation
- Text generation

## Technical Components

### 1. Language Understanding
```python
# Example: Intent Classification
def classify_intent(user_input):
    """
    Classifies user intent from natural language input
    
    Args:
        user_input: String containing user's message
        
    Returns:
        Intent classification with confidence score
    """
    # Pre-trained model or fine-tuned custom model
    intents = {
        "question": ["what", "how", "why", "when", "where"],
        "request": ["please", "can you", "would you", "need"],
        "complaint": ["issue", "problem", "not working", "error"],
        "feedback": ["great", "terrible", "good", "bad"]
    }
    
    # Simple keyword matching (replace with ML model in production)
    for intent, keywords in intents.items():
        if any(keyword in user_input.lower() for keyword in keywords):
            return {"intent": intent, "confidence": 0.85}
    
    return {"intent": "unknown", "confidence": 0.5}
```

### 2. Entity Extraction
```python
# Example: Extract key information from text
def extract_entities(text):
    """
    Extracts named entities and key information
    
    Args:
        text: Input text to analyze
        
    Returns:
        Dictionary of extracted entities
    """
    entities = {
        "dates": [],
        "amounts": [],
        "products": [],
        "locations": []
    }
    
    # Use NLP library (spaCy, NLTK, or transformer models)
    # This is a simplified example
    import re
    
    # Extract dates
    date_pattern = r'\d{1,2}[/-]\d{1,2}[/-]\d{2,4}'
    entities["dates"] = re.findall(date_pattern, text)
    
    # Extract amounts
    amount_pattern = r'\$\d+(?:,\d{3})*(?:\.\d{2})?'
    entities["amounts"] = re.findall(amount_pattern, text)
    
    return entities
```

## Integration Patterns

### REST API
```json
POST /api/nlp/analyze
{
  "text": "I need help with my order #12345. It hasn't arrived yet.",
  "analysis_types": ["intent", "entities", "sentiment"]
}

Response:
{
  "intent": {
    "primary": "support_request",
    "confidence": 0.92
  },
  "entities": {
    "order_id": "12345",
    "issue_type": "delivery_delay"
  },
  "sentiment": {
    "score": -0.3,
    "label": "concerned"
  }
}
```

## Use Cases
1. **Customer Support**: Understanding customer queries
2. **Document Processing**: Extracting information from documents
3. **Content Moderation**: Detecting inappropriate content
4. **Search**: Semantic search and query understanding
5. **Chatbots**: Natural conversation interfaces

## Dependencies
- Pre-trained language models (BERT, GPT, T5)
- NLP libraries (spaCy, transformers, NLTK)
- Language detection tools
- Custom fine-tuned models for domain-specific tasks

## Performance Considerations
- Model size vs. inference speed trade-offs
- Caching for common queries
- Batch processing for efficiency
- GPU acceleration for large models
