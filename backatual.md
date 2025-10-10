# 📋 DESIGN DOCUMENT - Plataforma de Tradução

## 🎯 **VISÃO GERAL DO PROJETO**

**Objetivo**: Plataforma de tradução colaborativa similar ao Smartcat, com sistema de monetização baseado em "moedas" e funcionalidades de IA/MT.

**Arquitetura**: Node.js + Express + MongoDB + Google Cloud Storage + WebSocket (futuro)

---

## 🏗️ **ARQUITETURA ATUAL IMPLEMENTADA**

### **Padrão Arquitetural: Controller-Entity Separation**

```
┌─────────────────────────────────────────────────┐
│                   ROUTES                        │
│  ┌─────────────────────────────────────────┐   │
│  │            MIDDLEWARES                  │   │
│  │  • AuthGuard (JWT)                      │   │
│  │  • CheckPermission (RBAC)              │   │
│  │  • Validation (express-validator)      │   │
│  │  • FileUpload (multer)                 │   │
│  └─────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────┐   │
│  │            CONTROLLERS                  │   │
│  │  • Apenas adaptação HTTP               │   │
│  │  • Logging estruturado                 │   │
│  │  • Tratamento de resposta              │   │
│  └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────┐
│                    ENTITIES                      │
│  • Lógica de negócio pura                      │
│  • Validações complexas                        │
│  • Interação com banco de dados               │
│  • Retorno estruturado padronizado            │
└─────────────────────────────────────────────────┘
```

---

## 📁 **ESTRUTURA DE ARQUIVOS IMPLEMENTADA**

```
baseexpress/
├── 📁 config/
│   └── db.js                    ✅ Conexão MongoDB
├── 📁 controllers/              ✅ Camada de apresentação
│   ├── userController.js        ✅ Implementado
│   ├── WorkspaceController.js   ✅ Implementado  
│   ├── ProjectController.js     ✅ Implementado
│   ├── adminController.js       ✅ Implementado
│   └── FileController.js        ❌ Vazio (pendente)
├── 📁 core/                     ✅ Núcleo da aplicação
│   ├── 📁 access/
│   │   ├── permissions.js       ✅ RBAC completo
│   │   └── roles.js            ✅ Roles admin/user
│   ├── 📁 entities/            ✅ Lógica de negócio
│   │   ├── UserEntity.js       ✅ Implementado
│   │   ├── WorkspaceEntity.js  ✅ Implementado
│   │   ├── ProjectEntity.js    ✅ Implementado
│   │   └── FileEntity.js       ❌ Pendente
│   ├── 📁 logger/
│   │   └── LoggerService.js    ✅ Winston estruturado
│   └── 📁 utils/
│       ├── userUtils.js        ✅ Hash, JWT, compare
│       ├── fileUtils.js        ✅ Google Cloud upload
│       └── XliffParser.js      ✅ Implementado
├── 📁 middlewares/             ✅ Camada intermediária
│   ├── authGuard.js           ✅ JWT verification
│   ├── checkPermission.js    ✅ RBAC middleware
│   ├── handleValidations.js   ✅ Validation handler
│   ├── fileUpload.js         ✅ Multer config
│   ├── imageUpload.js        ✅ Image specific
│   ├── userValidations.js    ✅ User validation rules
│   ├── workspaceValidations.js ✅ Workspace validation
│   └── projectValidations.js ✅ Project validation
├── 📁 models/                 ✅ Schemas MongoDB
│   ├── User.js               ✅ Implementado
│   ├── Workspace.js          ✅ Implementado
│   └── Project.js            ✅ Implementado
├── 📁 routes/                ✅ Definição de rotas
│   ├── Router.js             ✅ Main router
│   ├── UserRoutes.js         ✅ Implementado
│   ├── WorkspaceRoutes.js    ✅ Implementado
│   ├── ProjectRoutes.js      ✅ Implementado
│   ├── AdminRoutes.js        ✅ Implementado
│   └── FileRoutes.js         ❌ Pendente
├── 📁 uploads/               ✅ Storage local
│   └── usersProfile/         ✅ Profile images
├── server.js                 ✅ Express server
└── package.json              ✅ Dependencies
```

---

## 🔧 **FUNCIONALIDADES IMPLEMENTADAS**

