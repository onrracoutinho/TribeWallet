# TribeWallet

Sistema completo de gestão financeira colaborativa para grupos, desenvolvido com **.NET 8 (ASP.NET Core)** no backend e **React Native 0.87** no mobile. Permite criar grupos, lançar despesas rateadas, registrar pagamentos com comprovantes, gerar relatórios em PDF e acompanhar tudo em tempo real — do convite à quitação.

---

## 🎯 Sobre o Projeto

Aplicação full-stack para controle de finanças compartilhadas (vaquinhas, contas de casa, viagens, eventos), cobrindo todo o ciclo: criação de grupos, convite de membros, lançamento de compromissos com divisão flexível (igual, porcentagem, valor exato, proporcional), registro de pagamentos (PIX, dinheiro, cartão, transferência, boleto), anexo de comprovantes, notificações automáticas, histórico de auditoria e relatórios gerenciais em PDF.

**Destaques:**
- **Arquitetura limpa** no backend (Domain, Application, Infrastructure, Presentation) com injeção de dependência nativa
- **Autenticação JWT stateless** com refresh token, bloqueio de conta soft-deleted no login
- **Rateio inteligente** de despesas: 4 modos de divisão com cálculo automático de `ValorDevedor` / `ValorPago`
- **Polimorfismo de notificações e auditoria** — uma tabela serve para grupos, compromissos, pagamentos
- **Relatórios em PDF** gerados no backend e servidos via URL assinada (armazenamento externo)
- **Mobile nativo** com 5 abas principais, modais transparentes, animações Reanimated 4, listas virtuais FlashList
- **Estado global** com Zustand + TanStack Query (server state) + MMKV (persistência ultra-rápida)
- **Tema unificado** via Shopify Restyle (light/dark pronto, tokens de cor/espacamento/tipografia)
- **Validação ponta-a-ponta** com Zod + React Hook Form no mobile, DataAnnotations + FluentValidation no backend
- **Observabilidade**: Swagger/OpenAPI, health checks, logs estruturados, error boundaries no app

---

## 🛠 Stack Tecnológica

| Categoria | Tecnologias |
|-----------|-------------|
| **Backend** | .NET 8, ASP.NET Core, C# 12 |
| **ORM & Banco** | Entity Framework Core 8, PostgreSQL, Npgsql, EFCore.NamingConventions (snake_case) |
| **Auth & Segurança** | JWT (System.IdentityModel.Tokens.JWT), BCrypt.Net-Next, DotNetEnv |
| **Documentação API** | Swashbuckle.AspNetCore (Swagger UI + OpenAPI 3) |
| **Migrações** | EF Core Migrations (code-first) |
| **Mobile** | React Native 0.87, React 19, TypeScript 6 |
| **Navegação** | React Navigation 7 (Bottom Tabs + Native Stack) |
| **Estado Server** | TanStack Query v5 (cache, retry, stale-while-revalidate) |
| **Estado Cliente** | Zustand v5 (stores por feature) |
| **Persistência Local** | react-native-mmkv (JSI, síncrono, criptografado) |
| **Estilo & Tema** | @shopify/restyle (tokens, variantes, dark mode ready) |
| **Animações** | Reanimated 4, react-native-gesture-handler 3 |
| **Listas Performáticas** | @shopify/flash-list v2 |
| **Formulários** | React Hook Form 7 + @hookform/resolvers + Zod 4 |
| **SVG & Ícones** | react-native-svg 15 |
| **Safe Area & Device** | react-native-safe-area-context, react-native-iphone-x-helper |
| **Testes** | Jest 29 + @testing-library/react-native + @testing-library/jest-native |
| **Qualidade** | ESLint (Airbnb/React Native), Prettier, TypeScript strict |
| **CI/CD** | GitHub Actions (build, test, lint, deploy preview) |

---

## ✨ Funcionalidades Principais

### Autenticação & Onboarding
Cadastro com validação de e-mail/username únicos, login com JWT, renovação silenciosa de token, bloqueio de contas excluídas (soft delete), tela de recuperação de senha (fluxo backend via token).

### Grupos
Criação com nome, descrição, cor (tema: azul/verde/família) e tags. Lista de membros com avatar/iniciais, resumo financeiro (total aberto / total pago / pontualidade). Convite por link/token externo (SHA-256 64 chars). Edição e exclusão em cascata.

### Compromissos (Despesas Compartilhadas)
Lançamento com título, valor total (decimal), data, categoria, anexo de nota fiscal (URL externa). **4 modos de divisão:**
- **Igual** — rateio automático entre todos os integrantes
- **Porcentagem** — cada integrante define % da despesa
- **Valor Exato** — valores fixos por integrante
- **Proporcional** — peso customizado por integrante
Cálculo automático de `ValorDevedor` e sincronismo com `ValorPago` via pagamentos. Status derivado: `Pendente` | `Parcial` | `Quitado` | `Vencido`.

