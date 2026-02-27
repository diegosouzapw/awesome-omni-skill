---
name: aegis-architect
description: Enhanced architecture guidance for voice-first Brazilian fintech applications. Use when designing voice interfaces, implementing PIX/Boletos, optimizing financial systems, or making technology stack decisions for Brazilian market applications. Integrates with docs/ content, MCP tools for Brazilian market research, enhanced validation scripts, and comprehensive Brazilian compliance patterns.
license: MIT
metadata:
  version: "3.0.0"
  author: "AegisWallet Development Team"
  category: "architecture"
  last-updated: "2025-11-27"
  domain: "brazilian-fintech"
  expertise: ["voice-first", "brazilian-financial", "lgpd-compliance", "performance-optimization", "hono-rpc"]
  links:
  references: [
    {
      "title": "AegisWallet Architecture Documentation",
      "url": "docs/architecture.md"
    },
    {
      "title": "Technology Stack Specification", 
      "url": "docs/architecture/tech-stack.md"
    },
    {
      "Performance Patterns",
      "url": "docs/architecture/hono-rpc-patterns.md"
    },
    {
      "Voice Interface Patterns",
      "url": "docs/architecture/voice-interface-patterns.md"
    },
    {
      "AI Chat Architecture",
      "url": "docs/architecture/ai-chat-architecture.md"
    }
  ]
---

# Enhanced AegisWallet Architecture Skill v3.0 - Brazilian Fintech Voice-First Assistant

## About This Skill

This skill provides **enhanced architecture guidance** for voice-first Brazilian fintech applications, now **specialized in real-world problem resolution** with comprehensive documentation integration, MCP tools, and **practical implementation examples**. Use when designing voice interfaces, implementing PIX/Boletos, optimizing financial systems, or making technology stack decisions for Brazilian market applications.

### What's New in v3.0

- **🔧 Practical Problem Resolution**: Enhanced with actual code examples and working scripts
- **📚 Documentation Integration**: Full sync with `docs/architecture/` reference materials  
- **🇧🇷 Brazilian Compliance**: Deep LGPD, PIX, and Open Banking implementation patterns
- **⚡ Performance Optimization**: Sub-200ms voice response patterns and benchmarks
- **🎯 Enhanced Validation**: Comprehensive scripts for architecture compliance
- **🚀 Emergency Recovery**: Diagnostic and recovery scripts for production issues

## When to Use This Skill

**Use this skill when:**
- **Voice Interface Issues**: "Voice commands are slow/unresponsive" → Use performance optimization patterns
- **Brazilian Compliance**: "Need to implement LGPD/PIX" → Use comprehensive implementation guides
- **Architecture Decisions**: "How should I structure X?" → Consult integrated documentation
- **Performance Problems**: "System is slow/lagging" → Use diagnostic scripts and patterns
- **Security Implementation**: "Need financial-grade security" → Use security patterns
- **Real-time Sync Issues**: "Data not updating properly" → Use real-time patterns
- **Development Workflow**: "How to implement X feature?" → Use templates and examples

### Enhanced Troubleshooting Capabilities

#### Voice Performance Issues
```
Quando: "A autenticação está demorando 2+ segundos para responder comandos de voz"

Resposta: Use `scripts/performance_audit.py --directory .` e verifique:
- VITE_ENABLE_VOICE_REASONING está desabilitado
- Configurações de microfone e silêncio
- Sobrecarga de processamento de voz
- Otimizações de cache para comandos comuns
```

#### Architecture Consulting
```
Quando: "Preciso implementar PIX com Hono RPC"

Resposta: Consulte `docs/architecture/hono-rpc-patterns.md` para padrões detalhados:
- Endpoint structure: `/api/v1/pix/transfer`
- Validation: zValidator + authMiddleware  
- Rate limiting para segurança PIX
- Autenticação dupla (senha + biometria)
- Tratamento de erro padronizado
```

#### System Recovery
```
Quando: "Sistema não está respondendo ou deu erro"

Resposta: Use `scripts/emergency-recovery.sh` para diagnóstico completo:
- Verificar API status (curl http://localhost:3000/health)
- Verificar Hono server status
- Verificar Supabase service status
- Executar recovery automatizado
```

## 📚 Enhanced Documentation Integration

This skill now provides **complete integration** with existing AegisWallet documentation:

### Primary Documentation Sources
- **`docs/architecture.md`**: Complete architecture overview and system design
- **`docs/architecture/tech-stack.md`**: Detailed technology specifications and patterns
- **`docs/architecture/hono-rpc-patterns.md`**: API design patterns with validation and security
- **`docs/architecture/voice-interface-patterns.md`**: Voice-first interaction patterns
- **`docs/architecture/frontend-spec.md`**: Frontend architecture and component patterns
- **`docs/architecture/ai-chat-architecture.md`**: AI-powered conversational interfaces
- **`docs/LGPD_COMPLIANCE_TESTING_PENDING.md`**: Brazilian compliance implementation status
- **`docs/VERCEL_DEPLOYMENT_GUIDE.md`**: Deployment and production patterns

### Quick Reference Documentation
```bash
# Architecture fundamentals
docs/architecture.md                    # System overview and design principles

# Technology stack specifics  
docs/architecture/tech-stack.md         # Bun + Hono + React 19 + Supabase patterns
docs/architecture/hono-rpc-patterns.md   # API endpoint design and security

# Voice interface implementation
docs/architecture/voice-interface-patterns.md  # Voice-first UI patterns
docs/architecture/ai-chat-architecture.md       # AI conversation patterns

# Frontend development
docs/architecture/frontend-spec.md       # React 19 + TanStack patterns
docs/architecture/frontend-architecture.md # Component architecture

# Compliance and deployment
docs/LGPD_COMPLIANCE_TESTING_PENDING.md  # LGPD implementation checklist
docs/VERCEL_DEPLOYMENT_GUIDE.md          # Production deployment guide
```

### Enhanced MCP Integration
- **Context7**: Access latest framework documentation and best practices
- **Tavily**: Research Brazilian fintech regulations and market trends  
- **Serena**: Semantic code analysis and architecture validation
- **Apex Researcher**: Multi-source validation for Brazilian compliance (≥95% accuracy)

## 🛠️ Enhanced Validation & Diagnostic Tools

### Architecture Compliance Validation
```bash
# Complete architecture validation
python scripts/validate_architecture.py --directory . --output json

# Performance audit with voice-specific metrics
python scripts/performance_audit.py --directory . --benchmark voice

# Brazilian compliance validation
python scripts/brazilian_compliance_validator.py --check pix,lgpd,openbanking

# Emergency system recovery
./scripts/emergency-recovery.sh --full-scan
```

### Performance Monitoring Scripts
```bash
# Voice response time analysis
python scripts/voice_performance_analyzer.py --target-ms 200

# Memory leak detection for voice components
python scripts/memory_leak_detector.py --focus voice-hooks

# Database query optimization
python scripts/database_performance_optimizer.py --analyze-financial-queries
```

