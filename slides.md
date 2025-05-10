# Building Scalable AI Decision Systems in Python
## Balancing Automation and Human Review in Retail

Note: Opening slide - title only

---

## Autonomous Retail Challenge

- **The Four Critical Questions**: Who took What from Where and When
- **Multiple Decision Points**: Camera tracking, product recognition, action detection
- **Accuracy vs. Cost**: Customer trust requires precision

Note: Opening context slide

---

## The Human-in-the-Loop Dilemma

- Human review: **accurate but expensive**
- Pure automation: **risky for customer experience**
- Need a **balanced approach** configurable per location

---

## Our Challenge

- 100+ stores globally, many operating 24/7
- 50,000+ decision tasks daily
- Different store layouts and product selections
- Need to reduce human review cost by 30%
- Provide platform for ML researchers to safely test new algorithms

---

## System Architecture

- **API Layer**: FastAPI for high-performance async endpoints
- **Task Distribution**: Redis Streams for reliable messaging
- **Workers**: Custom Python workers with store-specific configuration
- **Storage**: MySQL + SQLAlchemy for task history and auditing
- **Deployment**: Kubernetes for scalable orchestration

---

## Technology Selection Rationale

- **FastAPI vs Flask/Django**: Async support, performance, built-in validation
- **Redis Streams vs Celery/RabbitMQ/Kafka**: Simpler operations, built-in consumer groups 
- **Custom Workers vs Off-the-shelf**: Fine-grained control over decision logic
- **MySQL vs NoSQL**: Transaction support for decision auditing

---

## Celery to Redis Streams Migration

```python
# Redis Streams consumer pattern
async def process_stream():
    while True:
        streams = await redis.xread(
            streams={STREAM_KEY: last_id}, 
            count=10,
            block=5000
        )
        
        for message_id, data in streams[0][1]:
            try:
                await process_message(data)
                # Only acknowledge after processing
                await redis.xack(STREAM_KEY, GROUP_NAME, message_id)
            except Exception as e:
                logger.error(f"Error processing: {e}")
                
            last_id = message_id
```

---

## Key Python Features Used

1. **Async/await**: Non-blocking operations for higher throughput
2. **Type annotations**: Complex data flow validation
3. **Context managers**: Resource management for workers
4. **Dataclasses**: Clean configuration representation
5. **Dependency injection**: Testable service components

```python
@dataclass
class StoreConfig:
    store_id: str
    confidence_thresholds: Dict[str, float]
    fallback_strategy: str = "human_review"
    
    def should_automate(self, task_type: str, confidence: float) -> bool:
        threshold = self.confidence_thresholds.get(task_type, 0.95)
        return confidence >= threshold
```

---

## Configuration Management

```yaml
# store_config.yaml
store_id: store_123
models:
  product_identification:
    model_path: /models/product_id_v3.pkl
    batch_size: 32
  customer_tracking:
    model_path: /models/tracking_v2.onnx
thresholds:
  product_identification: 0.92
  customer_tracking: 0.85
  cart_verification: 0.98
```

- Hot-reloading configuration without service restart
- Store-specific confidence thresholds
- Model versioning and selection

---

## Worker Implementation

```python
class DecisionWorker:
    def __init__(self, store_id: str, config_path: str):
        self.store_id = store_id
        self.config = self._load_config(config_path)
        self.models = self._initialize_models()
    
    async def process_task(self, task_data: Dict[str, Any]) -> Dict[str, Any]:
        task_type = task_data.get('type')
        
        # Apply store-specific confidence thresholds
        threshold = self.config.thresholds.get(task_type, 0.95)
        
        try:
            result = await self._run_model(task_type, task_data['input'])
            
            if result['confidence'] >= threshold:
                return {'decision': result['prediction'], 'automated': True}
            else:
                return await self._forward_to_hitl(task_data)
        except Exception as e:
            logger.error(f"Error in model: {e}")
            return await self._forward_to_hitl(task_data)
```

---

## Decision Flow

1. Request comes into API 
2. Task sent to appropriate worker
3. Worker applies ML models with store configuration
4. If confidence > threshold: automated decision
5. Otherwise: forward to human review
6. Results stored for training and audit

---

## Results

<!-- .slide: data-background="#e6f7e6" -->

- Serving **50,000+ tasks daily** across global stores
- Reduced human review workload by **30%**
- Improved average decision time by **45%**
- Created platform for ML researchers to test new models

```python
# Python-based monitoring dashboard
import altair as alt

# Create visualization of automated vs. human decisions
chart = alt.Chart(decisions_df).mark_area().encode(
    x='date:T',
    y='automated_decisions:Q',
    y2='human_decisions:Q',
    tooltip=['date', 'automated_decisions', 'human_decisions']
).properties(
    title='Automated vs Human Decisions Over Time'
)
```

---

## Python-Specific Lessons

1. **Type hints are crucial** for complex data flows
2. **Async programming** requires careful error handling
3. **Hot module reloading** is powerful but has edge cases
4. **Monitoring is essential** - instrument everything
5. **Test the error paths** thoroughly - especially with queues

---

## Broader Applications

- Similar patterns work for:
  - Content moderation systems
  - Medical diagnosis assistance
  - Financial fraud detection
  - Document processing pipelines

- Key pattern: **Human-AI collaboration** requires reliable handoffs

---

## Key Takeaways

1. **Balance automation and human review** based on confidence
2. **Store-specific configuration** is essential for real-world deployment
3. **Plan for safe experimentation** with new algorithms
4. **Design for observability** from day one
5. **Python's ecosystem** is well-suited for ML decision systems
