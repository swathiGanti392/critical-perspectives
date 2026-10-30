# Critical Perspectives Architecture

## Overview

This document details the technical architecture for a multi-agent system where AI agents with different perspectives engage in structured conversations and debates.

## System Architecture

### High-Level Block Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                      User Interface                          │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Web UI      │  │  API         │  │  Topic       │     │
│  │  Dashboard   │  │  Endpoints   │  │  Suggestion  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
┌───────────────────────────┼─────────────────────────────────┐
│                      Orchestration Layer                     │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Conversation│  │  Agent       │  │  Moderator    │     │
│  │  Manager     │  │  Manager     │  │  Engine       │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│         │                 │                 │               │
│         └─────────────────┼─────────────────┘               │
│                           │                                 │
└───────────────────────────┼─────────────────────────────────┘
                            │
┌───────────────────────────┼─────────────────────────────────┐
│                      Agent Layer                              │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Skeptic     │  │  Empiricist  │  │  Pragmatist   │     │
│  │  Agent       │  │  Agent       │  │  Agent       │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Ethicist    │  │  Historian   │  │  Futurist     │     │
│  │  Agent       │  │  Agent       │  │  Agent       │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐                      │
│  │  Artist      │  │  Advocate    │                      │
│  │  Agent       │  │  Agent       │                      │
│  └──────────────┘  └──────────────┘                      │
└─────────────────────────────────────────────────────────────┘
                            │
┌───────────────────────────┼─────────────────────────────────┐
│                      AI Model Layer                           │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  LLM         │  │  Knowledge   │  │  Context     │     │
│  │  Backend     │  │  Base        │  │  Manager     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
┌───────────────────────────┼─────────────────────────────────┐
│                      Data Layer                               │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Conversation│  │  Agent       │  │  Topic       │     │
│  │  Database    │  │  Profiles    │  │  Database    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

## Agent Architecture

### Base Agent Class

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Optional
from dataclasses import dataclass

@dataclass
class AgentProfile:
    name: str
    role: str
    perspective: str
    personality_traits: Dict[str, float]
    knowledge_domains: List[str]
    communication_style: str
    biases: List[str]

class CriticAgent(ABC):
    def __init__(self, profile: AgentProfile, llm_backend):
        self.profile = profile
        self.llm = llm_backend
        self.memory = ConversationMemory()
        self.beliefs = BeliefSystem()
        self.preferences = PreferenceSystem()

    @abstractmethod
    def analyze_statement(self, statement: str) -> Dict:
        """Analyze a statement from this agent's perspective"""
        pass

    @abstractmethod
    def generate_response(self, context: Dict) -> str:
        """Generate a response based on context"""
        pass

    @abstractmethod
    def reflect(self, conversation: Conversation) -> None:
        """Reflect on conversation and update beliefs"""
        pass

    def respond(self, conversation: Conversation) -> str:
        """Main response method"""
        context = self.build_context(conversation)
        analysis = self.analyze_context(context)
        response = self.generate_response(analysis)
        self.memory.add(response, context)
        return response

    def build_context(self, conversation: Conversation) -> Dict:
        """Build context from conversation history"""
        return {
            'topic': conversation.topic,
            'history': conversation.get_recent_messages(n=5),
            'current_round': conversation.round,
            'other_agents': conversation.get_other_agents(self),
            'moderator_notes': conversation.moderator_notes
        }
```

### Specific Agent Implementations

#### Skeptic Agent
```python
class SkepticAgent(CriticAgent):
    def __init__(self, llm_backend):
        profile = AgentProfile(
            name="Skeptic",
            role="Critical Questioner",
            perspective="Challenges assumptions and demands evidence",
            personality_traits={
                'skeptical': 0.9,
                'analytical': 0.8,
                'demanding': 0.7
            },
            knowledge_domains=['logic', 'critical_thinking', 'scientific_method'],
            communication_style='direct',
            biases=['confirmation_bias_awareness']
        )
        super().__init__(profile, llm_backend)

    def analyze_statement(self, statement: str) -> Dict:
        """Analyze statement for assumptions and evidence"""
        analysis = {
            'assumptions': self.identify_assumptions(statement),
            'evidence_quality': self.evaluate_evidence(statement),
            'logical_fallacies': self.detect_fallacies(statement),
            'claims_needing_support': self.identify_unsupported_claims(statement)
        }
        return analysis

    def generate_response(self, context: Dict) -> str:
        """Generate skeptical response"""
        prompt = f"""
        As a skeptic, respond to the following discussion:
        Topic: {context['topic']}
        Previous statements: {context['history']}

        Your response should:
        - Question assumptions
        - Demand evidence
        - Identify logical fallacies
        - Challenge weak arguments
        """
        return self.llm.generate(prompt, self.profile)

    def reflect(self, conversation: Conversation) -> None:
        """Reflect on conversation and update beliefs"""
        valid_evidence = self.identify_valid_evidence(conversation)
        for evidence in valid_evidence:
            self.beliefs.update(evidence)