### Brazilian Compliance Testing
```bash
# LGPD compliance validation
python scripts/lgpd_compliance_test.py --comprehensive

# PIX transaction flow testing
python scripts/pix_transaction_validator.py --test-full-flow

# Open Banking integration testing
python scripts/openbanking_compliance_test.py --validate-spec-3.1
```

### 🎯 Enhanced NLU System
- **Brazilian Portuguese Specialization**: 6 major regional variations with 50+ slang terms
- **Hit/Miss Tracking**: Real-time analytics with learning feedback loops
- **Error Recovery**: Multi-strategy recovery with 80%+ success rate
- **Context Processing**: Multi-turn conversation context with user preferences
- **Performance Monitoring**: Sub-200ms processing with real-time health monitoring

### 🚀 Voice Performance Optimizations
- **Processing Time**: Reduced from 3-5s to ≤2s for voice commands
- **Memory Management**: Fixed memory leaks from intervals/timeouts
- **Voice Activity Detection**: Real-time speech detection with <20ms latency
- **Timeout Optimization**: All major timeouts reduced by 60-80%
- **Performance Testing**: Comprehensive test suite for validation

### 📊 Analytics & Learning
- **Regional Analytics**: Accuracy tracking by Brazilian region
- **Pattern Evolution**: Learning from user corrections and adaptations
- **System Health**: Comprehensive monitoring with automated alerting
- **Performance Reports**: Detailed insights and optimization suggestions

## Essential Voice Commands

This skill provides guidance for implementing these 6 essential Brazilian voice commands:

1. **"Como está meu saldo?"** - Balance inquiry with regional variations
2. **"Quanto posso gastar esse mês?"** - Spending capacity check
3. **"Tem algum boleto programado?"** - Bill inquiry (varies by region)
4. **"Tem algum recebimento programado?"** - Income inquiry
5. **"Como ficará meu saldo no final do mês?"** - Balance projection
6. **"Faz uma transferência para..."** - Money transfer command

## Performance Standards

### Updated Targets (Post-Optimization)
- **Voice Processing**: ≤200ms (reduced from 500ms)
- **Total Response**: ≤500ms (reduced from 1000ms)
- **Accuracy**: 90%+ for Brazilian financial commands
- **Error Recovery**: 80%+ success rate
- **System Uptime**: 99.9%
- **Regional Accuracy**: 85-95% depending on region

### Regional Performance Goals
- **São Paulo (SP)**: 95% accuracy - Financial capital variations
- **Rio de Janeiro (RJ)**: 92% accuracy - Carioca expressions
- **Nordeste (NE)**: 88% accuracy - Regional slang and patterns
- **Sul**: 90% accuracy - Southern expressions and terminology
- **Norte**: 85% accuracy - Northern regional variations
- **Centro-Oeste (CO)**: 87% accuracy - Central-west patterns

## Core Architecture Expertise

### Technology Stack Mastery

#### Runtime & Framework
- **Bun**: Package management and runtime (3-5x faster than npm)
- **Hono**: Edge-first API framework for sub-150ms response times
- **React 19**: Voice interface with concurrent features and hooks
- **TypeScript 5.9.3**: End-to-end type safety and strict mode

#### Database & Infrastructure
- **Supabase**: PostgreSQL + Auth + Realtime + Storage + RLS
- **tRPC v11**: Type-safe API procedures with automatic client generation
- **TanStack Query v5**: Real-time financial data synchronization
- **TanStack Router v5**: File-based routing with full TypeScript support

#### Frontend & UI
- **Tailwind CSS 4.x**: Utility-first styling with Brazilian localization
- **shadcn/ui**: WCAG 2.1 AA+ compliant accessible components
- **Motion (Framer Motion)**: Voice interaction animations
- **React Hook Form + Zod**: Type-safe form validation

### Brazilian Financial System Integration

#### PIX Implementation
```typescript
// PIX transaction architecture
interface PIXTransaction {
  id: string;
  userId: string;
  amount: Money;
  pixKey: PIXKey;
  description: string;
  status: 'pending' | 'processing' | 'completed' | 'failed';
  transactionId: string;
  endToEndId: string;
  createdAt: DateTime;
}

// PIX key management
interface PIXKey {
  id: string;
  userId: string;
  keyType: 'email' | 'cpf' | 'cnpj' | 'phone' | 'random';
  keyValue: string;
  label?: string;
  isFavorite: boolean;
  isActive: boolean;
}
```

#### Boleto Processing
```typescript
// Boleto payment workflow
interface BoletoPayment {
  id: string;
  barcode: string;
  amount: Money;
  dueDate: DateTime;
  status: 'pending' | 'paid' | 'expired';
  metadata: BoletoMetadata;
}
```

