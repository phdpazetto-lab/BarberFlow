# 🚀 PLANEJAMENTO EXPANDIDO PARA O CODEX

### **(Backlog estruturado + Guidelines para execução + Critérios de Aceite)**

Este documento estende o *SPEC_DRIVEN.md* com um planejamento mais detalhado e executável pelo Codex.

---

# 🧭 **1. Objetivo do Documento**

Criar um **backlog técnico claro, granular e acionável** para que o Codex possa:

* criar tarefas corretamente
* gerar código com consistência
* evitar ambiguidade
* trabalhar fase por fase
* garantir que cada módulo entregue esteja funcional

---

# 🧱 **2. Estrutura de Fases (Expandida)**

A estrutura original (Fase 0 → Fase 4) é mantida, mas agora tem mais clareza, tarefas e dependências explícitas.

---

# 🟩 **FASE 0 — Infraestrutura & Multi-Tenant (BLOQUEADORA)**

> **Objetivo:** Criar fundação para que *todas* as próximas fases sejam possíveis.

### 🎯 Entregáveis obrigatórios:

* Supabase com RLS + Policies multi-tenant funcionando
* Estrutura inicial do repositório
* Variáveis de ambiente configuradas
* n8n conectado ao Supabase, OpenAI e provedores
* Função `identifyBarbershop(channel)` concluída

### 📌 **Backlog**

#### **0.1 — Estrutura do repositório**

* [ ] Criar pastas:

  * `/apps/web-admin`
  * `/apps/app-client`
  * `/services/n8n`
  * `/infra/supabase`
* [ ] Criar arquivo `.env.example`
* [ ] Criar `README_SETUP.md`

#### **0.2 — Supabase inicial**

* [ ] Criar tabelas conforme SPEC
* [ ] Criar PKs, FKs, indexes
* [ ] Criar policies RLS para cada tabela
* [ ] Criar seeds iniciais (opcional)

#### **0.3 — Função de identificação de barbearia**

Criar função:

```
getBarbershopByChannel(channel_identifier TEXT)
```

Retorno:

```
{
  barbershop_id: uuid,
  config: jsonb
}
```

#### **0.4 — Integração n8n**

* [ ] Criar credencial Supabase
* [ ] Criar credencial OpenAI
* [ ] Criar credencial WhatsApp
* [ ] Criar credencial Instagram
* [ ] Criar webhook inicial de teste

---

# 🟩 **FASE 1 — MVP Funcional (WhatsApp + App Cliente + Painel)**

> **Objetivo:** Usuário consegue **agendar totalmente** via WhatsApp e via App.

## 🔥 O MVP só é considerado finalizado quando:

1. O WhatsApp recebe mensagem
2. O fluxo IA funciona
3. A intenção é interpretada
4. O agendamento é criado no Supabase
5. Cliente recebe confirmação
6. O painel da barbearia exibe a agenda
7. O app lista horários + permite agendar

---

# 📌 **Backlog detalhado — FASE 1**

## **1.1 — Fluxo WhatsApp → IA → Agendamento (n8n)**

### Tasks

* [ ] Criar n8n workflow `wa_inbound`
* [ ] Normalizar evento recebido
* [ ] Chamar função `getBarbershopByChannel`
* [ ] Carregar `bot_configs` e concatenar com prompt base
* [ ] Chamar IA (OpenAI Chat)
* [ ] Extrair intenção JSON
* [ ] Validar intenção
* [ ] Consultar disponibilidade (Supabase)
* [ ] Criar agendamento
* [ ] Responder WhatsApp
* [ ] Criar logs no Supabase: `chatbot_logs`

### Critérios de Aceite

* Mensagens reais disparadas via WhatsApp são respondidas pela IA
* JSON de intenção é sempre validado
* Conflitos de horário são bloqueados
* Logs aparecem em `chatbot_logs`

---

## **1.2 — Lógica Backend (Supabase)**

### Tasks

* [ ] Criar função SQL `checkAvailability(service_id, date, time)`
* [ ] Criar função SQL `bookAppointment(payload json)`
* [ ] Criar função SQL `listAvailableSlots(date)`
* [ ] Garantir integridade referencial
* [ ] Criar `unique index` para evitar agendamento duplicado

### Critérios de Aceite

* Nenhuma sobreposição de horário é possível
* Slots indisponíveis não são retornados
* Erros aparecem como JSON amigável para o n8n