### Pagamentos
Registro de quitação com valor, data, método (PIX, Dinheiro, Cartão Crédito/Débito, Transferência, Boleto), anexo de comprovante (URL). Atualização automática do `ValorPago` no `IntegranteCompromisso`. Webhook-ready para confirmação automática via gateway (Asaas/Mercado Pago preparado).

### Notificações Inteligentes
Sistema polimórfico: `Entidade` + `EntidadeId` apontam para Grupo, Compromisso, Pagamento, etc. Tipos: Convite, Novo Compromisso, Pagamento Registrado, Cobrança Pendente, Compromisso Quitado, Alteração. Contador de não-lidas no header e badge nas abas.

### Relatórios & Exportação
Geração de PDF no backend (QuestPDF ou similar) com 4 tipos: Detalhes do Compromisso, Resumo do Grupo, Extrato Individual, Pendências. Armazenamento em bucket externo (S3/MinIO/Azure Blob), URL assinada retornada ao mobile. Histórico de relatórios gerados por usuário.

### Auditoria Completa
`HistoricoAlteracao` registra JSON antes/depois de toda operação (Criacao, Atualizacao, Exclusao) em qualquer entidade. Rastreabilidade total para compliance e debug.

### Perfil & Configurações
Edição de nome, username, foto (upload para storage externo), preferências de notificação (push/email), tema, idioma, exclusão de conta (soft delete com anonimização LGPD).

### Dashboard (Home)
Visão consolidada: grupos ativos, compromissos em aberto, próximos vencimentos, pagamentos recentes, estatísticas de pontualidade (widget estilo "4 Grupos / 12 Pagamentos / 87% Pontualidade").

---

## 🏗 Arquitetura (Resumo)

### Backend — Clean Architecture + Vertical Slice por Feature
```
TribeWallet.Backend/
├── Domain/                 # Entidades puras, enums, value objects, eventos de domínio
│   └── Entities/           # Usuario, Grupo, CompromissoFinanceiro, Integrante, IntegranteCompromisso,
│                           # Pagamento, Notificacao, Relatorio, TokenExterno, HistoricoAlteracao, Enums
├── Application/            # Casos de uso, DTOs, interfaces (ports)
│   └── Usuario/            # UsuarioService, IUsuarioRepository, DTOs (Login, Create, Return)
├── Infrastructure/         # Implementações concretas (adapters)
│   ├── UsuarioRepositoryImplementation.cs
│   ├── Data/               # AppDbContext, ConnectionString, Migrations, Seeder
│   └── Services/           # EmailService, StorageService, PdfGeneratorService, PaymentGatewayService
├── Presentation/           # Controllers, Middleware, Filters
│   └── UsuarioController.cs
└── Program.cs              # Composition root, DI, pipeline, Swagger, EF Core, Env loading
```
- **Domain-Driven**: Entidades ricas, value objects (`TokenExterno.Gerar()` com `RandomNumberGenerator` + SHA-256), enums semânticos
- **Repository Pattern** com interface no Application, implementação no Infrastructure (testável, trocável)
- **Service Layer** isolando regras de negócio (rateio, notificações, PDF, gateways)
- **Middleware centralizado**: exception handling, request logging, correlation ID, rate limiting
- **Configuração single-source**: `ConnectionString.Montar()` lê do `.env` + variáveis de ambiente, snake_case automático
- **Migrations & Seed** controlados por variável de ambiente (desligáveis em produção)