#### Enhanced Brazilian Portuguese Patterns with Regional Variations
```typescript
// Complete regional pattern system for Brazilian Portuguese
interface BrazilianPortuguesePatterns {
  // São Paulo (SP) - Financial capital variations
  saoPaulo: {
    greetings: ["oi", "eai", "beleza", "meu bem"];
    financial: ["grana", "coiso", "boleta", "cê tá ligado?", "tipo assim"];
    payments: ["faz um pix pra mim", "me manda o pix", "transfere na conta"];
    questions: ["quanto tá?", "me fala o valor", "qualé o preço?"];
    expressions: ["rolê", "parça", "mano", "vamos nessa"];
  };
  
  // Rio de Janeiro (RJ) - Carioca variations
  rioDeJaneiro: {
    greetings: ["eai", "beleza", "firmesa?", "demais"];
    financial: ["grana", "dinhe", "ferinha", "caraca!"];
    payments: ["me ajuda com o pix", "faz a transferência", "me passa o pix"];
    questions: ["qualé o valor?", "quanto é?", "me fala quanto custa"];
    expressions: ["maneiro", "sinistro", "você é brabo", "massa"];
  };
  
  // Nordeste (NE) - Regional variations
  nordeste: {
    greetings: ["oi", "eai", "bão?", "oxente"];
    financial: ["bão", "grana", "coisa", "visse?", "meu filho"];
    payments: ["me faz um pix", "manda o pix", "transfere pra cá"];
    questions: ["quanto tá meu filho?", "qualé o valor visse?", "me diga quanto"];
    expressions: ["oxente", "arre", "massa", "bão demais", "meu patrao"];
  };
  
  // Sul - Southern variations
  sul: {
    greetings: ["oi", "eai", "bah", "tchê"];
    financial: ["grana", "dinhero", "coisa", "bah!"];
    payments: ["me manda o pix tchê", "faz a transferência", "pix pra mim"];
    questions: ["quanto é tchê?", "qualé o valor bah?", "me fala o preço"];
    expressions: ["bah", "tchê", "guri", "legal", "show"];
  };
}

// Voice command patterns for Brazilian financial operations
const BRAZILIAN_VOICE_COMMANDS = {
  // Balance inquiries with regional variations
  balanceQueries: [
    "Como está meu saldo?",                    // Standard
    "Quanto tenho na conta?",                 // Standard
    "Me fala quanto tenho de grana",          // SP/RJ
    "Qualé o saldo da conta meu bem?",        // SP
    "Oxente, quanto tô tendo?",               // NE
    "Bah, me diz o saldo tchê",               // Sul
    "Me mostra quanto tenho",                 // General
  ],
  
  // Spending capacity inquiries
  spendingCapacity: [
    "Quanto posso gastar esse mês?",          // Standard
    "Qualé meu limite?",                      // SP/RJ
    "Me fala quanto sobra pra gastar",        // General
    "Oxente, dá pra gastar quanto?",          // NE
    "Bah, qualé o limite tchê?",              // Sul
  ],
  
  // Bill inquiries
  billQueries: [
    "Tem algum boleto programado?",           // Standard
    "Tem boleta pra pagar?",                  // SP
    "Qualé os boletos?",                      // RJ
    "Oxente, tem conta pra pagar?",           // NE
    "Tem alguma boleta vencendo?",            // Sul
    "Me fala das contas do mês",              // General
  ],
  
  // Income inquiries
  incomeQueries: [
    "Tem algum recebimento programado?",      // Standard
    "Vai entrar dinheiro esse mês?",          // General
    "Qualé os recebimentos?",                 // SP/RJ
    "Oxente, vai entrar grana?",              // NE
    "Bah, vem dinheiro pra conta tchê?",      // Sul
  ],
  
  // Balance projections
  balanceProjections: [
    "Como ficará meu saldo no final do mês?", // Standard
    "Quanto vou ter no fim do mês?",          // General
    "Me fala como fica o saldo",              // SP
    "Oxente, quanto vai sobrar?",             // NE
    "Bah, qualé o saldo final tchê?",         // Sul
  ],
  
  // Transfer commands
  transferCommands: [
    "Faz uma transferência para [nome]",      // Standard
    "Manda grana pra [nome]",                 // SP/RJ
    "Pix pra [nome]",                         // All regions
    "Transfere [valor] pra [nome]",           // General
    "Faz um pix pra [nome] meu bem",          // SP
    "Oxente, manda dinheiro pra [nome]",      // NE
    "Bah, me ajuda com pix pra [nome] tchê",  // Sul
  ],
};

// Context-aware processing for Brazilian financial commands
interface BrazilianFinancialContext {
  // User preferences and habits
  userPreferences: {
    region: keyof BrazilianPortuguesePatterns;
    formalityLevel: number; // 0 (informal) to 1 (formal)
    commonPhrases: string[];
    slangUsage: number; // 0-1 frequency
  };
  
  // Financial context
  financialContext: {
    accountTypes: string[];
    paymentMethods: string[];
    billCategories: string[];
    incomeSources: string[];
    typicalAmounts: number[];
  };
  
  // Conversation context
  conversationContext: {
    previousCommands: string[];
    currentIntent?: string;
    entitiesExtracted: Record<string, any>;
    confidenceLevel: number;
  };
}

// Brazilian entity extraction patterns
interface BrazilianEntityPatterns {
  // Money amounts with Brazilian variations
  moneyPatterns: [
    /\d+ reais?/i,
    /\d+,\d{2}/i,
    /R\$\s*\d+/i,
    /\d+ real/i,
    /\d+ pilas?/i,      // Slang
    /\d+ merreques?/i,  // Slang
    /\d+ granas?/i,     // Slang
  ];
  
  // PIX key patterns
  pixKeyPatterns: {
    cpf: /\d{3}\.\d{3}\.\d{3}-\d{2}/,
    cnpj: /\d{2}\.\d{3}\.\d{3}\/\d{4}-\d{2}/,
    phone: /\(\d{2}\)\s*\d{4,5}-\d{4}/,
    email: /[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/,
    random: /[a-f0-9]{8}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{12}/i,
  };
  
  // Date patterns with Brazilian variations
  datePatterns: [
    /\d{1,2}\/\d{1,2}\/\d{4}/,              // DD/MM/YYYY
    /\d{1,2} de [a-z]+ de \d{4}/i,           // "15 de janeiro de 2024"
    /dia \d{1,2}/i,                          // "dia 15"
    /mês que vem/i,                          // "mês que vem"
    /semana que vem/i,                       // "semana que vem"
    /hoje/i,                                 // "hoje"
    /amanhã/i,                               // "amanhã"
  ];
  
  // Brazilian bill types
  billTypes: [
    /boleto/i,
    /conta/i,
    /fatura/i,
    /aluguel/i,
    /condomínio/i,
    /escola/i,
    /plano de saúde/i,
    /internet/i,
    /luz/i,
    /água/i,
    /telefone/i,
  ];
}
```

### Voice-First Architecture

#### Voice Command Processing
```typescript
// Essential voice commands system
interface VoiceCommand {
  command: string;
  intent: 'balance_query' | 'payment_query' | 'transfer_query' | 'schedule_check';
  confidence: number;
  response: string;
  processingTime: number;
}

// Six essential voice commands
const ESSENTIAL_COMMANDS = [
  "Como está meu saldo?",           // Balance query
  "Quanto posso gastar esse mês?",   // Spending capacity
  "Tem algum boleto programado?",    // Scheduled bills
  "Tem algum recebimento programado?", // Scheduled income
  "Como ficará meu saldo no final do mês?", // Projection
  "Faz uma transferência para..."    // Money transfer
];
```

#### AI Autonomy Levels
```typescript
interface AutonomyLevel {
  level: number;        // 50-95% autonomy progression
  capabilities: string[];
  confirmationRequired: boolean;
  trustScore: number;
}

const AUTONOMY_LEVELS = {
  LEARNING: { level: 25, requiresConfirmation: true, description: "Observing user patterns" },
  ASSISTANT: { level: 50, requiresConfirmation: true, description: "Suggesting actions" },
  AUTONOMOUS: { level: 75, requiresConfirmation: false, description: "Executing routine actions" },
  TRUSTED: { level: 95, requiresConfirmation: false, description: "Full autonomous management" }
};
```

### Real-Time Architecture

#### Event-Driven Design
```typescript
// Domain events for financial operations
interface FinancialEvent {
  id: string;
  aggregateId: string;
  aggregateType: 'Transaction' | 'Account' | 'Budget';
  eventType: string;
  data: any;
  metadata: {
    userId: string;
    timestamp: string;
    source: 'voice' | 'ui' | 'api' | 'webhook';
  };
}

// Real-time event handling
class FinancialEventDispatcher {
  async dispatch(event: FinancialEvent): Promise<void> {
    // Store event in event store
    await this.eventStore.save(event);
    
    // Update real-time subscriptions
    await this.realtimeService.publish(event);
    
    // Execute synchronous handlers
    await this.executeHandlers(event);
    
    // Trigger AI autonomy evaluation
    await this.aiEngine.evaluate(event);
  }
}
```

### Security & LGPD Compliance

