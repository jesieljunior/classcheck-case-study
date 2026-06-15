# ClassCheck — Sistema de Controle de Presença e Análise de Evasão

> Plataforma desenvolvida para o **Instituto da Oportunidade Social (IOS) / Percorre** com o objetivo de digitalizar o controle de frequência de alunos, identificar padrões de evasão e apoiar decisões pedagógicas com dados reais.

**Em produção em 5 estados brasileiros: SP · MG · RS · RJ · PE**  
Desenvolvimento iniciado em 2024. Implantado em produção no 1º semestre de 2026.

---

## O Problema

Antes do ClassCheck, o controle de presença no IOS era feito manualmente — planilhas, papéis e registros descentralizados. Isso tornava impossível:

- Saber em tempo real quantos alunos estavam evadindo
- Identificar **em qual período do semestre** a evasão era mais crítica
- Entender **por que** alunos abandonavam o curso (transporte? renda? localização?)
- Agir preventivamente antes que o aluno desistisse

A instituição precisava de um sistema que transformasse chamada diária em inteligência pedagógica.

---

## Minha Atuação

Este projeto foi desenvolvido por mim de forma independente, da concepção ao deploy em produção.

**Fui responsável por:**

- **Levantamento de requisitos** — entrevistas com instrutores, pedagogos e gestores para entender o fluxo real de trabalho da instituição
- **Arquitetura da solução** — definição das camadas, escolha de tecnologias e modelagem do banco de dados
- **Backend completo** — desenvolvimento de toda a API REST em Python com FastAPI, incluindo autenticação JWT, RBAC com 5 perfis de usuário, regras de negócio, validações e endpoints de relatórios
- **Modelagem de dados** — estrutura das collections no MongoDB, índices de performance, migração de dados em produção e controle de compatibilidade entre versões do schema
- **Integrações** — GridFS para armazenamento de arquivos, exportação de CSV com streaming para grandes volumes, workflow de aprovação de alterações
- **Deploy e infraestrutura** — configuração do ambiente de produção no Render (backend) e Vercel (frontend), variáveis de ambiente, CORS e gestão de segredos
- **Evolução contínua** — manutenção, correção de bugs em produção e implementação de novas funcionalidades com base no feedback dos usuários reais

**Sobre o frontend:**  
A interface foi construída em React com Tailwind CSS e shadcn/ui. A velocidade de desenvolvimento do frontend contou com apoio de ferramentas de IA para geração de componentes visuais. A arquitetura, a integração com a API, as regras de exibição por perfil e a lógica de negócio no frontend foram definidas e implementadas por mim.

---

## A Solução

O ClassCheck é uma plataforma web completa que:

1. **Automatiza a chamada diária** — instrutores registram presença digitalmente, com alertas para chamadas pendentes
2. **Analisa padrões de evasão** — identifica períodos críticos por semestre, turma e unidade
3. **Cruza dados com IA** — um agente analisa se a evasão tem correlação com região, renda familiar, distância do transporte público e outros fatores socioeconômicos por região geográfica
4. **Gera relatórios exportáveis** — CSV com frequência por aluno, turma, unidade e período
5. **Controla acesso por perfil** — 5 perfis com permissões específicas e isolamento de dados entre unidades

---

## Resultados e Impacto

### Transformação do processo

| Antes | Depois |
|---|---|
| Chamada em papel ou planilha local | Registro digital em tempo real |
| Sem visibilidade de evasão | Dashboard com taxa de presença por turma |
| Motivos de desistência não rastreados | 14 categorias padronizadas + campo livre |
| Relatórios feitos manualmente | Exportação de CSV em segundos |
| Sem alerta de chamadas esquecidas | Notificações automáticas de pendências |

### Valor gerado por perfil

**Para gestores e coordenadores:**  
Visão consolidada da frequência por unidade, curso e período. Acesso a dados de evasão por região sem necessidade de consolidar planilhas manualmente.