```

#### Empiricist Agent
```python
class EmpiricistAgent(CriticAgent):
    def __init__(self, llm_backend):
        profile = AgentProfile(
            name="Empiricist",
            role="Data-Driven Analyst",
            perspective="Focuses on empirical evidence and scientific rigor",
            personality_traits={
                'analytical': 0.9,
                'methodical': 0.8,
                'evidence_focused': 0.9
            },
            knowledge_domains=['statistics', 'research_methods', 'data_science'],
            communication_style='technical',
            biases=['prefer_quantitative_evidence']
        )
        super().__init__(profile, llm_backend)

    def analyze_statement(self, statement: str) -> Dict:
        """Analyze statement for empirical validity"""
        analysis = {
            'evidence_cited': self.extract_evidence(statement),
            'methodology': self.evaluate_methodology(statement),
            'statistical_significance': self.check_statistics(statement),
            'replicability': self.assess_replicability(statement)
        }
        return analysis

    def generate_response(self, context: Dict) -> str:
        """Generate empiricist response"""
        prompt = f"""
        As an empiricist, respond to the following discussion:
        Topic: {context['topic']}
        Previous statements: {context['history']}

        Your response should:
        - Cite empirical evidence
        - Emphasize methodology
        - Demand statistical rigor
        - Prioritize peer-reviewed sources
        """
        return self.llm.generate(prompt, self.profile)
```

#### Ethicist Agent
```python
class EthicistAgent(CriticAgent):
    def __init__(self, llm_backend):
        profile = AgentProfile(
            name="Ethicist",
            role="Moral Philosopher",
            perspective="Considers ethical implications and moral frameworks",
            personality_traits={
                'principled': 0.9,
                'empathetic': 0.8,
                'value_focused': 0.9
            },
            knowledge_domains=['ethics', 'philosophy', 'moral_theory'],
            communication_style='reflective',
            biases=['utilitarian_bias']
        )
        super().__init__(profile, llm_backend)

    def analyze_statement(self, statement: str) -> Dict:
        """Analyze statement for ethical implications"""
        analysis = {
            'ethical_frameworks': self.apply_ethical_frameworks(statement),
            'stakeholders': self.identify_stakeholders(statement),
            'rights_violations': self.check_rights(statement),
            'justice_considerations': self.evaluate_justice(statement)
        }
        return analysis

    def generate_response(self, context: Dict) -> str:
        """Generate ethicist response"""
        prompt = f"""
        As an ethicist, respond to the following discussion:
        Topic: {context['topic']}
        Previous statements: {context['history']}

        Your response should:
        - Apply ethical frameworks
        - Consider stakeholders
        - Evaluate justice and fairness
        - Examine moral implications
        """
        return self.llm.generate(prompt, self.profile)
```

## Conversation Management

### Conversation Class

```python
class Conversation:
    def __init__(self, topic: str, context: str):
        self.topic = topic
        self.context = context
        self.messages = []
        self.round = 0
        self.phase = 'introduction'
        self.participants = []
        self.moderator_notes = []
        self.synthesis = None

    def add_message(self, agent: CriticAgent, message: str):
        """Add a message to the conversation"""
        self.messages.append({
            'agent': agent.profile.name,
            'role': agent.profile.role,
            'message': message,
            'timestamp': datetime.now(),
            'round': self.round,
            'phase': self.phase
        })

    def get_recent_messages(self, n: int = 5) -> List[Dict]:
        """Get the most recent messages"""
        return self.messages[-n:]

    def get_other_agents(self, current_agent: CriticAgent) -> List[str]:
        """Get names of other participating agents"""
        return [a for a in self.participants if a != current_agent.profile.name]

    def advance_round(self):
        """Advance to the next round"""
        self.round += 1
        if self.round == 1:
            self.phase = 'initial_positions'
        elif self.round == 3:
            self.phase = 'cross_examination'
        elif self.round == 6:
            self.phase = 'synthesis'
        elif self.round == 7:
            self.phase = 'reflection'

    def is_complete(self) -> bool:
        """Check if conversation is complete"""
        return self.phase == 'reflection' and self.round >= 8
