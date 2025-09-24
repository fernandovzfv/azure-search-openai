# Azure AI Search Cost Optimization Plan

## Context
This plan outlines strategies to reduce Azure AI Search costs for the Azure Search and OpenAI demo application, specifically tailored for technical documents containing tables, images, and figures. The goal is to maintain response quality while minimizing unnecessary processing for a small user base.

## Key Considerations
- **Data Characteristics**: Technical documents with multimodal elements (tables, images, figures)
- **Quality Requirements**: Must preserve retrieval effectiveness for complex technical content
- **User Base**: Small number of users, allowing for optimizations like caching
- **Current Architecture**: Uses vector search, semantic ranker, and multimodal features

## Optimization Strategies

### 1. Enable Agentic Retrieval
**Description**: Switch to agentic retrieval which uses AI to intelligently plan and execute searches, potentially reducing query volume and improving precision for technical content.

**Implementation**:
- Set `USE_AGENTIC_RETRIEVAL=true` in environment variables
- This replaces traditional search with agent-driven retrieval

**Benefits**:
- Reduces redundant searches
- Better handling of multimodal technical content
- Potentially lower overall query costs

**Risks**: Adds OpenAI API costs for agent model usage

### 2. Optimize Vector Search Configuration
**Description**: Maintain vector search for quality (essential for tables/images) but optimize usage patterns.

**Implementation**:
- Keep `USE_VECTORS=true`
- Set `RAG_SEARCH_TEXT_EMBEDDINGS=true` selectively
- Modify approach logic to prioritize vector search for image/table retrieval

**Benefits**:
- Preserves quality for multimodal elements
- Reduces unnecessary vector computations for pure text queries

### 3. Implement Result Caching
**Description**: Cache search results and embeddings to avoid repeated computations for frequently accessed technical content.

**Implementation**:
- Add in-memory or Redis caching layer
- Cache based on query hash and user context
- Example code addition to `approaches/approach.py`:
```python
import hashlib
from functools import lru_cache

@lru_cache(maxsize=1000)
def cached_search(query_hash, *args):
    # Implement caching logic
```

**Benefits**:
- Eliminates redundant searches for common technical queries
- Particularly effective with small user base

### 4. Reduce Default Top Results
**Description**: Lower the number of retrieved results from 3 to 2 to reduce processing while maintaining sufficient context.

**Implementation**:
- Change `top = overrides.get("top", 3)` to `top = overrides.get("top", 2)` in:
  - `app/backend/approaches/retrievethenread.py`
  - `app/backend/approaches/chatreadretrieveread.py`

**Benefits**:
- Direct cost reduction in result processing
- Minimal quality impact for detailed technical content

### 5. Optimize Indexing Operations
**Description**: Prevent unnecessary re-indexing of technical documents.

**Implementation**:
- Modify `app/backend/prepdocs.py` to check for document changes before re-indexing
- Implement incremental updates for document modifications
- Add data deduplication logic

**Benefits**:
- Reduces indexing operation costs
- Minimizes storage churn

### 6. Enable Semantic Captions
**Description**: Use semantic captions to provide richer context from search results without additional vector costs.

**Implementation**:
- Enable `semantic_captions` by default in approach files
- Set `use_semantic_captions = True` in search method calls

**Benefits**:
- Improves response quality for technical content
- Low additional cost compared to vector search

### 7. Tune Quality Thresholds
**Description**: Adjust scoring thresholds to filter out low-quality results early.

**Implementation**:
- Modify `minimum_search_score` and `minimum_reranker_score` defaults
- Add cost and usage monitoring/logging

**Benefits**:
- Reduces processing of irrelevant results
- Maintains quality for technical documents

## Estimated Cost Reduction
- **Overall Savings**: 30-50% reduction in Azure AI Search costs
- **Primary Drivers**: Agentic retrieval, caching, and optimized vector usage
- **Quality Impact**: Minimal to none, with potential improvements for technical content

## Implementation Priority
1. **High Priority**: Enable agentic retrieval and implement caching
2. **Medium Priority**: Optimize vector search configuration and reduce top results
3. **Low Priority**: Enable semantic captions and tune thresholds

## Monitoring and Validation
- Track query costs and performance metrics
- Monitor response quality through user feedback
- Adjust thresholds based on actual usage patterns
- Re-evaluate optimizations quarterly

## Rollback Plan
- All changes are configurable via environment variables
- Can disable features individually if quality issues arise
- Maintain baseline configuration for comparison