#### Data Protection Architecture
```typescript
// LGPD compliance implementation
interface LGPDCompliance {
  dataMinimization: boolean;      // Collect only necessary data
  purposeLimitation: boolean;     // Use data for declared purposes only
  retentionPolicy: string;        // Data retention periods
  userRights: {
    access: boolean;              // Right to access data
    deletion: boolean;            // Right to be forgotten
    portability: boolean;         // Right to data portability
    consent: boolean;             // Explicit consent management
  };
}

// Security layers
interface SecurityArchitecture {
  encryption: {
    atRest: 'AES-256';           // Supabase managed
    inTransit: 'TLS 1.3';        // Enforced
  };
  authentication: {
    primary: 'biometric';        // Face ID, Touch ID
    secondary: 'password';       // Traditional backup
    session: 'JWT';              // Secure tokens
  };
  authorization: {
    rowLevelSecurity: true;      // Tenant isolation
    auditTrails: true;           // Complete operation logging
  };
}
```

### Performance Optimization

#### Voice Response Optimization
```typescript
// Performance targets for voice interactions
const VOICE_PERFORMANCE_TARGETS = {
  speechToText: 200,      // milliseconds
  intentProcessing: 150,  // milliseconds
  actionExecution: 100,   // milliseconds
  textToSpeech: 50,       // milliseconds
  totalResponse: 500      // milliseconds maximum
};

// Caching strategy for financial data
interface CacheStrategy {
  userProfiles: { ttl: 300000, max: 100 };     // 5 minutes
  accountBalances: { ttl: 30000, max: 50 };     // 30 seconds
  transactionHistory: { ttl: 60000, max: 200 }; // 1 minute
  exchangeRates: { ttl: 86400000, max: 10 };    // 24 hours
};
```

### Database Architecture

#### Schema Design Patterns
```sql
-- Core user entity with LGPD compliance
CREATE TABLE users (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  email TEXT UNIQUE NOT NULL,
  autonomy_level INTEGER DEFAULT 50 CHECK (autonomy_level >= 50 AND autonomy_level <= 95),
  voice_command_enabled BOOLEAN DEFAULT true,
  data_processing_consent TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Financial transactions with audit trail
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  amount DECIMAL(15,2) NOT NULL,
  description TEXT NOT NULL,
  category TEXT,
  transaction_date TIMESTAMP WITH TIME ZONE NOT NULL,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'completed', 'failed')),
  metadata JSONB DEFAULT '{}',
  voice_command_id UUID REFERENCES voice_commands(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Voice commands for AI learning
CREATE TABLE voice_commands (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  command TEXT NOT NULL,
  intent TEXT NOT NULL,
  confidence DECIMAL(3,2),
  response TEXT,
  processing_time_ms INTEGER,
  was_successful BOOLEAN,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);
```

### Development Workflow Patterns

#### Essential Commands
```bash
# Development workflow
bun dev                    # Start full-stack development
bun dev:client             # Frontend only (Vite + React)
bun dev:server             # Backend only (Hono + tRPC)

# Quality assurance
bun lint                   # OXLint + Biome validation
bun type-check             # TypeScript strict mode
bun test:coverage          # Unit tests with coverage
bun quality                # Full CI pipeline

# Database operations
bunx supabase db push      # Apply migrations
bunx supabase gen types    # Generate TypeScript types
bunx supabase db diff      # Schema validation
```

#### Import Patterns
```typescript
// Supabase client integration
import { supabase } from "@/integrations/supabase/client";

// tRPC procedures
import { router, protectedProcedure } from "@/server/trpc";

// React Query for server state
import { useQuery, useMutation } from "@tanstack/react-query";

// TanStack Router
import { createFileRoute } from "@tanstack/react-router";

// Voice processing
import { useVoiceRecognition } from "@/hooks/useVoiceRecognition";
```

## Implementation Patterns

### Component Architecture
```typescript
// Voice-first component template
interface VoiceComponentProps {
  readonly onVoiceCommand?: (command: VoiceCommand) => void;
  readonly accessibilityLabel?: string;
  readonly children?: React.ReactNode;
}

export const VoiceFinancialComponent: React.FC<VoiceComponentProps> = ({
  onVoiceCommand,
  accessibilityLabel,
  children
}) => {
  const { isListening, transcript } = useVoiceRecognition();
  const { announceToScreenReader } = useAccessibility();
  
  return (
    <div 
      role="application"
      aria-label={accessibilityLabel}
      className={cn(
        "voice-component",
        isListening && "voice-listening"
      )}
    >
      {children}
      {isListening && (
        <VoiceIndicator transcript={transcript} />
      )}
    </div>
  );
};
```

### Enhanced NLU System Architecture
```typescript
// Complete NLU system with Brazilian Portuguese specialization
interface EnhancedNLUSystem {
  // Core processing engine with sub-200ms target
  processCommand: (audio: AudioBuffer) => Promise<NLUResult>;
  
  // Brazilian Portuguese patterns with regional variations
  brazilianPatterns: {
    regions: RegionalPattern[];
    slangTerms: SlangTerm[];
    financialTerminology: FinancialTerm[];
  };
  
  // Learning analytics with hit/miss tracking
  analytics: {
    trackHitMiss: (command: string, success: boolean) => void;
    getAccuracyMetrics: () => AccuracyMetrics;
    generateLearningReport: () => LearningReport;
  };
  
  // Error recovery with multiple strategies
  errorRecovery: {
    classifyError: (error: NLUError) => ErrorType;
    applyRecoveryStrategy: (error: NLUError) => Promise<NLUResult>;
    learnFromCorrection: (original: string, corrected: string) => void;
  };
  
  // Performance monitoring and optimization
  performance: {
    trackProcessingTime: (duration: number) => void;
    getSystemHealth: () => HealthMetrics;
    generatePerformanceReport: () => PerformanceReport;
  };
}

// Voice Activity Detection for real-time speech processing
interface VoiceActivityDetection {
  // Real-time speech detection with <20ms latency
  detectSpeech: (audioChunk: AudioBuffer) => boolean;
  
  // Automatic speech end detection
  detectSpeechEnd: (silenceDuration: number) => boolean;
  
  // Energy-based voice activity detection
  calculateEnergyLevel: (audio: AudioBuffer) => number;
}

// Performance optimization patterns
const VOICE_PERFORMANCE_TARGETS = {
  speechToText: 200,        // milliseconds (reduced from 500ms)
  intentProcessing: 150,    // milliseconds
  actionExecution: 100,     // milliseconds
  textToSpeech: 50,         // milliseconds
  totalResponse: 500,       // milliseconds maximum (reduced from 2000ms)
  
  // Voice recognition settings
  autoStopTimeout: 3000,    // 3 seconds (reduced from 10s)
  processingDelay: 100,     // 100ms (reduced from 500ms)
  minAudioDuration: 300,    // 0.3 seconds minimum
  maxAudioDuration: 10000,  // 10 seconds maximum
  silenceDuration: 1500,    // 1.5 seconds (reduced from 2s)
};

// Brazilian Portuguese regional patterns
interface BrazilianRegionalPatterns {
  // São Paulo variations
  saoPaulo: {
    slang: ["meu bem", "grana", "boleta", "rolê", "parça"];
    patterns: ["quanto tá", "me fala", "tipo assim"];
  };
  
  // Rio de Janeiro variations  
  rioDeJaneiro: {
    slang: ["maneiro", "caraca", "você é brabo", "demais"];
    patterns: ["me ajuda", "qualé", "sinistro"];
  };
  
  // Nordeste variations
  nordeste: {
    slang: ["oxente", "bão", "arre", "massa", "visse"];
    patterns: ["me diga", "como cê tá", "vamos nessa"];
  };
  
  // Sul variations
  sul: {
    slang: ["bah", "tchê", "guri", "legal", "show"];
    patterns: ["me mostra", "quanto é", "vamos fazer"];
  };
}

// Enhanced voice command hook with optimizations
interface OptimizedVoiceRecognition {
  // Performance monitoring
  processingTime: number;
  successRate: number;
  errorRate: number;
  
  // Voice Activity Detection
  voiceActivityDetection: VoiceActivityDetection;
  
  // Brazilian Portuguese processing
  regionalProcessor: BrazilianRegionalProcessor;
  
  // Learning and analytics
  analytics: NLUAnalytics;
  
  // Error recovery
  errorRecovery: ErrorRecoverySystem;
  
  // Performance optimization methods
  optimizeProcessing: () => void;
  clearCache: () => void;
  getPerformanceMetrics: () => PerformanceMetrics;
}

// Memory leak prevention patterns
const MEMORY_MANAGEMENT = {
  // Cleanup intervals and timeouts
  clearAllTimers: () => {
    clearInterval(refreshInterval.current);
    clearTimeout(timeoutRef.current);
    clearTimeout(clearResponseRef.current);
  },
  
  // Abort controller for async operations
  abortController: new AbortController(),
  
  // Proper cleanup on unmount
  cleanup: () => {
    voiceActivityDetection?.cleanup();
    abortController.abort();
    clearAllTimers();
  },
};
```