**Para pedagogos:**  
Identificação rápida de alunos com frequência abaixo de 75%, com histórico de faltas e justificativas centralizados. Suporte ao acompanhamento individual sem depender de relatórios de terceiros.

**Para instrutores:**  
Interface simplificada para registro de chamada diária. Alertas automáticos quando uma chamada não foi feita no dia programado. Solicitação de correção retroativa com fluxo de aprovação.

**Para a instituição:**  
Sistema em produção utilizado em 5 estados, substituindo processos manuais fragmentados por um único ponto de verdade sobre a frequência dos alunos.

---

## Arquitetura

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND                             │
│              React + Tailwind CSS + shadcn/ui               │
│                   Deploy: Vercel                            │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS / REST API
┌────────────────────────▼────────────────────────────────────┐
│                        BACKEND                              │
│              Python + FastAPI + Motor (async)               │
│                   Deploy: Render                            │
└────────────────────────┬────────────────────────────────────┘
                         │ Motor (async driver)
┌────────────────────────▼────────────────────────────────────┐
│                      BANCO DE DADOS                         │
│                   MongoDB Atlas (Cloud)                     │
│          Collections: turmas, alunos, attendances,          │
│          usuarios, desistentes, justifications, etc.        │
└─────────────────────────────────────────────────────────────┘
```

### Decisões técnicas

**Por que FastAPI?**  
FastAPI oferece validação automática de dados via Pydantic, documentação OpenAPI gerada automaticamente e suporte nativo a operações assíncronas. Para um sistema com múltiplas rotas, perfis de usuário e regras de negócio complexas, a tipagem forte e a organização por routers foi essencial para manter o código sustentável ao longo do tempo.

**Por que MongoDB?**  
O modelo de dados do ClassCheck é naturalmente hierárquico: uma turma tem alunos, que têm chamadas, que têm registros individuais de presença. O MongoDB permite representar essa estrutura sem joins complexos. Além disso, a flexibilidade de schema facilitou migrações incrementais em produção sem downtime — um requisito real dado que o sistema evoluiu enquanto estava em uso.

**Por que Motor (async)?**  
Com múltiplos usuários simultâneos fazendo chamadas e consultando relatórios, operações de I/O bloqueantes seriam um gargalo. O Motor permite que o FastAPI processe outras requisições enquanto aguarda respostas do banco, resultando em melhor performance sem necessidade de escalonamento horizontal.

**Streaming de CSV:**  
Relatórios com grande volume de dados (múltiplas turmas, semestres inteiros) causavam timeout na conexão quando gerados de forma síncrona. A solução foi implementar `StreamingResponse` no FastAPI, enviando o CSV linha a linha enquanto os dados são processados — eliminando o problema de timeout mesmo para exportações grandes.

### Stack completa

| Camada | Tecnologia |
|---|---|
| Frontend | React 18, JavaScript, Tailwind CSS, shadcn/ui |
| Backend | Python 3.11, FastAPI, Uvicorn |
| Banco de dados | MongoDB Atlas (Motor async driver) |
| Autenticação | JWT (PyJWT + bcrypt) |
| Armazenamento de arquivos | MongoDB GridFS (atestados médicos) |
| Deploy (frontend) | Vercel |
| Deploy (backend) | Render |
| Relatórios | CSV streaming (anti-timeout para grandes volumes) |

---

## Desafios Técnicos Resolvidos

### 1. RBAC com isolamento de dados por unidade

O sistema tem 5 perfis de usuário (admin, instrutor, pedagogo, monitor, gestor), cada um com visão e permissões distintas. O desafio foi garantir que um instrutor em SP não pudesse ver dados de alunos de MG, mesmo sem filtro explícito no frontend.

A solução foi implementar filtros de permissão diretamente nas queries do backend: cada endpoint calcula o escopo de dados visível baseado no `tipo` e nas propriedades `unidade_id` / `curso_id` do usuário autenticado, antes de executar qualquer consulta ao banco.

### 2. Autenticação JWT com primeiro acesso

Instrutores recebem uma senha temporária gerada no padrão `IOS2026` + iniciais do nome. No primeiro login, o sistema detecta o flag `primeiro_acesso: true` e força a troca de senha antes de liberar acesso à plataforma. O token JWT carrega o perfil do usuário, eliminando consultas extras ao banco a cada requisição.

### 3. Upload e armazenamento de atestados médicos

Justificativas de falta frequentemente vêm com documento comprobatório (PDF, JPG, PNG até 5MB). Armazenar esses arquivos no sistema de arquivos do servidor seria inviável em ambiente cloud com deploy em Render. A solução foi usar MongoDB GridFS como camada de armazenamento, com um endpoint de download que serve os arquivos com os headers corretos de Content-Type e Content-Disposition, respeitando as permissões de quem pode acessar cada arquivo.

### 4. Relatórios de grande volume sem timeout

Relatórios completos cruzam chamadas, alunos, turmas, cursos e unidades — podendo envolver milhares de registros. A geração síncrona causava timeout de gateway (504) nos primeiros testes em produção. 

A solução foi dupla: `StreamingResponse` para exports diretos (envia o CSV linha a linha enquanto processa) e um sistema de jobs em background (`BackgroundTasks` do FastAPI) para exportações muito grandes, onde o frontend verifica o status do job periodicamente até a conclusão.

### 5. Chamadas retroativas com workflow de aprovação

Instrutores às vezes esquecem de fazer a chamada e precisam registrá-la depois. Simplesmente permitir edição retroativa criaria risco de manipulação de dados. A solução foi um workflow de três etapas: o instrutor solicita a alteração com justificativa, o admin analisa e aprova ou nega, e a alteração só é aplicada na chamada original se aprovada. O sistema cria uma notificação para o solicitante com o resultado.

### 6. Migração de schema em produção

Durante o desenvolvimento, a estrutura de `instrutor_id` (campo único) precisou evoluir para `instrutor_ids` (array, suportando até 2 instrutores por turma). Com dados reais já em produção, não era possível fazer uma migração destrutiva.

A solução foi uma função de normalização aplicada na leitura: `parse_from_mongo()` detecta o formato antigo e converte para o novo transparentemente, enquanto todas as escritas novas já usam o formato atualizado. Isso permitiu migrar gradualmente sem downtime e sem quebrar dados existentes.

---

## Funcionalidades

### Controle de Presença
- Registro de chamada diária por turma
- Alertas automáticos de chamadas pendentes (hoje e ontem), baseados nos dias de aula configurados por curso
- Suporte a chamadas retroativas com fluxo de aprovação administrativa
- Registro de justificativas e atestados médicos (upload PDF/JPG/PNG via GridFS)

### Análise de Evasão
- Dashboard com taxa de presença por turma, unidade e período
- Identificação de alunos em risco (abaixo de 75% de presença)
- Registro estruturado de motivos de desistência (14 categorias padronizadas + campo personalizado)
- Reativação de alunos desistentes com fluxo de aprovação

### Agente de IA
- Cruzamento de dados de evasão com informações socioeconômicas por região
- Identificação de correlações entre evasão e fatores como transporte, renda familiar e localização geográfica
- Geração de insights para suporte à equipe pedagógica

### Gestão
- Cadastro de alunos individual ou em massa (CSV/Excel com validação de CPF — formato e dígitos verificadores)
- Gestão de turmas com até 2 instrutores por turma
- Controle de unidades, cursos e usuários com perfis diferenciados
- Exportação de relatórios em CSV (formato simples e completo com estatísticas por aluno)
- Notificações internas com marcação de lidas

### Controle de Acesso (RBAC)

| Perfil | Permissões |
|---|---|
| **Admin** | Acesso total ao sistema, aprovação de solicitações, reset de senhas |
| **Instrutor** | Suas turmas, alunos e chamadas; solicitações de alteração |
| **Pedagogo** | Turmas de extensão da sua unidade; justificativas e desistências |
| **Monitor** | Visualização das turmas da sua unidade; registro de chamadas |
| **Gestor** | Visualização global sem permissão de escrita |

---

## Estrutura do Projeto

```
classcheck/
├── backend/
│   ├── server.py          # API principal — rotas, models, autenticação, RBAC
│   ├── requirements.txt   # Dependências Python
│   └── .env               # Variáveis de ambiente (não versionado)
│
├── frontend/
│   ├── src/
│   │   ├── App.js         # Aplicação principal (componentes React)
│   │   ├── App.css        # Estilos globais
│   │   ├── index.js       # Entry point
│   │   ├── index.css      # Tailwind CSS base
│   │   └── components/    # Componentes shadcn/ui
│   └── package.json
│
└── README.md
```

### Componentes principais (frontend)

| Componente | Responsabilidade |
|---|---|
| `Dashboard` | KPIs e visão geral adaptados por perfil de usuário |
| `ChamadaManager` | Registro e gestão de chamadas diárias |
| `AlunosManager` | CRUD de alunos, upload em massa, histórico de frequência |
| `TurmasManager` | Gestão de turmas, instrutores e alunos |
| `RelatoriosManager` | Exportação de relatórios CSV |
| `UsuariosManager` | Gestão de usuários e permissões |
| `UnidadesManager` / `CursosManager` | Cadastro de unidades e cursos |
| `AttendanceModal` | Interface de chamada diária com lista de alunos |

### Endpoints principais (backend)

| Método | Rota | Descrição |
|---|---|---|
| `POST` | `/api/auth/login` | Autenticação JWT |
| `GET` | `/api/dashboard/stats` | KPIs por perfil de usuário |
| `POST` | `/api/attendance` | Registrar chamada |
| `GET` | `/api/classes/{id}/attendance` | Histórico de chamadas da turma |
| `GET` | `/api/notifications/pending-calls` | Chamadas pendentes |
| `POST` | `/api/students/bulk-upload` | Upload em massa (CSV/Excel com validação de CPF) |
| `GET` | `/api/reports/attendance` | Relatório de frequência (streaming) |
| `GET` | `/api/reports/student-frequency` | Frequência por aluno exportável |
| `POST` | `/api/dropouts` | Registrar desistência com motivo estruturado |
| `GET` | `/api/teacher/stats` | Estatísticas por tipo de usuário |
| `POST` | `/api/attendance-change-requests` | Solicitação de alteração retroativa |
| `PUT` | `/api/attendance-change-requests/{id}/respond` | Aprovação/negação pelo admin |

---

## Screenshots

> *Adicionar capturas de tela após configuração do ambiente de produção*

### Dashboard
![Dashboard](docs/screenshots/dashboard.png)
*Visão geral com taxa de presença, alunos em risco e chamadas do dia — adaptada por perfil de usuário*

### Tela de Chamada
![Chamada](docs/screenshots/chamada.png)
*Interface de registro de presença diária com lista de alunos da turma*

### Gestão de Alunos
![Alunos](docs/screenshots/alunos.png)
*CRUD completo de alunos com histórico de frequência e upload em massa via CSV*

### Relatórios
![Relatórios](docs/screenshots/relatorios.png)
*Exportação de relatórios de frequência por turma, unidade e período*

### Gestão de Usuários
![Usuários](docs/screenshots/usuarios.png)
*Cadastro de usuários com perfis diferenciados e aprovação de primeiro acesso*

---

## Como rodar localmente

### Pré-requisitos
- Python 3.11+
- Node.js 18+
- Conta no MongoDB Atlas (gratuita)

### Backend

```bash
# Clonar repositório
git clone https://github.com/jesiel-amaro/classcheck
cd classcheck/backend

