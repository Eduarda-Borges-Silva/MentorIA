# MentorIA

Plataforma para acelerar estudos através de geração inteligente de flashcards, chat com documentos (MentorAI Assistant) e gerenciamento de coleções. Inclui frontend moderno (React + Vite + Tailwind + shadcn/ui) e backend Node/Express com MongoDB e JWT.

## Sumário
- [Recursos Principais](#recursos-principais)
- [Arquitetura](#arquitetura)
- [Tecnologias](#tecnologias)
- [Pré-requisitos](#pré-requisitos)
- [Instalação](#instalação)
- [Variáveis de Ambiente](#variáveis-de-ambiente)
- [Scripts Disponíveis](#scripts-disponíveis)
- [Estrutura de Pastas](#estrutura-de-pastas)
- [Fluxo de Autenticação](#fluxo-de-autenticação)
- [Endpoints Principais](#endpoints-principais)
- [Contexto do Agente (Agent Context)](#contexto-do-agente-agent-context)
- [Geração de Flashcards](#geração-de-flashcards)
- [Segurança e Boas Práticas](#segurança-e-boas-práticas)
- [Roadmap / Próximos Passos](#roadmap--próximos-passos)
- [Licença](#licença)

## Recursos Principais
- Criação automática de flashcards a partir de texto ou documentos (PDF, CSV, TXT, URLs, YouTube).
- Chat inteligente com documentos (MentorAI Assistant) usando contexto customizável.
- Sistema de coleções e estudo com repetição espaçada (em evolução).
- Página de perfil para editar nome, preferências e senha.
- Planos de assinatura (Free, Premium, Pro) — base para futuras integrações de pagamento.
- Configuração avançada de “Agent Context” (system prompt, personalidade, expertise, instruções).
- Interface moderna com gradientes, animações e componentes acessíveis.

## Arquitetura
```
Frontend (Vite + React + TS) --> API REST (Express) --> MongoDB
                          \--> Auth (JWT) / Contexto IA / Flashcards
```
- Frontend consome a API via fetch, usando token JWT armazenado em `localStorage`.
- Backend expõe rotas REST protegidas por middleware `authenticateToken`.
- MongoDB armazena usuários, conjuntos de flashcards, sessões do Oráculo e logs de auditoria.

## Tecnologias
Frontend:
- React 18, Vite, TypeScript
- TailwindCSS + shadcn/ui + lucide-react
- React Router DOM

Backend:
- Node.js + Express
- MongoDB + Mongoose
- JWT para autenticação
- Joi para validação
- Multer para upload de arquivos
- Helmet + Rate Limit para segurança

Utilidades:
- Scripts `.bat` para iniciar backend e frontend rapidamente no Windows.

## Pré-requisitos
- Node.js (>= 18)
- NPM ou PNPM (frontend usa pnpm-lock.yaml, mas npm funciona)
- MongoDB (Atlas ou local)

## Instalação
Clone o repositório:
```bash
git clone https://github.com/Eduarda-Borges-Silva/MentorIA.git
cd MentorIA
```

Instale dependências do backend:
```bash
cd backend
npm install
```

Instale dependências do frontend:
```bash
cd ../frontend
npm install   # ou pnpm install
```

## Variáveis de Ambiente
Crie um arquivo `.env` na pasta `backend/` com:
```
MONGO_URL=mongodb+srv://usuario:senha@host/banco
JWT_SECRET=uma_chave_segura
JWT_EXPIRES_IN=7d
FRONTEND_URL=http://localhost:5173
GEMINI_API_KEY=chave_api_opcional
```
Outras possíveis futuras:
```
PORT=3000
RATE_LIMIT_WINDOW=15
RATE_LIMIT_MAX=100
```

## Scripts Disponíveis
Backend:
```bash
npm run start        # Inicia servidor em modo produção simples
npm run dev          # (se configurado) Modo desenvolvimento com nodemon
```
Frontend:
```bash
npm run dev          # Inicia Vite em http://localhost:5173
npm run build        # Build de produção
npm run preview      # Servir build local
```
Scripts auxiliares (raiz):
- `start-backend.bat` — atalho para subir backend
- `start-frontend.bat` — atalho para subir frontend

## Estrutura de Pastas (Resumo)
```
backend/
  src/
    authController.js
    authMiddleware.js
    oraculoController.js
    index.js
  models/
    User.js
    FlashcardSet.js
    Conversation.js
    Audit.js
frontend/
  src/
    pages/
      Index.tsx, About.tsx, Pricing.tsx
      Login.tsx, Register.tsx
      app/ (rotas protegidas)
        Profile.tsx
        AgentContextSettings.tsx
        Oraculo.tsx
    contexts/ AuthContext.tsx
    lib/ auth-api.ts, agent-context-api.ts
```

## Fluxo de Autenticação
1. Usuário registra ou faz login (`/api/auth/register` / `/api/auth/login`).
2. Backend retorna `token` JWT e objeto `user` sem senha.
3. Frontend salva `auth_token` e `user_data` em `localStorage`.
4. Para rotas protegidas, frontend envia header `Authorization: Bearer TOKEN`.
5. Middleware `authenticateToken` valida token e injeta `req.user`.

## Endpoints Principais
Autenticação:
```
POST   /api/auth/register       # Registro
POST   /api/auth/login          # Login
POST   /api/auth/logout         # Logout (auditoria)
GET    /api/auth/profile        # Perfil atual
PUT    /api/auth/profile        # Atualizar nome/preferências
PUT    /api/auth/password       # Alterar senha
GET    /api/auth/verify         # Validar token
```
Contexto do Agente:
```
GET    /api/auth/agent-context
PUT    /api/auth/agent-context  # Atualiza systemPrompt, personality, expertise, customInstructions
```
Flashcards:
```
POST   /api/flashcards/gerar                 # Geração a partir de texto
POST   /api/flashcards/salvar                # Salvar coleção (gera se necessário)
GET    /api/flashcards                       # Lista coleções do usuário
POST   /api/flashcards/gerar-de-documento    # Upload de documento para geração
```
Oráculo (Chat):
```
POST   /api/oraculo/inicializar
POST   /api/oraculo/chat
POST   /api/oraculo/chat/stream
GET    /api/oraculo/historico/:sessionId
GET    /api/oraculo/mensagens/:sessionId
GET    /api/oraculo/sessoes
```

## Contexto do Agente (Agent Context)
Estrutura armazenada em `user.preferences.agentContext`:
```json
{
  "systemPrompt": "Base pedagógica...",
  "personality": "friendly",
  "expertise": ["educação", "flashcards"],
  "customInstructions": "Use exemplos simples."
}
```
Validações (backend – Joi):
- `systemPrompt`: até 2000 chars (pode ser vazio)
- `personality`: formal | casual | friendly | professional
- `expertise`: até 10 itens (cada ≤ 50 chars)
- `customInstructions`: até 1000 chars (pode ser vazio)

## Geração de Flashcards
Fluxo:
1. Frontend envia texto/documento para `/api/flashcards/gerar` ou `/api/flashcards/gerar-de-documento`.
2. Backend usa função `gerarFlashcards` (Gemini/Groq ou mock) com `userContext`.
3. Resposta contém array de flashcards e metadata.
4. Usuário pode salvar em `/api/flashcards/salvar` montando coleção própria.

Exemplo de Flashcard:
```json
{
  "question": "O que é repetição espaçada?",
  "answer": "Técnica de revisão em intervalos crescentes para reforço de memória." 
}
```

## Segurança e Boas Práticas
- JWT com expiração configurável (`JWT_EXPIRES_IN`).
- Rate limiting para autenticação: 5 tentativas / 15 min.
- Helmet para cabeçalhos de segurança.
- Auditoria (`Audit`) com TTL (90 dias) e enum de ações (login, update, etc.).
- Validação robusta com Joi para registrar, logar, atualizar perfil e contexto.
- Upload restrito a tipos: `.pdf`, `.csv`, `.txt` (10MB).

## Roadmap / Próximos Passos
- [ ] Integração real com provedores (Stripe/Pagar.me) para planos.
- [ ] Implementar modo estudo (algoritmo de repetição espaçada).
- [ ] Exportação de flashcards (CSV / Anki).
- [ ] Painel de estatísticas de uso (ex: flashcards criados / sessões IA).
- [ ] Internacionalização (i18n) – já há campo `language` em `preferences`.
- [ ] Tema avançado persistente (usar `theme` existente). 
- [ ] Logs e métricas (Prometheus / OpenTelemetry).
- [ ] Testes automatizados (Jest, React Testing Library, Supertest).

## Licença
Projeto acadêmico — definir licença posteriormente (ex: MIT). Enquanto não definido, considerar uso interno/estudantil.

---
Made with 💡 + ⚡ + 📚 para acelerar seus estudos.