---

## **1.3 — App Cliente (PWA)**

### Tasks

* [ ] Login via Magic Link (Supabase Auth)
* [ ] Listagem de serviços
* [ ] Listagem de horários disponíveis
* [ ] Criação de agendamento
* [ ] Cancelamento
* [ ] Histórico
* [ ] UI responsiva com Tailwind ou Shadcn

### Critérios de Aceite

* App funciona no celular e no desktop
* Agendamento pelo app aparece no painel
* Cancelamento sincroniza com banco

---

## **1.4 — Painel Barbearia**

### Tasks

* [ ] Dashboard de hoje
* [ ] Agenda semanal
* [ ] CRUD Serviços
* [ ] CRUD Barbeiros
* [ ] Configuração do Chatbot (texto + tom de voz)
* [ ] Autenticação via Supabase

### Critérios de Aceite

* Dono consegue editar serviços
* Dono consegue ver agenda
* Configuração do chatbot reflete no fluxo real

---

# 🟩 **FASE 2 — Instagram + UX**

### Backlog

* [ ] Criar fluxo IG → IA → agendamento (mesma estrutura WA)
* [ ] Criar verificação de mídia (fotos enviadas)
* [ ] Melhorar responsividade do painel
* [ ] Adicionar componentes visuais (Skeleton, Toasts)

Critérios de Aceite:

* Agendamentos via Instagram funcionam 100%
* IA entende imagens de corte (se habilitado)

---

# 🟩 **FASE 3 — Pagamentos + Fidelidade**

### Backlog

* [ ] Criar endpoint de gerar QRCode PIX
* [ ] Associar pagamento ao agendamento
* [ ] Criar tabela `loyalty_points`
* [ ] Criar regra de pontos (exemplo: R$1 = 1 ponto)
* [ ] Criar página “Meus Pontos” no app

---

# 🟩 **FASE 4 — Analytics + Escala**

### Backlog

* [ ] Criar tabela `analytics_events`
* [ ] Registrar métricas (no-show, fluxo, origem)
* [ ] Criar painel de métricas para barbearias
* [ ] Criar painel geral para você (admin SaaS)
* [ ] Criar caching de slots de agenda
* [ ] Suporte multi-idioma

---

# 🟦 **3. Formato recomendado para cada Tarefa do Codex**

Para o Codex trabalhar perfeitamente, cada tarefa deve seguir o modelo:

```yaml
task:
  name: "Criar função SQL: listAvailableSlots"
  description: |
    Criar função SQL no Supabase que devolve os slots disponíveis em um dia,
    considerando duração do serviço, disponibilidade do barbeiro e horários
    já ocupados.
  input_example: |
    date: "2025-12-12"
    service_id: "uuid"
  output_example: |
    [
      "09:00",
      "09:30",
      "10:00",
      "14:00",
      "15:30"
    ]
  acceptance_criteria:
    - Função existe no Supabase
    - Retorna slots em ordem crescente
    - Bloqueia horários dentro de serviços em andamento
    - Impede agendamento antes da data atual
```

---

# 🟪 **4. Instruções Gerais para o Codex**

Inclua este bloco no arquivo:

```
O Codex deve seguir estas regras:

1. Sempre avaliar dependências da tarefa antes de iniciar o código.
2. Sempre gerar código completo e funcional, sem trechos omitidos.
3. Sempre testar localmente (quando aplicável) usando mocks DE SUPABASE.
4. Nunca criar tabelas fora do SPEC_DRIVEN.md sem aprovação.
5. Usar padrões consistentes de nomes (snake_case no SQL, camelCase no JS).
6. Sempre sugerir melhorias se perceber inconsistências no SPEC.
7. Se a tarefa estiver ambígua, o Codex deve pedir clarificação.
```

---

# 🎯 **5. O que você deve fazer agora**

Adicionar no repositório:

* `PLANEJAMENTO_EXPANDIDO.md` → usando todo o conteúdo acima
* manter o `SPEC_DRIVEN.md` como fonte técnica
* configurar o Codex para ler ambos os arquivos

E você terá um ambiente **profissional**, igual ao de equipes com Tech Leads reais.

---

# 🔥 Quer que eu gere este arquivo automaticamente para download em .md?

Se quiser, posso gerar agora como fiz com o SPEC.
