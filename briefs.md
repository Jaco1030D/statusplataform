# Briefs Detalhados - Testes Técnicos

## 🎯 **CANDIDATO 1: `useSegmentText` Hook**

### **Contexto da Plataforma**
Você está desenvolvendo o **hook central** para manipulação de segmentos de tradução em uma plataforma concorrente da Smartcat. Este hook será usado no editor principal onde tradutores passam a maior parte do tempo trabalhando.

### **Estratégia para Mock Individual**
Para focar apenas no contexto, você pode criar **mocks simples** para funcionalidades de segmento:

```typescript
// Mock simples para funcionalidades individuais
const mockSegmentOperations = {
  validateSegment: (segment: Segment) => ({ isValid: true, errors: [] }),
  updateSegmentText: (id: string, text: string) => ({ ...segment, target: { ...segment.target, text } }),
  highlightDifferences: (segment: Segment) => "texto com <mark>diferenças</mark>"
};
```

### **Dados de Entrada (baseado no JSON mock)**
```typescript
const segmentData = {
  source: {
    text: "Mauris id ex erat",
    rawText: "<mrk mid='1' mtype='seg'><g id='1'>Mauris id ex erat.</g></mrk>",
    tags: [
      {
        id: "1",
        type: "inline_open", 
        name: "g",
        position: { start: 0, end: 3 },
        placeholder: "⟨1⟩"
      }
    ]
  },
  target: {
    text: "Tradução do texto.",
    rawText: "<mrk mid='1' mtype='seg'><g id='1'>Tradução do texto.</g></mrk>",
    tags: [/* estrutura similar */]
  }
}
```

### **Requisitos Funcionais**

#### **1. Manipulação de Texto e Tags**
- Preservar tags XML/HTML durante edição da tradução
- Validar integridade das tags (abertura/fechamento corretos)
- Permitir inserção/remoção de tags manualmente
- Converter entre texto limpo e texto com tags
- Gerar placeholders visuais para tags (⟨1⟩, ⟨/1⟩)

#### **2. Validação em Tempo Real**
- Detectar tags órfãs (abertura sem fechamento)
- Identificar tags extras ou faltantes comparando source/target
- Validar posicionamento correto das tags
- Alertar sobre possíveis erros de formatação

#### **3. Funcionalidades do Editor**
- Cursor position tracking para inserção de tags
- Highlight de diferenças entre original e tradução
- Undo/redo para alterações de texto
- Auto-completar tags baseado no contexto

#### **4. Performance e UX**
- Debounce para validações pesadas
- Memoização de cálculos complexos
- Updates otimizados (evitar re-renders desnecessários)
- Feedback visual instantâneo

### **Interface Esperada**
```typescript
interface UseSegmentTextReturn {
  // Estado principal
  displayText: string;              // Texto visível no editor
  rawText: string;                 // Texto com tags XML
  cleanText: string;               // Texto sem tags
  
  // Manipulação
  updateText: (newText: string) => void;
  insertTag: (tagId: string, position: number) => void;
  removeTag: (tagId: string) => void;
  
  // Validação
  validation: {
    isValid: boolean;
    errors: ValidationError[];
    warnings: ValidationError[];
  };
  
  // Tags
  availableTags: Tag[];
  activeTags: Tag[];
  
  // Utilities
  getTagAtPosition: (position: number) => Tag | null;
  highlightDifferences: () => HighlightedSegment;
  
  // Estado
  isDirty: boolean;
  isValidating: boolean;
  
  // Histórico
  canUndo: boolean;
  canRedo: boolean;
  undo: () => void;
  redo: () => void;
}
```

### **Casos de Teste Críticos**
1. **Tag Matching**: Source tem `<g>text</g>`, target deve manter estrutura
2. **Tag Órfãs**: Target com `<g>text` (sem fechamento)
3. **Tags Extras**: Target com tags que não existem no source
4. **Posicionamento**: Tags no local errado quebram formatação
5. **Performance**: Textos longos (>1000 caracteres) devem ser fluidos
6. **Caracteres Especiais**: Unicode, emojis, caracteres de escape

### **Entregáveis**
- Hook `useSegmentText` completo e funcional
- Componente de demonstração usando o hook
- Testes unitários cobrindo casos críticos
- Documentação técnica das decisões implementadas
- Tratamento de edge cases documentado

---

## 🎯 **CANDIDATA 2: `SegmentEditorContext` + Provider**

### **Contexto da Plataforma** 
Você está criando o **contexto central** que gerencia o estado global do editor de segmentos. Este contexto coordena múltiplos segmentos, navegação, filtros, e otimizações de performance para uma experiência fluida.

### **Dados de Entrada (baseado no JSON mock)**
```typescript
const editorData = {
  transunits: [
    {
      id: "P244D057-tu5",
      segments: [/* array de segmentos */]
    }
  ],
  pagination: {
    total: 150,
    page: 1,
    limit: 50,
    hasNext: true
  },
  aggregations: {
    statusCount: { new: 30, in_progress: 45, translated: 75 },
    avgQualityScore: 0.87,
    totalWordCount: 3000
  }
}
```

### **Requisitos Funcionais**

#### **1. Gerenciamento de Estado**
- Estado global de todos os segmentos carregados
- Segmento atualmente ativo/sendo editado
- Alterações não salvas (dirty state) por segmento
- Cache inteligente para performance
- Sincronização com Redux store