# Criar ambiente virtual
python -m venv venv
source venv/bin/activate  # Linux/Mac
# ou: venv\Scripts\activate  # Windows

# Instalar dependências
pip install -r requirements.txt

# Configurar variáveis de ambiente
cp .env.example .env
# Editar .env com sua MONGO_URL e JWT_SECRET

# Iniciar servidor
uvicorn server:app --reload --port 8000
```

### Frontend

```bash
cd classcheck/frontend

# Instalar dependências
npm install

# Configurar backend URL
echo "REACT_APP_BACKEND_URL=http://localhost:8000" > .env

# Iniciar aplicação
npm start
```

A aplicação estará disponível em `http://localhost:3000`.

---

## Variáveis de ambiente

### Backend (`.env`)
```
MONGO_URL=mongodb+srv://usuario:senha@cluster.mongodb.net/
DB_NAME=IOS-SISTEMA-CHAMADA
JWT_SECRET=sua-chave-secreta-aqui
```

### Frontend (`.env`)
```
REACT_APP_BACKEND_URL=https://seu-backend.onrender.com
```

---

## Deploy em produção

### Frontend → Vercel
1. Conectar repositório no [vercel.com](https://vercel.com)
2. Definir `REACT_APP_BACKEND_URL` nas variáveis de ambiente
3. Deploy automático a cada push na branch `main`

### Backend → Render
1. Criar Web Service no [render.com](https://render.com)
2. Definir `MONGO_URL`, `DB_NAME` e `JWT_SECRET` nas variáveis de ambiente
3. Start command: `uvicorn server:app --host 0.0.0.0 --port $PORT`

---

## Aprendizados

Desenvolver um sistema que entrou em produção real em múltiplas unidades trouxe aprendizados que projetos acadêmicos não trazem.

**Levantamento de requisitos é iterativo.**  
A versão inicial do sistema tinha um campo `instrutor_id` singular por turma. Só depois de conversar com os pedagogos descobri que algumas turmas são co-lecionadas por dois instrutores. Isso gerou uma migração de schema com dados já em produção — um problema completamente diferente de escrever código novo.

**Usuários reais encontram casos de uso que você não previu.**  
O workflow de chamadas retroativas surgiu de uma necessidade real: instrutores que se esqueciam de fazer a chamada no dia e precisavam registrá-la depois. A solução inicial (simplesmente permitir edição) criava risco de manipulação. O workflow de aprovação resolveu o problema de forma mais segura.

**Performance importa quando há dados reais.**  
Relatórios que funcionavam perfeitamente com dados de teste começaram a causar timeout em produção quando as turmas acumularam meses de chamadas. Aprendi a identificar gargalos de I/O, implementar caches em memória dentro do processamento e usar streaming para evitar carregamento de grandes volumes na memória de uma vez.

**Manutenção é parte do desenvolvimento.**  
Manter um sistema em produção significa lidar com bugs que aparecem apenas em condições específicas de dados reais, com usuários que precisam de resposta rápida. Aprendi a priorizar estabilidade, manter logs de diagnóstico e construir endpoints de debug que não expõem dados sensíveis.

**Feedback de usuário acelera mais do que especificação.**  
As funcionalidades mais usadas e mais valorizadas não eram as que eu havia planejado como prioritárias. Observar como instrutores e pedagogos usavam o sistema me mostrou onde estava o valor real — e onde eu havia desenvolvido complexidade desnecessária.

---

## Autor

**Jesiel Amaro Junior**

Desenvolvedor Backend Python com foco em construção de APIs REST, integração de sistemas e automação de processos. Atuo também como instrutor de tecnologia no SENAC e no Percorre (IOS), onde leciono Análise de Dados com IA e Desenvolvimento Web.

O ClassCheck é o projeto que melhor representa minha trajetória: identifiquei um problema real na instituição onde trabalho, projetei e construí a solução do zero, implantei em produção e continuo evoluindo com base no uso real.

**Áreas de atuação:** Backend Python · FastAPI · APIs REST · MongoDB · Integração de Sistemas · Automação de Processos · Análise de Dados · Educação em Tecnologia

[LinkedIn](https://linkedin.com/in/jesiel-amaro-junior) · [GitHub](https://github.com/jesiel-amaro)

---

*Desenvolvido para o Instituto da Oportunidade Social (IOS) / Percorre — 2024 a 2026*