### ✅ **1. Sistema de Autenticação**
- **JWT Token**: Geração e verificação
- **Hash de Senha**: bcryptjs
- **Roles**: `user` e `admin`
- **Permissões**: RBAC completo (58 permissões)

### ✅ **2. Gestão de Usuários**
- **CRUD Completo**: Create, Read, Update, Delete
- **Upload de Avatar**: Google Cloud Storage
- **Validações**: Email único, senha forte
- **Logging**: Todas operações logadas

### ✅ **3. Gestão de Workspaces**
- **Criação**: Por usuário autenticado
- **Listagem**: Todos workspaces do usuário
- **Visualização**: Workspace específico por ID
- **Exclusão**: Apenas próprio workspace
- **Validações**: Nome obrigatório, tamanho

### ✅ **4. Gestão de Projetos**
- **Criação**: Dentro de workspace
- **Numeração**: Auto-geração única
- **Idiomas**: Suporte a 60+ idiomas
- **Status**: pending, in_progress, completed, cancelled
- **Progresso**: 0-100%
- **Deadline**: Data opcional
- **Validações**: Completa validação de idiomas

### ✅ **5. Sistema de Logging**
- **Winston**: Logging estruturado
- **Arquivos**: error.log, combined.log
- **Metadados**: IP, User ID, timestamp
- **Níveis**: info, warn, error

### ✅ **6. Validações**
- **express-validator**: Validação robusta
- **Sanitização**: Trim, escape
- **Mensagens**: Português brasileiro
- **Custom Rules**: Validações específicas

### ✅ **7. Upload de Arquivos**
- **Multer**: Upload local temporário
- **Google Cloud**: Storage permanente
- **Tipos**: Imagens de perfil
- **Nomenclatura**: Nomes únicos

### ✅ **8. Parser XLIFF**
- **XML Parsing**: xmldom
- **Extração**: Base64 + segmentos
- **Metadados**: Idioma, versão, arquivo original
- **Validação**: Estrutura XLIFF

---

## 🚧 **FUNCIONALIDADES PENDENTES**

### ❌ **1. Sistema de Arquivos**
- **FileEntity**: Lógica de negócio
- **FileController**: Endpoints HTTP
- **FileRoutes**: Rotas de arquivo
- **Model File**: Schema MongoDB
- **Integração Java**: API Okapi

### ❌ **2. Sistema de Segmentos**
- **SegmentEntity**: Lógica de negócio
- **SegmentController**: Endpoints HTTP
- **SegmentRoutes**: Rotas de segmento
- **Model Segment**: Schema MongoDB
- **WebSocket**: Lock/unlock real-time

### ❌ **3. Sistema de Monetização**
- **Moedas**: Pacotes de crédito
- **Cobrança**: Por operação (MT, IA, humana)
- **Histórico**: Transações
- **Pagamento**: Stripe/integration

### ❌ **4. Sistema de TM (Translation Memory)**
- **Armazenamento**: Pares fonte-destino
- **Busca**: Similaridade fuzzy
- **Escopo**: Por workspace
- **Import/Export**: TMX

### ❌ **5. Integração com IA/MT**
- **APIs**: Google Translate, DeepL, etc.
- **Custo**: Por caractere/palavra
- **Fallback**: Múltiplos provedores
- **Cache**: Resultados

### ❌ **6. Sistema Admin**
- **Dashboard**: Métricas e analytics
- **Usuários**: Gestão completa
- **Financeiro**: Relatórios de receita
- **Sistema**: Logs e monitoramento

---

## 📊 **MODELOS DE DADOS IMPLEMENTADOS**

### **User Schema**
```javascript
{
  name: String,
  email: String (unique),
  password: String (hashed),
  profileImage: String (GCS URL),
  country: String,
  role: "user" | "admin",
  isActive: Boolean,
  permissions: [String],
  createdAt: Date,
  updatedAt: Date
}
```

### **Workspace Schema**
```javascript
{
  userId: ObjectId (ref: User),
  nameWorkSpace: String (1-100 chars),
  createdAt: Date,
  updatedAt: Date
}
```

### **Project Schema**
```javascript
{
  workspaceId: ObjectId (ref: Workspace),
  projectNumber: String (unique),
  projectName: String (1-200 chars),
  subject: String (max 500 chars),
  comments: String (max 1000 chars),
  progress: Number (0-100),
  status: "pending" | "in_progress" | "completed" | "cancelled",
  creator: ObjectId (ref: User),
  deadline: Date,
  languagePair: [String] (exactly 2),
  createdAt: Date,
  updatedAt: Date
}
```

