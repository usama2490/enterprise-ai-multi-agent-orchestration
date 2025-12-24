# Enterprise Multi-Agent AI Orchestration Platform

## Problem Statement
Enterprise hiring decisions require explainability, consistency, and multi-source intelligence. 
Single-LLM approaches fail to scale, lack transparency, and are costly.

## Requirements
- Multi-agent reasoning
- Explainable outputs
- Cost and token optimization
- Azure OpenAI integration
- Reproducible & environment-independent design

## Architecture Overview
The system utilises an intelligent planner to dynamically orchestrate specialised AI agents based on query intent.

## Multi-Agent RAG Orchestrator Architecture

## Overview
The Enterprise RAG Orchestration platform is a sophisticated multi-agent system designed to provide comprehensive candidate analysis by orchestrating multiple specialized AI agents in Azure AI Foundry. It introduces intelligent query interpretation, dedicated synthesis agents, and advanced quality validation for gold-standard responses.

<img width="1633" height="659" alt="Enterprise-Multi-Agent-AI-Orchestration" src="https://github.com/user-attachments/assets/ad994636-34b6-46dd-85a8-47b9416f61b8" />

## Agent Architecture

### 1. CV Agent
- **Purpose**: Retrieves candidate information from CVs and resumes
- **Knowledge Base**: AI Search index with candidate CV data
- **Model**: GPT-4.1-mini
- **Search**: Hybrid (keyword + vector search)

### 2. Assessment Agent
- **Purpose**: Retrieves raw assessment scores (DISC, StrengthFinder, etc.)
- **Knowledge Base**: AI Search index with assessment data files
- **Model**: GPT-4.1-mini
- **Search**: Hybrid (keyword + vector search)
- **Note**: Contains raw scores but needs interpretation help

### 3. Guides Agent
- **Purpose**: Provides proprietary interpretation of assessment scores
- **Knowledge Base**: AI Search index with detailed guides and instructions
- **Model**: GPT-4.1-mini
- **Search**: Hybrid (keyword + vector search)
- **Note**: Contains proprietary guides for score interpretation

### 4. Interpreter Agent
- **Purpose**: Analyzes user queries and returns a structured interpretation
- **Model**: GPT-4.1-mini
- **Output**: JSON with intent, sources, entities, confidence, and safety
- **Configuration**: Prompts configured in the Azure AI Foundry portal

### 5. Synthesis Agent
- **Purpose**: Combines all agent results into a final comprehensive response
- **Model**: GPT-4.1-nano
- **Input**: Results from all other agents + deterministic mappings
- **Configuration**: Prompts configured in the Azure AI Foundry portal

## Execution Flow
1. User submits query
2. Interpreter analyzes query
3. Planner decides execution plan
4. Agents execute in parallel
5. Results are synthesized into a final response


## Enhanced Agent Collaboration

### Two-Stage Assessment Process

#### Stage 1: Raw Score Retrieval
1. Assessment Agent retrieves raw scores from the knowledge base
2. System extracts numerical scores using regex patterns
3. Scores are validated and structured

#### Stage 2: Score Interpretation
1. Guides Agent receives raw scores + original question
2. Guides Agent provides proprietary interpretation based on the knowledge base
3. System combines raw scores with interpreted insights
4. Result includes both raw data and actionable insights

### Concurrent Execution
- All agents run in parallel using ThreadPoolExecutor
- Results collected as they complete
- System continues even if some agents fail

## Intelligent Planning

### Query Interpretation
The system uses an AI-powered Interpreter Agent to analyze queries:

#### Structured Interpretation
- **Intent Classification**: factual, comparative, team, hypothetical, general
- **Source Detection**: cv_files, assessment_files, company_info
- **Entity Extraction**: Names, roles, skills, departments with confidence scores
- **Confidence Scoring**: 0.0-1.0 confidence in interpretation accuracy

#### Smart Agent Selection
- **High Confidence (≥0.6)**: Uses interpreter's source recommendations
- **Low Confidence (<0.6)**: Falls back to enhanced keyword analysis
- **Default Behavior**: Always uses assessment data unless explicitly limited