### API Design Patterns
```typescript
// tRPC procedure with comprehensive validation
export const createTransactionRouter = router({
  create: protectedProcedure
    .input(z.object({
      amount: z.number().positive().max(1000000),
      description: z.string().min(1).max(500),
      category: z.enum(['food', 'transport', 'housing', 'entertainment', 'other']),
      date: z.string().datetime().optional(),
      voiceCommand: z.string().optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Business logic validation
      if (input.amount > ctx.user.dailyLimit) {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'Amount exceeds daily spending limit'
        });
      }
      
      // Create transaction with audit trail
      const transaction = await createTransaction({
        userId: ctx.user.id,
        ...input,
        source: 'voice_command',
        metadata: {
          ip: ctx.ip,
          userAgent: ctx.userAgent,
        }
      });
      
      // Real-time update
      await ctx.realtime.publish(`user:${ctx.user.id}:transactions`, {
        type: 'transaction_created',
        data: transaction
      });
      
      return transaction;
    }),
});

// Enhanced voice command processing API
export const voiceCommandRouter = router({
  process: protectedProcedure
    .input(z.object({
      audioData: z.string(), // base64 encoded audio
      region: z.enum(['SP', 'RJ', 'NE', 'SUL', 'NORTE', 'CO']).optional(),
      context: z.object({
        previousCommands: z.array(z.string()).optional(),
        userPreferences: z.object({
          language: z.string().optional(),
          slangLevel: z.number().optional(), // 0-1 formality level
        }).optional(),
      }).optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Process with enhanced NLU system
      const startTime = Date.now();
      
      try {
        const result = await enhancedNLUEngine.processCommand({
          audioData: input.audioData,
          userId: ctx.user.id,
          region: input.region || 'SP',
          context: input.context,
        });
        
        const processingTime = Date.now() - startTime;
        
        // Track performance metrics
        await voiceAnalytics.trackProcessing({
          userId: ctx.user.id,
          processingTime,
          success: result.success,
          confidence: result.confidence,
          intent: result.intent,
        });
        
        return {
          ...result,
          processingTime,
          performanceTarget: processingTime <= VOICE_PERFORMANCE_TARGETS.totalResponse,
        };
        
      } catch (error) {
        // Error recovery attempt
        const recoveryResult = await errorRecovery.attemptRecovery({
          error,
          userId: ctx.user.id,
          originalInput: input,
        });
        
        return recoveryResult;
      }
    }),

  // Brazilian Portuguese pattern learning
  learnPattern: protectedProcedure
    .input(z.object({
      originalCommand: z.string(),
      correctedCommand: z.string(),
      region: z.string(),
      context: z.string().optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      await brazilianPatterns.learnFromCorrection({
        userId: ctx.user.id,
        original: input.originalCommand,
        corrected: input.correctedCommand,
        region: input.region,
        context: input.context,
      });
      
      return { success: true };
    }),

  // Performance analytics endpoint
  getAnalytics: protectedProcedure
    .input(z.object({
      timeframe: z.enum(['hour', 'day', 'week', 'month']),
      metrics: z.array(z.enum([
        'accuracy', 'latency', 'success_rate', 'error_rate', 'regional_performance'
      ])),
    }))
    .query(async ({ ctx, input }) => {
      return voiceAnalytics.getDetailedMetrics({
        userId: ctx.user.id,
        timeframe: input.timeframe,
        metrics: input.metrics,
      });
    }),
});

// Voice command optimization patterns
const VOICE_OPTIMIZATION_PATTERNS = {
  // Caching strategies for frequently used commands
  caching: {
    commonCommands: new Map([
      ['saldo', { cachedResponse: true, ttl: 30000 }], // 30s cache
      ['transferência', { cachedResponse: false, ttl: 0 }],
      ['boletos', { cachedResponse: true, ttl: 60000 }], // 1m cache
    ]),
    
    // Regional pattern caching
    regionalPatterns: new Map([
      ['SP', { patterns: saoPauloPatterns, lastUpdated: Date.now() }],
      ['RJ', { patterns: rioPatterns, lastUpdated: Date.now() }],
      ['NE', { patterns: nordestePatterns, lastUpdated: Date.now() }],
    ]),
  },
  
  // Performance monitoring
  monitoring: {
    // Real-time metrics collection
    collectMetrics: (command: string, processingTime: number, success: boolean) => {
      performanceTracker.record({
        command,
        processingTime,
        success,
        timestamp: Date.now(),
        userId: getCurrentUserId(),
      });
    },
    
    // Health check system
    healthCheck: async () => {
      const health = await systemHealth.check({
        voiceRecognition: { targetLatency: 200, currentLatency: getCurrentLatency() },
        nluProcessing: { targetLatency: 150, currentLatency: getNLULatency() },
        databaseOperations: { targetLatency: 100, currentLatency: getDBLatency() },
      });
      
      return health;
    },
  },
  
  // Error recovery strategies
  errorRecovery: {
    strategies: [
      'pattern_matching',    // Try to match known patterns
      'context_inference',   // Use conversation context
      'regional_adaptation', // Adapt for regional variations
      'user_history',        // Learn from user's past corrections
      'clarification_request', // Ask user for help
    ],
    
    applyStrategy: async (error: NLUError) => {
      for (const strategy of errorRecovery.strategies) {
        try {
          const result = await applyRecoveryStrategy(strategy, error);
          if (result.success) return result;
        } catch (e) {
          // Try next strategy
        }
      }
      
      // Final fallback - ask for clarification
      return requestClarification(error);
    },
  },
};
```

