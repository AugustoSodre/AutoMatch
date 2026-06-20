# 🚗 AutoMatch: Ecossistema Completo

O **AutoMatch** é uma plataforma inteligente de recomendação de veículos que une uma interface moderna, um backend robusto e um motor de IA avançado baseado em Deep Learning.

---

## 🏗️ Arquitetura Geral

O ecossistema é distribuído em três repositórios principais que se comunicam de forma integrada:

```text
┌─────────────────────────────────────────────────────────────┐
│                     🚗 AutoMatch Frontend                   │
│                   (Angular + SCSS + RxJS)                   │
│                         PORT: 4200                          │
└────────────────────┬────────────────────────────────────────┘
                     │ HTTP REST API
┌────────────────────▼────────────────────────────────────────┐
│                     ⚙️ AutoMatch Backend                    │
│                 (Node.js + Express + Prisma)                │
│                         PORT: 3000                          │
└────────────────────┬────────────────────────────────────────┘
                     │ Fetch API
┌────────────────────▼────────────────────────────────────────┐
│                    🧠 AutoMatch AI Engine                   │
│                 (Python + FastAPI + PyTorch)                │
│                         PORT: 8000                          │
└─────────────────────────────────────────────────────────────┘

```

---

## 📦 Detalhamento dos Repositórios

### 1. 🚗 AutoMatch-Front (Frontend)

Responsável pela experiência do usuário, entregando uma interface premium, responsiva e dinâmica.

* **Localização:** `AutoMatch-Front/`
* **Recursos:** Novo Match Wizard (quiz de 4 etapas), Dashboard de resultados com *match percentage*, UI premium e integração em tempo real via REST API.
* **Stack:** Angular 14+, SCSS, Bootstrap 5, Reactive Forms, RxJS Observables.
* **Servidor de Desenvolvimento:** `http://localhost:4200`

### 2. ⚙️ AutoMatch-Back (Backend Orquestrador)

O coração da plataforma. Gerencia regras de negócio, dados e orquestra o fluxo de informações entre o cliente e o motor de IA.

* **Localização:** `AutoMatch-Back/`
* **Recursos:** Autenticação (JWT + bcrypt), gestão de usuários, catálogo de veículos (CRUD), orquestração de recomendações e API RESTful segura.
* **Stack:** Node.js v22+, Express.js, TypeScript, Prisma 5, SQLite.
* **Servidor de Desenvolvimento:** `http://localhost:3000`
* **Endpoints Principais:** `/api/auth/register`, `/api/auth/login`, `/api/cars`, `/api/matches/recommendations`.

### 3. 🧠 Deep-Learning-Match-Automatch (Motor de IA)

Microsserviço especializado que processa o *Two-Tower Model* para calcular a afinidade exata entre usuários e veículos.

* **Localização:** `Deep-Learning-Match-Automatch/`
* **Stack:** Python 3.9+, FastAPI, PyTorch, Uvicorn.
* **Servidor de Desenvolvimento:** `http://localhost:8000` (Docs: `http://localhost:8000/docs`)
* **API Principal:** `POST /match` (Recebe usuário e lista de carros; retorna o ranking).

**Composição do Scoring Híbrido (Modelo Two-Tower):**

| Métrica Principal | Peso Total | Composição do Peso |
| --- | --- | --- |
| **Similaridade de Cosseno** | 38% | Análise vetorial direta entre as *Towers* de Usuário e Carro. |
| **Preference Boost** | 62% | Uso Principal (34%), Categoria (28%), Ambiente (22%), Grupo/Família (10%), Faixa de Ano (6%). |

---

## 🚀 Como Iniciar o Ambiente de Desenvolvimento

Siga a ordem abaixo para garantir que os serviços dependentes estejam disponíveis.

**Pré-requisitos:** Node.js (v18+), npm/yarn, Python (3.9+) e Git.

### Passo 1: Motor de IA

```bash
cd Deep-Learning-Match-Automatch
python -m venv venv
source venv/bin/activate  # No Windows: venv\Scripts\activate
pip install -r requirements.txt
python main.py

```