#### Enhanced Planning Logic
- **Context-Aware**: Agents receive rich context including interpretation results
- **Accuracy-First**: Prioritizes comprehensive analysis over cost optimization
- **Fallback Safety**: Graceful degradation when interpretation fails

## Response Structure

### Agent Results
```json
{
  "cv": {
    "success": true,
    "snippet": "...",
    "error": null
  },
  "assessment": {
    "success": true,
    "snippet": "...",
    "stage": "interpreted", // "raw_only" or "interpreted"
    "error": null
  },
  "guides": {
    "success": true,
    "snippet": "...",
    "error": null
  }
}
```

### Score Processing
```json
{
  "scores": {
    "DISC": 75,
    "StrengthFinder": 82
  },
  "mapped": {
    "DISC": {
      "label": "High",
      "interpretation": "High D: assertive, goal-driven, natural leader."
    }
  },
  "interpreted_scores": "Detailed proprietary interpretation from Guides agent..."
}
```

### Final Output (Enhanced in v2.0)
```json
{
  "question": "Compare John Doe and Jane Doe for UX lead position",
  "interpretation": {
    "intent": "comparative",
    "sources": ["cv_files", "assessment_files"],
    "entities": ["John Doe", "Jane Doe"],
    "confidence": 0.85
  },
  "plan": {
    "use_cv": true,
    "use_assess": true,
    "use_guides": true,
    "reasoning": {...}
  },
  "final_output": "**Analysis Results**\n\nComprehensive comparison...",
  "synthesis": {
    "success": true,
    "quality_validation": {
      "overall_score": 0.86,
      "meets_gold_standard": true,
      "improvement_areas": []
    }
  },
  "execution_time": 2.3,
  "cost_optimization": {
    "agents_used": 3,
    "cached_responses": 1,
    "estimated_cost_usd": 0.000010
  }
}
```

## Error Handling

### Graceful Degradation
- System continues if individual agents fail
- Detailed error logging for debugging
- Fallback to available data when possible
- Clear error messages in responses

### Retry Logic
- 3 retry attempts with exponential backoff
- Specific exception handling for different error types
- Comprehensive logging for troubleshooting

## Performance Optimizations

### Advanced Caching
- **Interpretation Cache**: 24-hour TTL for query interpretations
- **Planner Cache**: 5-minute TTL for agent selection decisions
- **Agent Response Cache**: 10-minute TTL for agent responses
- **Score Interpretation Cache**: 20-minute TTL for assessment interpretations
- **Semantic Cache**: 30-minute TTL for similar questions
- **Multi-level caching**: Reduces costs and improves response times

### Concurrent Execution
- Parallel agent calls reduce response time
- ThreadPoolExecutor with optimal worker count
- Results collected as they complete

### Structured Logging
- JSON-formatted logs for easy parsing
- Detailed timing and performance metrics
- Error tracking and debugging information

## Usage Examples

### Comparison Question
```bash
curl -X POST "http://127.0.0.1:8000/ask" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "Compare John Doe and Jane Doe for a UX/UI design team lead position.",
    "mode": "auto"
  }'
```

### Assessment Question
```bash
curl -X POST "http://127.0.0.1:8000/ask" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What is John Doe's DISC score and what does it mean?",
    "mode": "auto"
  }'
```

### CV-Only Question
```bash
curl -X POST "http://127.0.0.1:8000/ask" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What is John Doe's work experience?",
    "mode": "cv_only"
  }'
```

## Key Engineering Challenges
- Preventing duplicate agent calls
- Extracting structured data from LLM output
- Ensuring deterministic behavior
- Managing token costs

## Design Decisions
- Guides agent is embedded inside the assessment flow
- Interpreter confidence controls fallbacks
- Caching is used for repeated score interpretations

## Scalability & Reproducibility
- Stateless agents
- IaC-ready design (Terraform)
- Multi-tenant friendly
- Portable across Azure subscriptions

## Business Outcomes
- Faster hiring decisions
- Explainable AI outputs
- Reduced operational costs

## Disclaimer
This repository demonstrates architecture patterns only.