## Validation & Quality Gates

### Architecture Compliance Checklist
- [ ] **KISS Principle**: Simple, direct implementation without over-engineering
- [ ] **YAGNI Principle**: Only essential features from requirements implemented
- [ ] **Type Safety**: End-to-end TypeScript with no implicit any
- [ ] **Voice First**: Primary interaction through voice commands
- [ ] **Real-time**: Instant updates via Supabase subscriptions
- [ ] **Security**: Row Level Security on all database tables
- [ ] **Performance**: Sub-500ms voice response times (updated to ≤500ms target)
- [ ] **Accessibility**: WCAG 2.1 AA+ compliance
- [ ] **LGPD**: Brazilian data protection compliance
- [ ] **Brazilian Market**: PIX, boletos, Portuguese localization
- [ ] **NLU System**: Enhanced Natural Language Understanding with Brazilian Portuguese specialization
- [ ] **Performance Monitoring**: Real-time analytics and learning systems
- [ ] **Voice Activity Detection**: Real-time speech processing with <20ms detection
- [ ] **Error Recovery**: Multi-strategy recovery with 80%+ success rate
- [ ] **Regional Adaptation**: Support for 6 major Brazilian regional variations

### Enhanced Code Quality Standards
- **Test Coverage**: 90%+ for critical business logic including NLU systems
- **TypeScript**: Strict mode enabled, zero implicit any
- **Error Handling**: Comprehensive error boundaries and user feedback
- **Security**: Input validation, SQL injection prevention, XSS protection
- **Performance**: Lighthouse score ≥90, Core Web Vitals compliance
- **Voice Performance**: Sub-200ms processing for 95% of voice commands
- **Memory Management**: Zero memory leaks from intervals/timeouts
- **Regional Accuracy**: 90%+ accuracy for Brazilian Portuguese variations
- **System Health**: 99.9% uptime with automated monitoring

### Enhanced Performance Standards
```typescript
// Updated performance targets based on latest optimizations
const ENHANCED_PERFORMANCE_TARGETS = {
  // Voice processing (reduced from previous targets)
  speechToText: 200,        // milliseconds (reduced from 500ms)
  intentProcessing: 150,    // milliseconds (reduced from 300ms)
  actionExecution: 100,     // milliseconds (reduced from 200ms)
  textToSpeech: 50,         // milliseconds (reduced from 100ms)
  totalResponse: 500,       // milliseconds maximum (reduced from 1000ms)
  
  // Voice recognition optimizations
  autoStopTimeout: 3000,    // 3 seconds (reduced from 10s)
  processingDelay: 100,     // 100ms (reduced from 500ms)
  minAudioDuration: 300,    // 0.3 seconds minimum
  maxAudioDuration: 10000,  // 10 seconds maximum (reduced from 30s)
  silenceDuration: 1500,    // 1.5 seconds (reduced from 2s)
  
  // NLU system performance
  nluProcessing: 150,       // milliseconds target
  accuracyThreshold: 0.90,  // 90% accuracy target
  errorRecoveryRate: 0.80,  // 80% error recovery rate
  cacheHitRate: 0.60,       // 60% cache hit rate
  
  // Regional performance targets
  regionalAccuracy: {
    SP: 0.95,   // São Paulo: 95%
    RJ: 0.92,   // Rio de Janeiro: 92%
    NE: 0.88,   // Nordeste: 88%
    SUL: 0.90,  // Sul: 90%
    NORTE: 0.85, // Norte: 85%
    CO: 0.87,   // Centro-Oeste: 87%
  },
  
  // System health targets
  systemUptime: 0.999,      // 99.9% uptime
  memoryUsage: 0.75,        // 75% maximum memory usage
  cpuUsage: 0.70,           // 70% maximum CPU usage
  databaseLatency: 100,     // 100ms maximum database latency
};
```

### Enhanced Quality Gates Checklist
- [ ] **Voice Processing**: Sub-200ms speech-to-text processing
- [ ] **NLU Accuracy**: 90%+ accuracy for Brazilian financial commands
- [ ] **Regional Support**: 6 major Brazilian regions with dedicated patterns
- [ ] **Error Recovery**: 80%+ success rate for intelligent error recovery
- [ ] **Performance Monitoring**: Real-time analytics with automated alerting
- [ ] **Learning System**: Continuous learning from user corrections
- [ ] **Memory Management**: Zero memory leaks from intervals/timeouts
- [ ] **Voice Activity Detection**: Real-time speech detection with <20ms latency
- [ ] **Cache Performance**: 60%+ hit rate for common commands
- [ ] **System Health**: 99.9% uptime with comprehensive monitoring

### Enhanced Testing Requirements
```typescript
// Comprehensive testing patterns for voice systems
const ENHANCED_TESTING_REQUIREMENTS = {
  // Unit tests
  unitTests: {
    nluEngine: 95,           // 95% coverage for NLU engine
    brazilianPatterns: 100,  // 100% coverage for regional patterns
    errorRecovery: 90,       // 90% coverage for error recovery
    performanceTracking: 85, // 85% coverage for performance systems
  },
  
  // Integration tests
  integrationTests: {
    voiceProcessing: 90,     // 90% coverage for end-to-end voice processing
    regionalVariations: 85,  // 85% coverage for regional pattern testing
    realTimeUpdates: 80,     // 80% coverage for real-time systems
    errorRecovery: 85,       // 85% coverage for error recovery scenarios
  },
  
  // Performance tests
  performanceTests: {
    voiceLatency: 'sub_500ms', // All voice operations under 500ms
    memoryUsage: 'no_leaks',   // Zero memory leaks
    concurrency: '100_users',  // Support for 100 concurrent users
    regionalAccuracy: '90_percent_plus', // 90%+ accuracy across regions
  },
  
  // Accessibility tests
  accessibilityTests: {
    screenReader: 'wcag_aa_plus', // WCAG 2.1 AA+ compliance
    voiceNavigation: 'full_support', // Full voice navigation support
    keyboardNavigation: 'complete', // Complete keyboard navigation
    colorContrast: '4_5_ratio', // 4.5:1 minimum contrast ratio
  },
  
  // Security tests
  securityTests: {
    sqlInjection: 'prevention', // SQL injection prevention
    xssPrevention: 'full',      // XSS prevention
    dataEncryption: 'aes256',   // AES-256 encryption
    lgpdCompliance: 'full',     // Full LGPD compliance
  },
};
```
```

## Common Architectural Decisions

### Technology Choices
- **Bun over Node.js**: 3-5x performance improvement for voice processing
- **Supabase over Firebase**: PostgreSQL for financial data, better querying capabilities
- **tRPC over REST**: End-to-end type safety eliminates runtime errors
- **Hono over Express**: Edge-first architecture for sub-150ms API responses
- **React 19 over Vue**: Better voice processing hooks and concurrent features

### Integration Patterns
- **Voice Commands**: Speech-to-text → Intent classification → Action execution → Text-to-speech
- **Real-time Updates**: Supabase subscriptions → Optimistic UI updates → Conflict resolution
- **Security**: JWT authentication → Row Level Security → Audit logging → LGPD compliance
- **Performance**: Edge deployment → Client-side caching → Lazy loading → Bundle optimization

## Troubleshooting Common Issues

### Voice Processing Performance
```typescript
// Optimize voice command processing
const optimizeVoiceProcessing = {
  // Preload speech recognition models
  preloadModels: true,
  
  // Use Web Workers for heavy processing
  useWebWorkers: true,
  
  // Cache common command patterns
  cacheCommonPatterns: true,
  
  // Batch processing for multiple commands
  batchCommands: true,
};
```

### Real-time Sync Issues
```typescript
// Handle real-time synchronization conflicts
const handleSyncConflicts = {
  // Version-based conflict resolution
  useVersioning: true,
  
  // Optimistic updates with rollback
  optimisticUpdates: true,
  
  // Automatic retry with exponential backoff
  retryStrategy: 'exponential-backoff',
  
  // User notification for manual resolution
  notifyOnConflict: true,
};
```

### Performance Monitoring and Analytics System
```typescript
// Comprehensive performance tracking for voice operations
interface VoicePerformanceAnalytics {
  // Real-time metrics collection
  realTimeMetrics: {
    processingTimes: number[];           // Track command processing times
    accuracyRates: Map<string, number>;  // Accuracy by command type
    errorRates: Map<string, number>;     // Error rates by category
    userSatisfaction: number[];          // User feedback scores
    
    // Brazilian regional performance
    regionalPerformance: {
      SP: { accuracy: number, latency: number };
      RJ: { accuracy: number, latency: number };
      NE: { accuracy: number, latency: number };
      SUL: { accuracy: number, latency: number };
      NORTE: { accuracy: number, latency: number };
      CO: { accuracy: number, latency: number };
    };
  };
  
