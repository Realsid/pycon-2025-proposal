# Building Scalable AI Decision Systems in Python: Balancing Automation and Human Review in Checkout Free Retail

## Abstract

Autonomous checkout-free stores represent the future of retail, allowing customers to simply take products and walk out without scanning or waiting in line. However, these systems create significant technical challenges: they must identify who took what products from where, and do so with extraordinary accuracy while controlling operational costs.

This talk explores how we built a distributed Python-based decision system that serves as a configurable proxy between AI models and human reviewers across 100+ retail locations worldwide. By implementing an asynchronous architecture with FastAPI, Redis Streams, and custom Python workers, we created a platform that reduced human review workload by 30% while maintaining the high accuracy essential for customer trust.

You'll learn practical techniques for building configurable Python systems that intelligently route decisions between automation and human review, approaches for handling location-specific configurations at scale, and strategies for safely deploying experimental ML models without disrupting critical production components.

## Who Should Attend

This talk is aimed at intermediate to advanced Python developers who:
- Are building systems that combine ML automation with human review
- Need to deploy ML models safely in production environments
- Work with distributed systems that must handle variable load
- Are interested in event-driven architectures for real-time decisions

**Prerequisites**: Familiarity with Python web frameworks, basic understanding of async programming concepts, and interest in ML deployment patterns.

## Outline

### Core Challenges of Autonomous Retail (5 mins)
- The four fundamental questions: who, what, when, and where
- Multiple decision points requiring intelligence:
  - Multi-camera tracking across store environments
  - Product identification in varying conditions
  - Action recognition (take vs. return)
  - Cart verification and reconciliation
- The human-in-the-loop (HITL) dilemma:
  - Need for human review in ambiguous cases
  - Reducing HITL cost while maintaining accuracy
  - The ROI curve: investment in decision systems vs. operational costs
- Decoupling specific solutions from general decision architecture
- Creating systems for safe experimentation with new algorithms

### System Architecture (15 mins)

#### Functional and Non-functional Requirements
- Store-specific configuration capabilities
- Consistent API signature across automated and human decisions
- High precision prioritized over recall (with fallback to human review)
- Ability to handle traffic bursts during peak shopping hours
- Hot-reloading of configurations without service interruption
- Comprehensive monitoring and observability

#### Technology Selection and Comparison

- **API Framework Selection**
  - Why we selected FastAPI over alternatives for our use case

- **Message Queue Evolution**
  - Our journey from Celery to Redis Streams
  - Critical lessons from production issues (referencing the Celery bug: https://github.com/celery/celery/discussions/7276)
  - Other available options and why we chose to build our own smol framework.
  - Why Redis Streams provided the right balance for our specific requirements

- **Worker Implementation**
  - Standalone Python worker built on top of redis streams
  - Design patterns for configuration management
  - Dynamic task routing based on store profiles
  - Alternative approaches and when they might be preferable

- **Deployment Architecture**
  - Container orchestration with Kubernetes
  - Database design for tracking decision paths
  - Monitoring and observability implementation

### Real-world Results and Lessons (10 mins)

- Performance metrics:
  - 50,000+ daily tasks processed across all environments
  - Reduction in human review costs by 30%
  - Decision time improvements and customer experience impact
  
- Technical insights:
  - Challenges with scaling async code in production
  - Testing strategies for complex event-driven systems
  
### Future Work and Broader Applications (3 mins)

- Applications beyond retail:
  - Similar patterns for other human-AI collaboration domains
  - Adaptation to different decision-making contexts
  - Extension to other industries and problem spaces
  
- Next steps in the architecture:
  - Federated learning opportunities
  - Enhanced explainability for automated decisions
  - More granular fallback strategies
  
- Open questions for the community to explore

## Speaker Bio

## Supplementary Materials

## Talk Links
