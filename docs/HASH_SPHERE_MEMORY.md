# 🧠 Hash Sphere Memory System

## Overview

The **Hash Sphere Memory System** is ResonantGenesis's revolutionary 9-layer architecture for semantic memory storage, retrieval, and visualization. Unlike traditional vector databases, Hash Sphere Memory combines cryptographic hashing, 3D spatial coordinates, physics-based resonance scoring, and multi-method retrieval to create a truly intelligent memory system.

## 🏗️ 9-Layer Architecture

### Layer 1: Input Processing
- **Normalization**: Text preprocessing and cleaning
- **Tokenization**: Breaking down input into semantic units
- **Context Extraction**: Identifying relevant contextual information

### Layer 2: Hash Generation
Multiple hash types for different semantic properties:
- **Meaning Hash**: SHA-256 hash of semantic content (20 chars)
- **Energy Hash**: Hash of emotional/intensity indicators (8 chars)
- **Spin Hash**: Hash of sentiment/direction (8 chars)
- **Universe ID**: 256-bit SHA-256 hash for unique identification

### Layer 3: Universe ID
- **Deterministic Identity**: Each memory gets unique 256-bit identifier
- **Collision Resistance**: Cryptographically secure hash prevents duplicates
- **Temporal Uniqueness**: Nanosecond-precision timestamps ensure uniqueness

### Layer 4: Embedding Generation
- **Vector Embeddings**: 1536-dimensional semantic vectors
- **Task-Specific Prefixes**: Different embeddings for storage vs. retrieval
- **Caching**: Intelligent embedding cache for performance

### Layer 5: Coordinate Calculation
Memories are positioned in 3D semantic space:

**Cartesian Coordinates (x, y, z)**:
- Derived from semantic embeddings using PCA (Principal Component Analysis)
- Similar meanings cluster together in 3D space
- Normalized to consistent scale for visualization

**Hyperspherical Coordinates (r, φ, θ)**:
- **r**: Radius (distance from origin, typically 1.0 for unit sphere)
- **φ (phi)**: Latitude in radians (-π/2 to π/2)
- **θ (theta)**: Longitude in radians (-π to π)

### Layer 6: Resonance Scoring

**Resonance Function**: `R(h) = sin(a·x) + cos(b·y) + tan(c·z)`

Where:
- `a = π/4` (resonance coefficient for x)
- `b = e/3` (resonance coefficient for y)
- `c = φ/2` (resonance coefficient for z, φ = golden ratio)

**Normalized Resonance**: Sigmoid normalization to 0-1 range
- `R_normalized = 1 / (1 + e^(-R))`

**Anchor Energy**: `E_j(s) = exp(-β·||s - A_j||²)`
- Measures attraction to nearest anchor point
- β controls "sharpness" of resonance
- Higher energy = stronger alignment

### Layer 7: Evidence Aggregation

**Weighted Sum**: `E* = Σ_{i∈R} w_i · s_i`

Combines multiple memory positions into single evidence vector:
- Weighted by relevance scores
- Normalized to unit sphere
- Used for context-aware retrieval

### Layer 8: Multi-LLM Routing
- **Model Selection**: Choose optimal LLM based on query type
- **Load Balancing**: Distribute requests across multiple models
- **Fallback Logic**: Automatic failover to backup models

### Layer 9: Output Correction

**Correction Formula**: `o_corrected = λ·o_k* + (1-λ)·Ê*`

Blends LLM output with evidence to reduce hallucination:
- λ = 0.8 (80% LLM, 20% evidence) - typical value
- Ensures outputs are grounded in actual memories
- Prevents AI from inventing non-existent information

## 🎯 Advanced Features

### Spin Vectors
**Semantic Rotation**: 3D vectors representing semantic "direction"
- **X-axis**: Topic/domain direction (technical ↔ creative)
- **Y-axis**: Emotional valence (positive ↔ negative)
- **Z-axis**: Complexity/abstraction level

**Spin Magnitude**: `||spin|| = √(spin_x² + spin_y² + spin_z²)`