```

### Conversation Manager

```python
class ConversationManager:
    def __init__(self, agent_manager: AgentManager, moderator: Moderator):
        self.agent_manager = agent_manager
        self.moderator = moderator
        self.active_conversations = {}
        self.conversation_history = []

    def start_conversation(self, topic: str, context: str,
                          agent_names: List[str]) -> Conversation:
        """Start a new conversation"""
        conversation = Conversation(topic, context)

        # Register participating agents
        for name in agent_names:
            agent = self.agent_manager.get_agent(name)
            conversation.participants.append(name)

        # Initialize with opening statements
        self.moderator.initialize(conversation)

        self.active_conversations[conversation.id] = conversation
        return conversation

    def continue_conversation(self, conversation_id: str) -> bool:
        """Continue an existing conversation"""
        conversation = self.active_conversations[conversation_id]

        if conversation.is_complete():
            self.moderator.synthesize(conversation)
            self.archive_conversation(conversation)
            return False

        # Determine next speaker
        next_agent = self.moderator.next_speaker(conversation)
        agent = self.agent_manager.get_agent(next_agent)

        # Generate response
        response = agent.respond(conversation)
        conversation.add_message(agent, response)

        # Moderate the response
        self.moderator.moderate(conversation, response)

        # Advance round if needed
        conversation.advance_round()

        return True

    def archive_conversation(self, conversation: Conversation):
        """Archive a completed conversation"""
        self.conversation_history.append(conversation)
        del self.active_conversations[conversation.id]
```

## Moderator System

### Moderator Class

```python
class Moderator:
    def __init__(self):
        self.rules = ConversationRules()
        self.quality_assessor = QualityAssessor()
        self.synthesizer = ConversationSynthesizer()

    def initialize(self, conversation: Conversation):
        """Initialize conversation with opening statements"""
        conversation.phase = 'introduction'

        for agent_name in conversation.participants:
            agent = self.agent_manager.get_agent(agent_name)
            opening = agent.generate_opening(conversation.topic)
            conversation.add_message(agent, opening)

    def next_speaker(self, conversation: Conversation) -> str:
        """Determine who should speak next"""
        if conversation.phase == 'introduction':
            # Random order for initial statements
            return self.select_random_agent(conversation)
        else:
            # Round-robin or strategic selection
            return self.select_next_in_rotation(conversation)

    def moderate(self, conversation: Conversation, response: str):
        """Moderate the response"""
        quality = self.quality_assessor.assess(response)
        relevance = self.quality_assessor.assess_relevance(
            response, conversation.topic
        )

        if quality < 0.7:
            self.request_clarification(conversation)
        if relevance < 0.7:
            self.redirect_topic(conversation)

        # Add moderator notes
        conversation.moderator_notes.append({
            'quality': quality,
            'relevance': relevance,
            'timestamp': datetime.now()
        })

    def synthesize(self, conversation: Conversation):
        """Synthesize the conversation"""
        synthesis = self.synthesizer.synthesize(conversation)
        conversation.synthesis = synthesis
        return synthesis