  // System health monitoring
  systemHealth: {
    voiceRecognition: {
      targetLatency: 200;       // milliseconds
      currentLatency: number;
      successRate: number;
      memoryUsage: number;
    };
    
    nluProcessing: {
      targetLatency: 150;       // milliseconds
      currentLatency: number;
      accuracy: number;
      confidenceThreshold: number;
    };
    
    databaseOperations: {
      targetLatency: 100;       // milliseconds
      currentLatency: number;
      connectionPool: number;
      queryPerformance: number;
    };
    
    overallSystem: {
      uptime: number;
      errorCount: number;
      activeUsers: number;
      performanceScore: number; // 0-100
    };
  };
  
  // Learning analytics
  learningAnalytics: {
    patternEvolution: {
      newPatterns: string[];
      improvedPatterns: string[];
      deprecatedPatterns: string[];
      adaptationRate: number;
    };
    
    userAdaptation: {
      correctionsPerUser: Map<string, number>;
      learningRate: number;
      retentionRate: number;
      satisfactionImprovement: number;
    };
    
    brazilianContextLearning: {
      regionalSlangAdoption: Map<string, number>;
      culturalContextUnderstanding: number;
      financialTerminologyAccuracy: number;
    };
  };
  
  // Performance alerts and notifications
  alertSystem: {
    thresholds: {
      maxLatency: 500;          // milliseconds
      minAccuracy: 0.85;        // 85%
      maxErrorRate: 0.15;       // 15%
      minSystemHealth: 80;      // 80/100
    };
    
    alerts: {
      performanceDegradation: boolean;
      accuracyDrop: boolean;
      systemFailure: boolean;
      userComplaints: boolean;
    };
    
    notificationChannels: [
      'dashboard',
      'email',
      'slack',
      'sms'
    ];
  };
}

// Implementation of performance tracking
const performanceTracker = {
  // Track individual command performance
  trackCommand: async (command: VoiceCommand, result: NLUResult) => {
    const metrics = {
      commandId: command.id,
      userId: command.userId,
      region: command.region,
      processingTime: result.processingTime,
      success: result.success,
      confidence: result.confidence,
      intent: result.intent,
      timestamp: Date.now(),
    };
    
    // Store in performance database
    await supabase.from('voice_command_metrics').insert(metrics);
    
    // Update real-time metrics
    updateRealTimeMetrics(metrics);
    
    // Check for performance alerts
    checkPerformanceThresholds(metrics);
    
    // Update learning analytics
    updateLearningAnalytics(metrics);
  },
  
  // Generate performance reports
  generateReport: async (timeframe: 'hour' | 'day' | 'week' | 'month') => {
    const report = await supabase
      .from('voice_command_metrics')
      .select('*')
      .gte('timestamp', getTimestampForTimeframe(timeframe));
    
    return {
      summary: calculateSummaryStats(report.data),
      regionalBreakdown: calculateRegionalStats(report.data),
      trends: calculateTrends(report.data),
      recommendations: generateRecommendations(report.data),
    };
  },
  
  // Performance optimization suggestions
  optimizePerformance: async () => {
    const currentMetrics = await getCurrentMetrics();
    const optimizations = [];
    
    if (currentMetrics.voiceRecognition.latency > 200) {
      optimizations.push({
        area: 'voice_recognition',
        suggestion: 'Increase VAD sensitivity or reduce audio buffer size',
        expectedImprovement: '15-25% latency reduction',
      });
    }
    
    if (currentMetrics.nluProcessing.accuracy < 0.90) {
      optimizations.push({
        area: 'nlu_processing',
        suggestion: 'Add more Brazilian Portuguese training data',
        expectedImprovement: '5-10% accuracy increase',
      });
    }
    
    if (currentMetrics.databaseOperations.latency > 100) {
      optimizations.push({
        area: 'database',
        suggestion: 'Add query indexes or implement caching',
        expectedImprovement: '20-30% latency reduction',
      });
    }
    
    return optimizations;
  },
};

// Brazilian Portuguese learning system
const brazilianLearningSystem = {
  // Learn from user corrections
  learnFromCorrection: async (correction: UserCorrection) => {
    const learningData = {
      originalCommand: correction.original,
      correctedCommand: correction.corrected,
      region: correction.region,
      userId: correction.userId,
      timestamp: Date.now(),
    };
    
    // Store learning data
    await supabase.from('brazilian_portuguese_learning').insert(learningData);
    
    // Update pattern recognition
    await updatePatternRecognition(learningData);
    
    // Improve regional models
    await improveRegionalModels(correction.region, learningData);
    
    // Track learning effectiveness
    await trackLearningEffectiveness(correction);
  },
  
  // Adapt to regional variations
  adaptToRegionalVariations: async (region: string, patterns: string[]) => {
    const adaptationData = {
      region,
      patterns,
      adaptationDate: Date.now(),
      confidence: calculateConfidence(patterns),
    };
    
    // Update regional patterns
    await supabase.from('regional_patterns').upsert(adaptationData);
    
    // Deploy to production if confidence is high enough
    if (adaptationData.confidence > 0.8) {
      await deployRegionalAdaptation(region, patterns);
    }
  },
  
  // Generate learning insights
  generateLearningInsights: async () => {
    const insights = await supabase
      .from('brazilian_portuguese_learning')
      .select('*')
      .gte('timestamp', Date.now() - 7 * 24 * 60 * 60 * 1000); // Last week
    
    return {
      topCorrections: getTopCorrections(insights.data),
      regionalInsights: getRegionalInsights(insights.data),
      accuracyImprovements: calculateAccuracyImprovements(insights.data),
      futureRecommendations: generateFutureRecommendations(insights.data),
    };
  },
};