### Semantic Components
- **Meaning Score**: Content richness (vocabulary diversity, length)
- **Intensity Score**: Emotional/urgency indicators (0-1)
- **Sentiment Score**: Positive/negative sentiment (0-1, 0.5 = neutral)

### Cluster Assignment
- **K-means Clustering**: Automatic grouping of similar memories
- **Centroid Calculation**: Average position of cluster members
- **Cluster Names**: Semantic labels for memory groups

### Magnetic Pull System (HS-MPS)
**Non-linear Boost**: `magnetic = min((resonance² × 1.5), 1.0)`

Amplifies strong memories:
- Low resonance (0.3) → 0.135 (weaker)
- Medium resonance (0.6) → 0.54 (moderate)
- High resonance (0.9) → 1.0 (strong, capped)

### Drift & Decay
**Drift Formula**: `s_{t+1} = s_t + γ(A_{j*} - s_t)`

Memories gradually move toward anchor points:
- γ ∈ [0,1] = drift coefficient (decay rate)
- Simulates memory consolidation over time
- Older memories drift toward stable anchors

## 🔄 Multi-Method Retrieval

### 1. RAG (Retrieval-Augmented Generation)
- **Vector Similarity**: Cosine similarity on embeddings
- **Top-K Selection**: Retrieve most similar memories
- **Context Window**: Fit within LLM token limits

### 2. Vector Embeddings (pgvector)
- **Fast Similarity Search**: PostgreSQL pgvector extension
- **Approximate Nearest Neighbors**: HNSW indexing
- **Scoped Retrieval**: User/org/agent isolation

### 3. Hash Sphere Proximity
- **3D Distance**: Euclidean distance in semantic space
- **Proximity Score**: `score = e^(-distance)`
- **Spatial Clustering**: Nearby memories in 3D space

### 4. Resonance Filtering
- **Hash Similarity**: Hamming distance on hash strings
- **Resonance Threshold**: Filter by minimum resonance score
- **Energy-based Ranking**: Sort by anchor energy

### 5. Hybrid Ranking
Combines all methods with weighted scores:
```
hybrid_score = 
  w1 × rag_score +
  w2 × resonance_score +
  w3 × proximity_score +
  w4 × recency_score +
  w5 × anchor_energy
```

## 📊 Visualization

### 3D Sphere View
- **Memory Nodes**: Colored spheres positioned by xyz coordinates
- **Size**: Scaled by importance score
- **Color**: By memory type (chat, code, function, etc.)
- **Connections**: Lines between related memories (resonance > 0.6)

### Spin Vector Arrows
- **Direction**: Shows semantic rotation
- **Length**: Proportional to spin magnitude
- **Color**: Indicates spin type (topic/emotion/complexity)

### Cluster Regions
- **Boundaries**: Semi-transparent spheres around clusters
- **Centroids**: Larger markers at cluster centers
- **Color-coded**: Each cluster has unique color

### Energy Fields
- **Gradient Visualization**: Color intensity by anchor energy
- **Attraction Lines**: Curves showing drift toward anchors
- **Pulsing Effect**: Animated for high-energy memories

### Semantic Overlays
**Color Modes**:
- **Type**: Color by memory type (default)
- **Meaning**: Gradient from low to high meaning score
- **Intensity**: Heat map of emotional intensity
- **Sentiment**: Red (negative) → Yellow (neutral) → Green (positive)

### Timeline View
- **Chronological**: Memories grouped by date
- **Temporal Patterns**: Identify memory creation trends
- **Decay Visualization**: Older memories fade/shrink

### Hyperspherical View
- **Latitude/Longitude**: Alternative coordinate system
- **Radius Scaling**: Distance from origin
- **Spherical Grid**: Overlay showing coordinate lines

## 🔐 Security & Privacy

### Encryption
- **Algorithm**: AES-256-GCM
- **Key Size**: 256 bits
- **Per-User Keys**: Each user has unique encryption key
- **At-Rest Encryption**: All memories encrypted in database

### Isolation
- **User Universes**: Each user has isolated memory space
- **Org Scoping**: Organization-level memory sharing
- **Agent Overlays**: Agent-specific memory layers