### Mobile — Feature-First + Atomic Design
```
TribeWalletNative/
├── src/
│   ├── components/         # Átomos/UI Kit (Button, Card, Input, Modal, Pill, List, Loading, EmptyState, ScreenHeader, TopBar, SideMenu)
│   ├── contexts/           # Providers globais (Theme, Auth, App)
│   ├── features/           # Vertical slices por domínio
│   │   ├── auth/           # Login, Register, ForgotPassword, stores, services, validations, types
│   │   ├── grupos/         # CRUD grupos, membros, convites, detail modal
│   │   ├── compromissos/   # CRUD despesas, split modes, detail modal, new modal
│   │   ├── pagamentos/     # Registro, comprovante, lista por compromisso/grupo
│   │   ├── relatorios/     # Geração, lista, download, compartilhamento
│   │   ├── usuario/        # Perfil, edição, histórico, suporte
│   │   ├── configuracoes/  # Preferências, tema, conta
│   │   └── dashboard/      # Home, widgets, alertas
│   ├── navigation/         # AppNavigator (tabs + stack), AuthNavigator, TabBar custom, types
│   ├── services/           # apiClient (fetch + interceptors + timeout + error mapping), storage (MMKV)
│   ├── config/             # environment, api endpoints
│   ├── theme/              # Restyle tokens (cores, spacing, tipografia, breakpoints)
│   ├── utils/              # formatters, date helpers, validation helpers
│   └── types/              # Contratos globais (ApiResponse, transactionStatus, etc.)
├── android/                # Build nativo Gradle (Assinatura, ProGuard, Play Console ready)
├── ios/                    # (Estrutura pronta para Xcode/CocoaPods)
└── App.tsx                 # Root: Providers (QueryClient, Theme, SafeArea, GestureHandler) + RootNavigator
```
- **Navegação híbrida**: 5 Bottom Tabs (Home, Grupos, Compromissos, Pagamentos, Relatórios) + Stack modais transparentes para criar/editar/detalhar
- **Side Menu** (Drawer-style custom) para ações globais: Perfil, Editar Perfil, Histórico, Suporte, Configurações, Alertas, Logout
- **Estado desacoplado**: `authStore` (sessão, user), `userStore` (perfil, stats), `groupStore`, `commitmentStore`, `paymentStore`, `reportStore`, `preferencesStore` — todos Zustand com persistência seletiva no MMKV
- **Server State**: TanStack Query com `staleTime: 60s`, `retry: 2`, `refetchOnWindowFocus: true`, invalidação por mutation keys
- **API Client** tipado: interceptors de auth (JWT), timeout configurável, mapeamento de erros ASP.NET (ProblemDetails + texto puro), auto-logout no 401
- **Formulários**: React Hook Form + Zod schemas por feature (login, registro, grupo, compromisso, pagamento)
- **Validação visual**: `Input` com erro inline, `Select` nativo, `Modal` com focus trap, `FlashList` para listas longas
- **Acessibilidade**: `accessibilityLabel`, `accessibilityRole`, `accessibilityState` em componentes base; suporte a Dynamic Type

---

## 🎯 Competências Demonstradas

- **Backend .NET Moderno**: Clean Architecture, EF Core 8 (snake_case, value converters, owned types), JWT stateless, middleware pipeline, DI nativo, configuração por ambiente, health checks, OpenAPI
- **Domain Modeling**: Entidades ricas, enums semânticos, value objects criptográficos (`TokenExterno`), polimorfismo via `Entidade`/`EntidadeId` (notificações, auditoria), decimal para dinheiro, soft delete
- **Mobile React Native Avançado**: Arquitetura feature-first, navegação complexa (tabs + stack + modais transparentes + side menu), estado dual (Zustand + TanStack Query), animações Reanimated 4, listas virtualizadas FlashList, MMKV JSI
- **TypeScript Strict**: Tipagem ponta-a-ponta (API contracts ↔ mobile types), discriminated unions para status, branded types para IDs, utility types para DTOs
- **UX/UI System**: Design system próprio com Restyle (tokens, variantes, dark mode), componentes compostos acessíveis, feedback visual (loading, empty, error, success), gestos nativos
- **Integração Real**: API client resiliente (timeout, abort, retry, error mapping), upload de arquivos (presigned URL), webhooks prontos, geração de PDF server-side
- **Observabilidade & Qualidade**: Swagger, logs estruturados, error boundaries, testes unitários/integração (Jest + RTL), lint/format CI, typecheck rigoroso
- **DevOps Ready**: Dockerfile multi-stage (backend + frontend), GitHub Actions (build, test, lint, security scan, deploy staging), variáveis de ambiente por ambiente, assinatura Play Store / App Store configurada

---

## 📦 Como Executar (Resumo)

### Backend
```bash
cd Backend
cp .env.example .env          # configure ConnectionStrings, Jwt:Key, Jwt:Issuer, Storage:, Email:
dotnet restore
dotnet ef database update     # aplica migrations
dotnet run                    # http://localhost:5000 + Swagger em /swagger
```

### Mobile
```bash
cd TribeWalletNative
cp .env.example .env          # configure API_URL, ENV=development
npm install
npm run android               # ou: npx react-native run-android
# npm run ios                 # macOS + Xcode + CocoaPods
```

> **Variáveis de ambiente** são carregadas via `DotNetEnv` (backend) e `react-native-config` / `expo-constants` (mobile). Nunca commite `.env` real.

---

## 📄 Licença

Projeto privado — código não público. Este README serve para fins de portfólio e demonstração de competências técnicas.

---

## 🤝 Contato

**Desenvolvedor Full Stack .NET / React Native**

- LinkedIn: [Gabriel José Silva](https://linkedin.com/in/gabriel-josé-silva-060103255)
- GitHub: [Onrracoutinho](https://github.com/onrracoutinho)
- E-mail: canalgabrieljose@gmail.com

> *Disponível para oportunidades remotas/presenciais (Brasil/Europa).*