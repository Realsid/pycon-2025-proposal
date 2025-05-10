# Python at the Checkout
## Building Scalable AI Decision Systems That Reduced Human Review by 30%

Note: 
- Welcome everyone! I'm excited to share how we built a scalable AI decision system at Zippin that reduced our human review workload by 30%.
- This talk will walk through our journey from problem to solution, with practical insights you can apply to your own projects.

---

## About Me

- Senior ML Engineer at Zippin
- Building distributed ML systems for checkout-free retail
- Python enthusiast working with ML infrastructure
- [Your additional background]

Note:
- Brief introduction about myself
- Mention relevant experience that establishes credibility for this topic

---

## What is Checkout-Free Retail?

![Checkout-Free Store](https://via.placeholder.com/800x400.png?text=Checkout-Free+Store+Example)

- Customers walk in, take products, and walk out
- No scanning, no waiting in line
- AI determines: Who took What from Where and When

Note:
- Brief explanation of checkout-free technology for context
- The fundamental technical challenge: identifying who took what items

---

## The Challenge

<!-- .slide: data-background="#f7f7f7" -->

- 100+ stores globally, many operating 24/7
- Multiple decision points across the shopping journey
- Need for high accuracy while controlling costs
- Human-in-the-Loop (HITL) for critical decisions
- ML research team needs rapid deployment pathway

Note:
- Explain the scale of the operation
- Highlight why this is a difficult technical problem
- Introduce the need for human review in certain cases

---

## The Core Problem

![HITL Cost vs System Capability](https://via.placeholder.com/800x400.png?text=HITL+Cost+vs+System+Capability+Graph)

- Human review is accurate but expensive and doesn't scale
- Pure automation risks customer experience issues
- Need a balanced approach that's configurable per store

Note:
- Visual showing the tradeoff between human review costs and automated system capability
- Emphasize the business problem: reducing operational costs while maintaining accuracy

------

## Decision Points in Checkout-Free Retail

1. Multi-camera tracking <!-- .element: class="fragment" -->
2. Product identification <!-- .element: class="fragment" -->
3. Action recognition (take vs. return) <!-- .element: class="fragment" -->
4. Cart verification <!-- .element: class="fragment" -->

Note:
- Break down the specific technical challenges in more detail
- Each of these represents an AI decision point that could require human verification
- Explain why these are difficult problems (occlusion, similar products, etc.)

---

## Our Solution

<!-- .slide: data-background="#e8f4f8" -->

A distributed Python-based proxy system that:

- Acts as an intelligent layer between store systems and human reviewers
- Routes decisions based on confidence scores and store-specific rules
- Allows safe testing of new ML models without disrupting critical flows
- Provides consistent API interfaces for both automated and human decisions

Note:
- Introduce the high-level solution architecture
- Emphasize that we created a system that looks the same to other components whether decisions come from humans or ML

---

## System Requirements

### Functional Requirements:
- Store-specific configuration
- Drop-in API compatibility with HITL backend
- High precision over recall (safety first)

### Non-Functional Requirements:
- Scale to handle traffic bursts
- Support hot reloading of configurations
- Real-time performance

Note:
- Detail both the functional and non-functional requirements
- Explain why each requirement matters in this context
- Highlight the precision vs. recall tradeoff - we prefer to fall back to humans than make wrong automated decisions

---

## Architecture Overview

```python
# Simplified service structure
@app.post("/decision/{task_type}")
async def handle_decision(task_type: str, request: DecisionRequest):
    # Send to queue for async processing
    task_id = await enqueue_task(task_type, request)
    
    # Return immediately with task ID
    return {"task_id": task_id, "status": "processing"}
```

![System Architecture](https://via.placeholder.com/800x350.png?text=System+Architecture+Diagram)

Note:
- Walk through the high-level architecture
- Explain the API-first approach
- Highlight the asynchronous processing model

------

## Technology Stack

<!-- .slide: data-background="#f0f0f0" -->

- **FastAPI**: High-performance API server
- **Redis Streams**: Task distribution mechanism
- **Standalone Python workers**: Decision processing
- **MySQL**: Task session storage
- **Kubernetes**: Container orchestration

Note:
- Go through each technology choice and explain why it was selected
- Highlight the Python-centric nature of the stack
- Mention other options we considered before making these choices

------

## The Celery Journey

```python
# Initial Celery implementation
@app.task(bind=True, acks_late=True)
def process_decision_task(self, task_type, task_data):
    try:
        result = decision_models[task_type].predict(task_data)
        if result.confidence > CONFIDENCE_THRESHOLD:
            return {"decision": result.decision, "automated": True}
        else:
            return forward_to_hitl(task_data)
    except Exception as e:
        self.retry(exc=e, countdown=5)
```

### The Critical Bug
![Celery Bug](https://via.placeholder.com/800x200.png?text=Celery+Task+Acknowledgment+Issue)

Note:
- Share the story of initially using Celery
- Explain the critical bug we encountered (link to the actual GitHub issue)
- Lessons learned about production deployment of task queues

------

## Redis Streams Solution

```python
# Redis Streams consumer
async def process_stream():
    while True:
        # Read new messages from stream
        streams = await redis.xread(
            streams={STREAM_KEY: last_id}, 
            count=10,
            block=5000
        )
        
        for stream_id, messages in streams:
            for message_id, data in messages:
                try:
                    await process_message(data)
                    # Acknowledge only after successful processing
                    await redis.xack(STREAM_KEY, GROUP_NAME, message_id)
                except Exception as e:
                    logger.error(f"Error processing message: {e}")
                
                last_id = message_id
```

Note:
- Explain how we transitioned to Redis Streams
- Highlight the reliability improvements
- Show how we handle acknowledgments and error cases

---

## Worker Implementation

```python
class DecisionWorker:
    def __init__(self, store_id, config_path):
        self.store_id = store_id
        self.config = self._load_config(config_path)
        self.models = {}
        self._initialize_models()
    
    async def process_task(self, task_data):
        task_type = task_data.get('type')
        model = self.models.get(task_type)
        
        if not model:
            return await self._forward_to_hitl(task_data)
            
        result = model.predict(task_data['input'])
        
        # Apply store-specific confidence thresholds
        threshold = self.config['thresholds'].get(task_type, 0.95)
        
        if result['confidence'] >= threshold:
            return {'decision': result['prediction'], 'automated': True}
        else:
            return await self._forward_to_hitl(task_data)
```

Note:
- Walk through the core worker implementation
- Highlight the configuration loading and threshold application
- Show how workers make the decision to automate or forward to humans

------

## Dynamic Configuration

```yaml
# store_config.yaml
store_id: store_123
models:
  product_identification:
    model_path: /models/product_id_v3.pkl
    batch_size: 32
  customer_tracking:
    model_path: /models/tracking_v2.onnx
    max_history_frames: 30
thresholds:
  product_identification: 0.92
  customer_tracking: 0.85
  cart_verification: 0.98
```

```python
# Hot reload implementation
def _watch_config_changes(self):
    last_modified = os.path.getmtime(self.config_path)
    
    while True:
        time.sleep(30)  # Check every 30 seconds
        current_modified = os.path.getmtime(self.config_path)
        
        if current_modified > last_modified:
            logger.info(f"Config changed for store {self.store_id}")
            self.config = self._load_config(self.config_path)
            self._reinitialize_models()
            last_modified = current_modified
```

Note:
- Explain the store-specific configuration approach
- Show how we implement hot reloading of configurations
- Discuss the importance of per-store customization

---

## Deployment Architecture

![Kubernetes Deployment](https://via.placeholder.com/800x400.png?text=Kubernetes+Deployment+Architecture)

- Separate deployments per store cluster
- Autoscaling based on queue depth
- Centralized monitoring and alerting
- Blue/green deployment for updates

Note:
- Discuss the Kubernetes deployment architecture
- Explain how we handle scaling for different traffic patterns
- Talk about the deployment and monitoring strategy

---

## Results

<!-- .slide: data-background="#e6f7e6" -->

- Serving 50,000+ tasks daily across all stores
- Reduced human review workload by 30%
- Improved average decision time by 45%
- Created a platform for ML researchers to deploy models safely

![Results Dashboard](https://via.placeholder.com/800x400.png?text=Performance+Metrics+Dashboard)

Note:
- Share the concrete metrics and results
- Highlight both the business impact and technical achievements
- Show a visualization of the performance over time

---

## Lessons Learned

1. Test message broker behavior under failure conditions <!-- .element: class="fragment" -->
2. Store-specific configurations are essential for real-world deployment <!-- .element: class="fragment" -->
3. Start with high precision requirements and gradually increase automation <!-- .element: class="fragment" -->
4. Design for observability from day one <!-- .element: class="fragment" -->
5. Create a safe path for experimental models <!-- .element: class="fragment" -->

Note:
- Share key insights and lessons from the project
- Be honest about what worked well and what didn't
- Focus on actionable takeaways for the audience

---

## Future Work

- Federated learning across stores for improved models
- Automated configuration tuning based on performance metrics
- Enhanced explainability for decision review
- More granular fallback strategies

Note:
- Discuss what we're planning next
- Show that this is an evolving system
- Highlight interesting research directions

---

## Thank You!

**Questions?**

- Twitter: @yourhandle
- GitHub: github.com/yourusername
- Email: your.email@example.com

![QR Code](https://via.placeholder.com/300x300.png?text=Contact+Info)

Note:
- Thank the audience
- Provide contact information
- Be ready for Q&A