### Passo 2: Backend Orquestrador

Crie um arquivo `.env` na raiz do backend contendo `DATABASE_URL="file:./dev.db"`, `JWT_SECRET="sua_chave"` e `AI_SERVICE_URL="http://localhost:8000"`.

```bash
cd AutoMatch-Back
npm install
npx prisma migrate dev
npm run db:seed  # Opcional: popular dados iniciais
npm run dev

```

### Passo 3: Frontend

```bash
cd AutoMatch-Front
npm install
npm start

```

---

## 📊 Fluxos de Dados Principais

### Fluxo de Recomendação (Novo Match)

1. Usuário preenche o Quiz de 4 etapas no Frontend.
2. Frontend envia o `UserProfile` via `POST /api/matches/recommendations`.
3. Backend valida os dados e resgata a lista de carros no banco de dados.
4. Backend encaminha o pacote (Usuário + Carros) para a IA (Python).
5. Motor de IA processa o *Two-Tower* e aplica o *Preference Boost*.
6. Backend recebe os *scores* ordenados e persiste o melhor *match*.
7. Frontend renderiza o Dashboard com as porcentagens de compatibilidade.

### Fluxo de Autenticação

1. Usuário submete credenciais de Login/Registro.
2. Backend valida os dados e processa o *hash* via bcrypt.
3. Backend emite e retorna um Token JWT.
4. Frontend armazena o token no *Storage* do navegador.
5. Requisições subsequentes autenticadas trafegam com o cabeçalho `Authorization: Bearer <token>`.

---

## 📂 Estrutura Simplificada de Diretórios

```text
Automatch-Oficial/
├── AutoMatch-Back/                 # Node.js/Express
│   ├── prisma/                     # Schema, migrations e seeds
│   └── src/
│       ├── lib/                    # Instâncias globais (Prisma)
│       ├── middleware/             # Interceptadores (Auth, Errors)
│       └── routes/                 # Endpoints da API
├── AutoMatch-Front/                # Angular
│   └── src/
│       ├── app/
│       │   ├── auth/               # Login e Registro
│       │   ├── matches/            # Dashboard
│       │   ├── new-match-wizard/   # Componentes do Quiz
│       │   └── systems-services/   # Integrações HTTP
│       └── environments/           # Configurações de API URL
└── Deep-Learning-Match-Automatch/  # Python/FastAPI
    └── app/
        ├── api/                    # Rotas do FastAPI
        ├── models/                 # Redes Neurais PyTorch
        └── utils/                  # Scripts de normalização

```

---

## 🔐 Variáveis do Quiz (Novo Match)

Os dados capturados formam a base do vetor do usuário:

| Etapa | Contexto | Variáveis Capturadas |
| --- | --- | --- |
| **1. Perfil & Uso** | Hábitos e necessidades | Tamanho do grupo (2, 3-4, 5+), Uso (Trabalho, Passeio, Ambos), Ambiente (Urbano, Rural, Ambos). |
| **2. Financeiro** | Poder de compra | Orçamento máximo (R$). |
| **3. Preferências** | Fatores técnicos | Categoria (Hatch, Sedan, SUV, Picape, Elétrico, Premium), Ano (Faixas), Câmbio (Manual, Auto, Ambos). |
| **4. Prioridades** | Afinação final | Peso para Economia (1 a 5) e Potência (1 a 5). |

---

## 🎯 Próximos Passos (Roadmap)

* [ ] Realizar ajustes de pesos e calibração no modelo *Two-Tower*.
* [ ] Implementar camada de cache para acelerar recomendações recorrentes.
* [ ] Desenvolver interface de histórico de matches salvos pelo usuário.
* [ ] Criar sistema de *feedback* (avaliações) para retreinar o modelo de IA contínuamente.
* [ ] Configurar pipelines de CI/CD para deploy em produção (AWS / Railway).

**Dúvidas específicas?** Consulte os documentos `README.md` presentes na raiz de cada um dos três repositórios.