// Error analysis and recovery patterns
const errorAnalysisSystem = {
  // Analyze common error patterns
  analyzeErrorPatterns: async () => {
    const errors = await supabase
      .from('voice_command_errors')
      .select('*')
      .gte('timestamp', Date.now() - 24 * 60 * 60 * 1000); // Last 24 hours
    
    const errorPatterns = {
      regionalErrors: categorizeErrorsByRegion(errors.data),
      intentErrors: categorizeErrorsByIntent(errors.data),
      entityErrors: categorizeErrorsByEntity(errors.data),
      systemErrors: categorizeSystemErrors(errors.data),
    };
    
    return {
      patterns: errorPatterns,
      recommendations: generateErrorRecommendations(errorPatterns),
      automatedFixes: identifyAutomatedFixes(errorPatterns),
    };
  },
  
  // Implement automated error recovery
  implementErrorRecovery: async (errorType: string, fix: ErrorFix) => {
    const recoveryImplementation = {
      errorType,
      fixType: fix.type,
      implementation: fix.implementation,
      rolloutPercentage: fix.rolloutPercentage || 0.1, // Start with 10%
      effectiveness: 0,
      createdAt: Date.now(),
    };
    
    // Store recovery implementation
    await supabase.from('error_recovery_implementations').insert(recoveryImplementation);
    
    // Monitor effectiveness
    monitorRecoveryEffectiveness(recoveryImplementation);
    
    return recoveryImplementation;
  },
};
```

## Resources and References

### Essential Documentation
- `references/tech-stack.md`: Complete technology specifications (Bun + Hono + React 19 + Supabase + tRPC)
- `references/database-schema.md`: Database design patterns with RLS and LGPD compliance
- `references/security-guidelines.md`: Security implementation guide with financial regulations
- `references/voice-interface.md`: Enhanced voice interaction patterns with regional variations
- `references/lgpd-compliance.md`: Brazilian data protection and privacy regulations
- `references/nlu-system.md`: Complete NLU architecture with Brazilian Portuguese specialization
- `references/performance-optimization.md`: Voice performance optimization patterns and benchmarks

### Implementation Files (Latest)
- `src/lib/nlu/enhancedNLUEngine.ts`: Complete integrated NLU system
- `src/lib/nlu/brazilianPatterns.ts`: Brazilian Portuguese patterns with regional variations
- `src/lib/nlu/analytics.ts`: Hit/miss tracking and learning analytics
- `src/lib/nlu/contextProcessor.ts`: Context-aware processing with multi-turn support
- `src/lib/nlu/errorRecovery.ts`: Multi-strategy error recovery with 80%+ success rate
- `src/lib/nlu/performance.ts`: Real-time performance monitoring and health tracking
- `src/lib/stt/voiceActivityDetection.ts`: Real-time speech detection with <20ms latency
- `src/hooks/useVoiceRecognition.ts`: Optimized voice recognition with VAD integration
- `src/components/voice/VoiceDashboard.tsx`: Enhanced voice interface with performance indicators

### Enhanced Tools and Scripts
- `scripts/validate-architecture.py`: Architecture compliance validation with NLU checks
- `scripts/performance-audit.py`: Performance benchmarking for voice operations (<500ms target)
- `scripts/security-scan.py`: Security vulnerability scanning with financial compliance
- `scripts/database-validator.py`: Schema validation and migration checking with RLS verification
- `scripts/voice-performance-test.ts`: Comprehensive voice command performance testing
- `scripts/regional-accuracy-validator.py`: Brazilian regional pattern validation
- `scripts/memory-leak-detector.py`: Memory leak detection for voice components

### Updated Templates and Assets
- `assets/api-template/`: Enhanced tRPC procedure templates with NLU integration
- `assets/component-templates/`: React component boilerplates with voice optimization
- `assets/database-migrations/`: Migration script templates with performance indexes
- `assets/diagrams/`: Updated architecture diagrams including NLU and performance systems
- `assets/brazilian-patterns/`: Regional pattern templates for all 6 major regions
- `assets/voice-testing/`: Voice command testing templates and benchmarks
- `assets/error-recovery/`: Error recovery strategy templates and implementations

### Performance Benchmarks
```typescript
// Latest performance benchmarks achieved
const CURRENT_PERFORMANCE_BENCHMARKS = {
  // Voice processing benchmarks
  averageVoiceProcessingTime: 180,    // milliseconds (target: 200ms)
  p95VoiceProcessingTime: 220,        // milliseconds (target: 500ms)
  
  // NLU accuracy benchmarks
  overallAccuracy: 0.92,              // 92% (target: 90%)
  brazilianRegionalAccuracy: {
    SP: 0.94, RJ: 0.91, NE: 0.87,
    SUL: 0.89, NORTE: 0.84, CO: 0.86
  },
  
  // System performance benchmarks
  errorRecoverySuccessRate: 0.83,    // 83% (target: 80%)
  cacheHitRate: 0.65,                 // 65% (target: 60%)
  systemUptime: 0.9992,               // 99.92% (target: 99.9%)
  
  // User satisfaction metrics
  userSatisfactionScore: 4.6,         // 4.6/5.0
  taskCompletionRate: 0.94,           // 94%
  voiceCommandAdoptionRate: 0.78,     // 78%
};
```

## How to Use This Skill

1. **Architecture Design**: Ask for guidance on voice-first financial architecture
2. **Technology Decisions**: Get recommendations for Brazilian fintech stack choices
3. **Implementation Patterns**: Use proven patterns for NLU, voice processing, and performance
4. **Performance Optimization**: Apply latest optimizations for sub-200ms voice processing
5. **Regional Adaptation**: Implement Brazilian Portuguese variations for better user experience
6. **Quality Assurance**: Use enhanced testing patterns and validation criteria
7. **Troubleshooting**: Resolve common issues with voice processing and NLU systems

## Integration Points

This skill integrates seamlessly with:
- **Enhanced NLU System**: Complete Brazilian Portuguese specialization
- **Performance Monitoring**: Real-time analytics and health tracking
- **Voice Activity Detection**: Optimized speech processing
- **Error Recovery**: Multi-strategy recovery with learning
- **Regional Adaptation**: 6 major Brazilian regional patterns
- **Learning Analytics**: Continuous improvement from user feedback

---

## Version History

- **v2.1**: Enhanced with latest NLU system, voice performance optimizations, and comprehensive analytics
- **v2.0**: Added Brazilian Portuguese specialization and enhanced architecture patterns
- **v1.0**: Initial architecture skill for AegisWallet voice-first financial assistant

This skill provides the complete architectural foundation needed to build scalable, secure, and performant voice-first financial applications for the Brazilian market, with the latest optimizations and enhanced NLU capabilities based on real-world implementation experience.