#### **2. Navegação e Filtros**
- Navegação sequencial entre segmentos (next/previous)
- Pular para segmento específico por índice
- Filtros por status (new, translated, reviewed, etc.)
- Busca por texto nos segmentos
- Filtros por qualidade, TM matches, etc.

#### **3. Performance e Virtualização**
- Paginação lazy loading
- Virtualização para grandes listas
- Pre-loading inteligente de segmentos próximos
- Debounce em buscas e filtros
- Cleanup de memória para segmentos não utilizados

#### **4. Colaboração e Sync**
- Lock/unlock de segmentos para colaboração
- WebSocket integration para updates em tempo real
- Conflict resolution para edições simultâneas
- Status sync entre usuários

#### **5. Auto-save e Persistência**
- Auto-save inteligente com debounce
- Retry logic para falhas de rede
- Persistência local para backup
- Indicadores visuais de estado de salvamento

### **Interface Esperada**
```typescript
interface SegmentEditorContextValue {
  // Estado principal
  segments: Record<string, Segment>;
  segmentsList: string[];           // IDs ordenados
  currentSegmentId: string | null;
  
  // Navegação
  currentIndex: number;
  totalSegments: number;
  goToSegment: (index: number) => void;
  nextSegment: () => void;
  previousSegment: () => void;
  
  // Filtros e busca
  filters: SegmentFilters;
  setFilters: (filters: Partial<SegmentFilters>) => void;
  searchQuery: string;
  setSearchQuery: (query: string) => void;
  filteredSegmentIds: string[];
  
  // CRUD operations
  updateSegment: (id: string, updates: Partial<Segment>) => void;
  saveSegment: (id: string) => Promise<void>;
  saveAllPending: () => Promise<void>;
  
  // Estado de loading/saving
  loadingStates: Record<string, boolean>;
  savingStates: Record<string, boolean>;
  
  // Performance
  pagination: PaginationState;
  loadNextPage: () => Promise<void>;
  
  // Colaboração
  lockedSegments: Record<string, string>; // segmentId -> userId
  lockSegment: (id: string) => void;
  unlockSegment: (id: string) => void;
  
  // Agregações e stats
  stats: {
    totalSegments: number;
    completedSegments: number;
    progress: number;
    statusCounts: Record<SegmentStatus, number>;
  };
  
  // Utilities
  getSegmentByKey: (key: string) => Segment | undefined;
  hasUnsavedChanges: boolean;
  dirtySegmentIds: string[];
}
```

### **Arquitetura Esperada**
```typescript
// Provider component
export const SegmentEditorProvider: React.FC<{
  projectId: string;
  children: React.ReactNode;
}> = ({ projectId, children }) => {
  // Implementação do provider
};

// Hook para consumir o contexto
export const useSegmentEditor = (): SegmentEditorContextValue => {
  const context = useContext(SegmentEditorContext);
  if (!context) {
    throw new Error('useSegmentEditor must be used within SegmentEditorProvider');
  }
  return context;
};

// Hook especializado para segmento atual
export const useCurrentSegment = () => {
  const { currentSegmentId, segments } = useSegmentEditor();
  return segments[currentSegmentId || ''] || null;
};
```

### **Integrações Necessárias**
- **Redux Store**: Sync com segmentSlice, projectSlice
- **API Services**: segmentService para CRUD operations
- **WebSocket**: Real-time updates de outros usuários
- **LocalStorage**: Backup de alterações não salvas
- **URL Sync**: Atualizar URL com segmento atual

### **Casos de Teste Críticos**
1. **Performance**: 1000+ segmentos sem lag
2. **Memory Management**: Cleanup correto ao sair
3. **Concurrent Edits**: Dois usuários editando simultaneamente
4. **Network Failures**: Auto-retry e fallback local
5. **Navigation**: Filtros mantêm posição correta
6. **Auto-save**: Não perder dados em falhas

### **Entregáveis**
- Context e Provider implementados
- Hooks auxiliares (useSegmentEditor, useCurrentSegment)
- Componente de demonstração usando o contexto
- Testes de integração com cenários realistas
- Documentação da arquitetura e padrões utilizados
- Performance benchmarks para cenários de uso

---

## 🔗 **INTEGRAÇÃO ENTRE AS DUAS TAREFAS**

### **Como elas se complementam:**
- **useSegmentText** foca na **manipulação individual** de cada segmento
- **SegmentEditorContext** gerencia o **estado global** e coordenação entre segmentos
- Context chama o hook para segmento ativo
- Hook reporta mudanças para o Context
- Juntos criam uma experiência de edição profissional

### **Fluxo de Dados:**
```
User Edit → useSegmentText → validation → SegmentEditorContext → 
→ updateSegment → auto-save → API → WebSocket → other users
```

### **Critérios de Avaliação:**

#### **Aspectos Técnicos:**
- Qualidade do código TypeScript
- Performance e otimizações
- Tratamento de edge cases
- Arquitetura e padrões utilizados

#### **Experiência do Desenvolvedor:**
- Facilidade de uso da API
- Documentação clara
- Testes abrangentes
- Extensibilidade da solução

#### **Pragmatismo:**
- Solve problemas reais da plataforma
- Balanceamento entre complexidade e utilidade
- Consideração de requisitos não-funcionais
- Pensamento em escala e manutenibilidade

**Prazo:** 4-6 dias para cada candidato
**Formato:** Hook + demo + testes + documentação
**Tecnologias:** React 18+, TypeScript, Custom Hooks patterns
