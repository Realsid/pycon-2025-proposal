# Building Scalable AI Decision Systems in Python: Balancing Automation and Human Review in Checkout Free Retail

## Abstract

Autonomous checkout-free stores represent the future of retail, allowing customers to simply take products and walk out without scanning or waiting in line. However, these systems create significant technical challenges: they must identify who took what products from where, and do so with extraordinary accuracy and strict latency requirements while controlling operational costs.

This talk explores how we built a distributed Python-based decision system that serves as a configurable proxy between decision systems responsible for tracking and cart verification and human reviewers across 100+ retail locations worldwide. By implementing an asynchronous architecture with FastAPI, Redis Streams, and custom Python workers, we created a platform that reduced human review workload by 30% while maintaining the high accuracy essential for customer trust.

We will share practical techniques for building configurable Python systems that intelligently route decisions between automation and human review, approaches for handling location-specific configurations at scale, and strategies for safely deploying experimental ML models without disrupting critical production components. By the end of the talk, you will have a good understanding what it takes to build a scalable decision system that can be deployed in a production environment.

## Who Should Attend

This talk is aimed at intermediate to advanced Python developers who:
- Who want to understand the current SOTA in checkout free retails systems
- Are building systems that combine ML automation with human review
- Need to deploy ML models safely in production environments
- Work with distributed systems that must handle variable load
- Are interested in event-driven architectures for real-time decisions

**Prerequisites**: Familiarity with Python web frameworks, basic understanding of async programming concepts, and interest in ML deployment patterns.

## Outline

## Introduction to checkout-free retail (5 mins)
- We will begin by answering the question "What is checkout-free retail?"
- We will introduce the four fundamental questions of checkout free retail: WHO picked WHAT from WHERE and WHEN
- We will then dive into the various decision systems that are used to make decisions in checkout free retail:
  - Person tracking service: Responsible for tracking people in the store
  - Product recognition service: Responsible for recognizing the products in the store
  - Action recognition service: Responsible for recognizing the action of the person in the store near shelves could be based on camera feeds or sensors
  - Cart services : Responsible for reconciling processing tracking data with cart state and generating receipts

### Core Challenges 
- We will then dive into the multiple decision points requiring intelligence:
  - Multi-camera tracking across store environments
  - Product identification in varying conditions
  - Action recognition (take vs. return)
  - Cart verification and reconciliation
- We will then discuss the human-in-the-loop (HITL) dilemma:
  - Retail is a low margin business and customer experience is paramount
  - You have to avoid charging the wrong customer for wrong products and also avoid losing money from products that are taken without paying. Human review is expensive and you can't scale it.
  - Need for human review in ambiguous cases.
  - Reducing HITL cost while maintaining accuracy
- We will introduce the need for a middle layer between decision systems and human reviewers.
  - Introduce "The Bitter lesson", how programming rules dont scale and how we need to decouple specific solutions from general decision architecture, but that doesnt mean that programmed rules dont yield gains. They just dont scale, so do not over engineer your general decision system.
  - Creating systems for safe experimentation with new algorithms

## System Architecture (15 mins)

### Functional and Non-functional Requirements
- High precision prioritized over recall (with fallback to human review)
- Ability to handle traffic bursts during peak shopping hours
- Hot-reloading of configurations without service interruption
- Comprehensive monitoring and observability at all layers, specially the decision logic to track effectiveness of the system

### Technology Selection and Comparison

- Fast API for the web 
- Mysql to persist task session data

- **Message Queue Evolution**
  - Our journey from Celery to Redis Streams
  - Critical lessons from production issues (referencing the Celery bug: https://github.com/celery/celery/discussions/7276)
  - Lesson: Scan for open issues in the library you are using.
  - Lesson: If you dont require generalist capabilities, building your own small framework might be the better option.
  - Why Redis Streams provided the right balance for our specific requirements

- **Worker Implementation**
  - Standalone Python worker built on top of redis streams
  - Design patterns for configuration management
    - Config cached in Redis
    - Invalidated on updates to the database
  - Dynamic task routing
    - First level of routing is based on the service type (tracking, cart verification, etc)
    - Second level of routing is configuration for stores
  - Fail Early, Fail Loudly

- **Monitoring and Observability**
  - We will discuss importance of contextual logging and monitoring to debug production issues
  - How we monitor the system using Grafana 

- **Deployment Architecture**
  - Container orchestration with Kubernetes
  - Horizontal auto scaling of workers using Kubernetes Event Driven Autoscaler (KEDA)
  - Monitoring and observability implementation

## Results and Lessons (5 mins)

- Performance metrics:
  - 50,000+ daily tasks processed across all environments
  - Reduction in human review costs by 30%
  - Decision time improvements and customer experience impact

- Critical production lessons:
  - Have a plan for deployment and rollbacks for releases
  - Contextual logging is key to debugging production issues
  - Avoiding dual write patterns to databases to avoid inconsistencies
  - Preventing worker blockages with appropriate timeout configurations
  - Designing idempotent operations for safe retries

## Q/A (5 mins)

## Speaker Bio

Hareesh Kolloru, Head of AI/ML at Motive

Sidharth Singh, Staff Software Engineer, ML at Zippin, has 8 years of experience architecting ML systems processing 10M+ daily inference requests across 120+ retail locations. Led design and implementation of distributed ML pipelines across retail, manufacturing, and enterprise domains. 

## Supplementary Materials

## Talk Links