## 📈 Performance

### Caching
- **Embedding Cache**: LRU cache for query embeddings
- **Semantic Cache**: Full query result caching
- **Cache Hit Rate**: Typically 60-80% for repeated queries

### Indexing
- **pgvector HNSW**: Fast approximate nearest neighbor search
- **B-tree Indexes**: On user_id, created_at, hash fields
- **Partial Indexes**: For active (non-archived) memories

### Metrics
- **Retrieval Time**: < 100ms for typical queries
- **Storage**: ~2KB per memory (including embeddings)
- **Throughput**: 1000+ memories/second ingestion rate

## 🎓 Use Cases

### Personal Knowledge Base
- Store chat conversations, notes, documents
- Retrieve relevant context for new queries
- Build long-term memory for AI assistants

### Code Memory
- Remember code snippets, functions, patterns
- Retrieve similar code for new tasks
- Track code evolution over time

### Research Assistant
- Store papers, articles, research notes
- Find related research by semantic similarity
- Build knowledge graphs from memories

### Customer Support
- Remember customer interactions
- Retrieve relevant past conversations
- Personalize responses based on history

## 🔬 Technical Details

### Database Schema
```sql
CREATE TABLE memory_records (
  id UUID PRIMARY KEY,
  user_id UUID,
  org_id UUID,
  source VARCHAR,
  content TEXT ENCRYPTED,
  
  -- Layer 2: Hashes
  hash VARCHAR(64),
  meaning_hash VARCHAR(20),
  energy_hash VARCHAR(8),
  spin_hash VARCHAR(8),
  
  -- Layer 3: Universe ID
  universe_id VARCHAR(64),
  
  -- Layer 5: Coordinates
  xyz_x FLOAT,
  xyz_y FLOAT,
  xyz_z FLOAT,
  sphere_r FLOAT,
  sphere_phi FLOAT,
  sphere_theta FLOAT,
  
  -- Layer 6: Resonance
  resonance_score FLOAT,
  normalized_resonance FLOAT,
  anchor_energy FLOAT,
  
  -- Spin & Semantic
  spin_x FLOAT,
  spin_y FLOAT,
  spin_z FLOAT,
  spin_magnitude FLOAT,
  meaning_score FLOAT,
  intensity_score FLOAT,
  sentiment_score FLOAT,
  
  -- Clustering
  cluster_name VARCHAR,
  
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);
```

### API Endpoints

**Ingest Memory**:
```http
POST /api/v1/memory/ingest
{
  "content": "Memory text",
  "source": "chat",
  "generate_embedding": true
}
```

**Search Memories**:
```http
POST /api/v1/memory/search
{
  "query": "Search query",
  "limit": 10,
  "retrieval_mode": "hybrid"
}
```

**Hash Sphere Extract**:
```http
POST /api/v1/memory/hash-sphere/extract
{
  "query": "Query text",
  "use_anchors": true,
  "use_proximity": true,
  "use_resonance": true,
  "apply_magnetic_pull": true
}
```

**Get Anchors**:
```http
GET /api/v1/memory/hash-sphere/anchors?limit=1000
```

## 📚 References

- **Resonance Hashing**: Based on cryptographic hash functions and semantic similarity
- **Hyperspherical Coordinates**: Standard spherical coordinate system
- **PCA Dimensionality Reduction**: Scikit-learn implementation
- **Vector Embeddings**: OpenAI text-embedding-3-large (1536 dimensions)
- **pgvector**: PostgreSQL extension for vector similarity search

## 🚀 Future Enhancements

- **Multi-modal Memories**: Images, audio, video embeddings
- **Temporal Dynamics**: Time-based memory evolution
- **Cross-user Clustering**: Shared semantic spaces
- **Quantum Resistance**: Post-quantum cryptography
- **Federated Learning**: Privacy-preserving memory sharing

---

**Experience the future of AI memory at [resonantgenesis.xyz/resonant-memory](https://resonantgenesis.xyz/resonant-memory)**