```

### Quality Assessment

```python
class QualityAssessor:
    def __init__(self):
        self.criteria = {
            'coherence': 0.3,
            'evidence': 0.25,
            'novelty': 0.2,
            'relevance': 0.15,
            'clarity': 0.1
        }

    def assess(self, response: str) -> float:
        """Assess overall quality of response"""
        scores = {
            'coherence': self.assess_coherence(response),
            'evidence': self.assess_evidence(response),
            'novelty': self.assess_novelty(response),
            'relevance': self.assess_relevance(response),
            'clarity': self.assess_clarity(response)
        }

        weighted_score = sum(
            scores[criterion] * weight
            for criterion, weight in self.criteria.items()
        )

        return weighted_score

    def assess_coherence(self, response: str) -> float:
        """Assess coherence of response"""
        # Use NLP to assess coherence
        sentences = self.split_sentences(response)
        coherence_score = self.calculate_coherence(sentences)
        return coherence_score

    def assess_evidence(self, response: str) -> float:
        """Assess quality of evidence provided"""
        evidence_keywords = ['study', 'research', 'data', 'evidence', 'proven']
        evidence_count = sum(1 for word in evidence_keywords if word in response.lower())
        return min(evidence_count / 3, 1.0)
```

### Synthesis Engine

```python
class ConversationSynthesizer:
    def __init__(self):
        self.agreement_detector = AgreementDetector()
        self.clustering = TopicClustering()

    def synthesize(self, conversation: Conversation) -> Dict:
        """Synthesize conversation into key insights"""
        synthesis = {
            'agreements': self.find_agreements(conversation),
            'disagreements': self.find_disagreements(conversation),
            'key_insights': self.extract_insights(conversation),
            'open_questions': self.identify_open_questions(conversation),
            'recommendations': self.generate_recommendations(conversation),
            'perspective_summary': self.summarize_perspectives(conversation)
        }
        return synthesis

    def find_agreements(self, conversation: Conversation) -> List[str]:
        """Find areas of agreement between agents"""
        agreements = []
        messages = conversation.messages

        # Compare pairs of messages for agreement
        for i in range(len(messages)):
            for j in range(i+1, len(messages)):
                if self.agreement_detector.detect_agreement(
                    messages[i]['message'],
                    messages[j]['message']
                ):
                    agreements.append({
                        'agents': [messages[i]['agent'], messages[j]['agent']],
                        'topic': self.extract_topic(messages[i]['message'])
                    })

        return agreements

    def find_disagreements(self, conversation: Conversation) -> List[str]:
        """Find areas of disagreement between agents"""
        disagreements = []
        messages = conversation.messages

        for i in range(len(messages)):
            for j in range(i+1, len(messages)):
                if self.agreement_detector.detect_disagreement(
                    messages[i]['message'],
                    messages[j]['message']
                ):
                    disagreements.append({
                        'agents': [messages[i]['agent'], messages[j]['agent']],
                        'topic': self.extract_topic(messages[i]['message'])
                    })

        return disagreements
```

## LLM Backend Integration

### LLM Backend Interface

```python
class LLMBackend:
    def __init__(self, model_name: str, api_key: str):
        self.model_name = model_name
        self.api_key = api_key
        self.client = self.initialize_client()

    def generate(self, prompt: str, profile: AgentProfile) -> str:
        """Generate response from LLM with agent personality"""
        system_prompt = self.build_system_prompt(profile)
        full_prompt = f"{system_prompt}\n\n{prompt}"

        response = self.client.chat.completions.create(
            model=self.model_name,
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": prompt}
            ],
            temperature=self.get_temperature(profile),
            max_tokens=1000
        )

        return response.choices[0].message.content

    def build_system_prompt(self, profile: AgentProfile) -> str:
        """Build system prompt for agent"""
        return f"""
        You are {profile.name}, a {profile.role}.
        Your perspective: {profile.perspective}
        Your personality traits: {profile.personality_traits}
        Your knowledge domains: {profile.knowledge_domains}
        Your communication style: {profile.communication_style}

        Always respond in character as {profile.name}.
        Maintain your perspective and personality throughout the conversation.
        """

    def get_temperature(self, profile: AgentProfile) -> float:
        """Get appropriate temperature for agent"""
        if profile.communication_style == 'direct':
            return 0.7
        elif profile.communication_style == 'reflective':
            return 0.8
        elif profile.communication_style == 'technical':
            return 0.5
        else:
            return 0.7