---

## 🔐 **SISTEMA DE PERMISSÕES IMPLEMENTADO**

### **Common User (22 permissões)**
- Account: create, deactivate, edit_own, view_own_profile
- Workspace: view, create, edit_own, delete_own, invite_members, leave
- Project: view, create, edit_own_or_assigned, delete_own, assign_members, interact_tasks

### **Admin User (58 permissões)**
- Todas permissões de usuário comum +
- User Management: view_all, edit_any, delete_any, activate_deactivate, change_role, view_logs, reset_password
- Workspace Management: view_all, edit_any, delete_any, manage_members, archive
- Project Management: view_all, edit_any, delete_any, manage_members, archive
- System Admin: dashboard, logs, export/import, settings, backup, analytics, notifications

---

## 🌐 **ENDPOINTS IMPLEMENTADOS**

### **Users** (`/api/users`)
- `POST /register` - Criar conta
- `POST /login` - Login
- `GET /:id` - Buscar usuário por ID
- `GET /me` - Usuário atual
- `PUT /update` - Atualizar perfil

### **Workspaces** (`/api/workspaces`)
- `POST /create` - Criar workspace
- `GET /all` - Listar workspaces do usuário
- `GET /:id` - Buscar workspace específico
- `DELETE /:id` - Deletar workspace

### **Projects** (`/api/projects`)
- `POST /create` - Criar projeto
- `GET /all` - Listar projetos (com workspaceId)
- `GET /:id` - Buscar projeto específico
- `PUT /:id` - Atualizar projeto
- `DELETE /:id` - Deletar projeto

### **Admin** (`/api/admin`)
- `GET /users` - Listar todos usuários
- `GET /workspaces` - Listar todos workspaces
- `GET /projects` - Listar todos projetos

---

## 🛠️ **TECNOLOGIAS E DEPENDÊNCIAS**

### **Core**
- **Node.js**: Runtime
- **Express**: Web framework
- **MongoDB**: Database
- **Mongoose**: ODM

### **Autenticação & Segurança**
- **jsonwebtoken**: JWT tokens
- **bcryptjs**: Password hashing
- **express-validator**: Input validation

### **Storage & Files**
- **multer**: File upload
- **@google-cloud/storage**: Cloud storage
- **xmldom**: XML parsing

### **Logging & Monitoring**
- **winston**: Structured logging
- **cors**: Cross-origin requests

---

## 📈 **PRÓXIMOS PASSOS PARA DESENVOLVIMENTO**

### **Fase 1: Core Functionality (MVP)**
1. **FileController + FileEntity** - Upload e processamento
2. **SegmentController + SegmentEntity** - Gestão de segmentos
3. **WebSocket Integration** - Real-time collaboration
4. **Java API Integration** - Okapi framework

### **Fase 2: Monetization**
1. **Coin System** - Pacotes e saldo
2. **Billing Logic** - Cobrança por operação
3. **Payment Integration** - Stripe/PagSeguro
4. **Transaction History** - Histórico de gastos

### **Fase 3: AI/MT Integration**
1. **Translation APIs** - Google, DeepL, etc.
2. **Cost Calculation** - Por caractere/palavra
3. **Fallback System** - Múltiplos provedores
4. **Caching Layer** - Otimização de custos

### **Fase 4: Advanced Features**
1. **Translation Memory** - TM por workspace
2. **Admin Dashboard** - Analytics e gestão
3. **Export/Import** - TMX, XLIFF
4. **Collaboration Tools** - Comments, reviews

---

## 🎯 **CRITÉRIOS DE SUCESSO**

### **MVP (Fase 1)**
- ✅ Upload DOCX → XLIFF processado
- ✅ Editor de segmentos funcional
- ✅ Lock/unlock real-time
- ✅ Export de arquivos traduzidos

### **Monetização (Fase 2)**
- ✅ Sistema de moedas operacional
- ✅ Cobrança automática por operação
- ✅ Relatórios de gastos
- ✅ Integração de pagamento

### **Escalabilidade (Fase 3+)**
- ✅ Suporte a múltiplos idiomas
- ✅ TM inteligente
- ✅ Dashboard administrativo
- ✅ Analytics avançados