```

## Data Models

### Conversation Schema
```json
{
  "conversation_id": "conv_001",
  "topic": "AI and Employment",
  "context": "Discussing the impact of AI on job markets",
  "created_at": "2024-01-15T10:00:00Z",
  "participants": ["Skeptic", "Empiricist", "Pragmatist", "Ethicist"],
  "messages": [
    {
      "agent": "Skeptic",
      "role": "Critical Questioner",
      "message": "We need to question the assumption that AI will automatically create more jobs than it destroys...",
      "timestamp": "2024-01-15T10:01:00Z",
      "round": 0,
      "phase": "introduction"
    }
  ],
  "synthesis": {
    "agreements": [...],
    "disagreements": [...],
    "key_insights": [...],
    "recommendations": [...]
  }
}
```

### Agent Profile Schema
```json
{
  "agent_id": "agent_001",
  "name": "Skeptic",
  "role": "Critical Questioner",
  "perspective": "Challenges assumptions and demands evidence",
  "personality_traits": {
    "skeptical": 0.9,
    "analytical": 0.8,
    "demanding": 0.7
  },
  "knowledge_domains": ["logic", "critical_thinking", "scientific_method"],
  "communication_style": "direct",
  "biases": ["confirmation_bias_awareness"]
}
```

## API Design

### REST Endpoints

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class ConversationRequest(BaseModel):
    topic: str
    context: str
    agents: List[str]
    max_rounds: int = 8

@app.post("/api/conversations/start")
async def start_conversation(request: ConversationRequest):
    """Start a new conversation"""
    conversation = conversation_manager.start_conversation(
        topic=request.topic,
        context=request.context,
        agent_names=request.agents
    )
    return {
        "conversation_id": conversation.id,
        "topic": conversation.topic,
        "participants": conversation.participants
    }

@app.post("/api/conversations/{conversation_id}/continue")
async def continue_conversation(conversation_id: str):
    """Continue an existing conversation"""
    success = conversation_manager.continue_conversation(conversation_id)
    if not success:
        conversation = conversation_manager.get_archived(conversation_id)
        return {
            "status": "complete",
            "synthesis": conversation.synthesis
        }
    return {"status": "continued"}

@app.get("/api/conversations/{conversation_id}")
async def get_conversation(conversation_id: str):
    """Get conversation details"""
    conversation = conversation_manager.get_conversation(conversation_id)
    return {
        "topic": conversation.topic,
        "messages": conversation.messages,
        "synthesis": conversation.synthesis
    }

@app.get("/api/agents")
async def list_agents():
    """List available agents"""
    agents = agent_manager.list_agents()
    return {
        "agents": [
            {
                "name": agent.profile.name,
                "role": agent.profile.role,
                "perspective": agent.profile.perspective
            }
            for agent in agents
        ]
    }
```

## Performance Considerations

### Response Time Optimization
- Parallel agent processing
- Response caching
- Streaming responses
- Load balancing

### Scalability
- Horizontal scaling of LLM backends
- Database sharding for conversations
- Caching layer for agent profiles
- Queue system for conversation processing

### Cost Management
- Token usage optimization
- Model selection based on complexity
- Response length limits
- Conversation caching

## Monitoring & Analytics

### Metrics Collection
```python
class MetricsCollector:
    def __init__(self):
        self.metrics = {
            'conversations_started': 0,
            'conversations_completed': 0,
            'average_rounds': 0,
            'agent_usage': {},
            'quality_scores': []
        }

    def record_conversation_start(self, agents: List[str]):
        """Record conversation start"""
        self.metrics['conversations_started'] += 1
        for agent in agents:
            self.metrics['agent_usage'][agent] = \
                self.metrics['agent_usage'].get(agent, 0) + 1

    def record_conversation_complete(self, rounds: int, quality: float):
        """Record conversation completion"""
        self.metrics['conversations_completed'] += 1
        self.metrics['average_rounds'] = \
            (self.metrics['average_rounds'] * (self.metrics['conversations_completed'] - 1) + rounds) / \
            self.metrics['conversations_completed']
        self.metrics['quality_scores'].append(quality)
```

## Conclusion

The Critical Perspectives architecture provides a comprehensive framework for multi-agent conversations where AI agents with different perspectives engage in structured debates. By implementing distinct agent personalities, conversation protocols, and quality assessment mechanisms, the system enables nuanced exploration of complex topics from multiple viewpoints.

Success requires collaboration between AI researchers, philosophers, UX designers, and domain experts to ensure agent authenticity, conversation quality, and user value.
