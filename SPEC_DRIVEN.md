# STARCUT – SPEC TÉCNICA
## Chatbot Multicanal + App de Agendamentos para Barbearias

**Versão:** 1.0  
**Responsável:** StarMKT / StarLab – Soluções para Barbearias  
**Data:** 2025-12-09  

---

## 🧭 Sumário

1. [Visão Geral do Produto](#1-visão-geral-do-produto)  
2. [Objetivos do Sistema](#2-objetivos-do-sistema)  
3. [Escopo do MVP](#3-escopo-do-mvp)  
4. [Personas e Canais](#4-personas-e-canais)  
5. [Arquitetura de Alto Nível](#5-arquitetura-de-alto-nível)  
6. [Requisitos Funcionais](#6-requisitos-funcionais)  
7. [Requisitos Não Funcionais](#7-requisitos-não-funcionais)  
8. [Modelagem de Dados – Supabase](#8-modelagem-de-dados--supabase)  
9. [Fluxos de Integração (n8n)](#9-fluxos-de-integração-n8n)  
10. [Lógica de IA e Prompts](#10-lógica-de-ia-e-prompts)  
11. [Aplicativo de Agendamento (Cliente Final)](#11-aplicativo-de-agendamento-cliente-final)  
12. [Painel Web da Barbearia](#12-painel-web-da-barbearia)  
13. [Gestão de Multi-Barbearias (SaaS)](#13-gestão-de-multi-barbearias-saas)  
14. [Segurança, Logs e Monitoramento](#14-segurança-logs-e-monitoramento)  
15. [Roadmap de Evolução](#15-roadmap-de-evolução)  

---

## 1. Visão Geral do Produto

Sistema **StarCut**: plataforma SaaS para barbearias que combina:

- **Chatbot inteligente multicanal** (WhatsApp + Instagram DM)
- **App de agendamentos** para clientes finais
- **Painel web** para donos de barbearia e barbeiros
- **Backend centralizado em Supabase**
- **Orquestração de fluxos via n8n**
- **IA personalizada por barbearia** (com prompts configuráveis e base de conhecimento própria)

Objetivo principal: permitir que barbearias automatizem **atendimento, agendamentos e lembretes**, mantendo o cliente nos canais onde ele já está (WhatsApp/Instagram), mas **conduzindo gradualmente** para o uso do aplicativo próprio.

---

## 2. Objetivos do Sistema

1. **Reduzir carga manual de atendimento** no WhatsApp/Instagram.
2. **Automatizar agendamentos**, cancelamentos e remarcações.
3. **Centralizar dados** de clientes, serviços, horários e histórico.
4. **Oferecer um app simples** para clientes frequentes verem horários e agendarem sozinhos.
5. Permitir que o sistema seja **reutilizável para muitas barbearias** (modelo SaaS multi-tenant).
6. Facilitar **personalização do chatbot** por barbearia (tom de voz, serviços, regras).

---

## 3. Escopo do MVP

### Incluído no MVP

- Receber mensagens via **WhatsApp** e **Instagram DM**.
- Identificar intenções básicas:
  - tirar dúvida (preços, horário de funcionamento, endereço)
  - pedir agendamento
  - remarcar/cancelar
  - falar com humano
- Criar agendamentos no **Supabase**.
- Consultar disponibilidade (dia, horário, barbeiro, serviço).
- App (mobile ou web responsivo) para cliente:
  - ver horários disponíveis
  - criar/cancelar agendamento
- Painel web simples para barbearia:
  - ver agenda do dia
  - ver agendamentos futuros
  - cadastrar serviços
- Personalização de chatbot por barbearia via configuração (sem código).

### Fora do MVP (futuro)

- Pagamento via PIX integrado.
- Programa de fidelidade/pontos.
- Envio de campanhas de marketing automático.
- Dashboard avançado com métricas (no-show, ticket médio, etc.).
- Suporte a múltiplos idiomas.

---

## 4. Personas e Canais

### 4.1 Personas

1. **Cliente da Barbearia (Usuário Final)**
   - Quer agendar rápido, sem burocracia.
   - Usa principalmente **WhatsApp**; alguns usam **Instagram**.
   - Eventualmente passa a usar o **app** para ver horários e promoções.

2. **Dono da Barbearia**
   - Quer reduzir o tempo respondendo mensagens.
   - Quer ter controle da agenda.
   - Não é técnico; precisa de painel simples.

3. **Barbeiro**
   - Quer ver sua agenda diária e semanal.
   - Quer saber quais serviços fará e para quem.

4. **Admin da Plataforma (você)**
   - Configura novas barbearias.
   - Ajusta prompts e regras de IA.
   - Dá suporte e faz on-boarding.

### 4.2 Canais Atendidos

- **WhatsApp**
  - Canal principal de atendimento.
  - Entrada de novos e antigos clientes.

- **Instagram (DM)**
  - Canal de aquisição de novos clientes.
  - Respostas a dúvidas e redirecionamento para agendamentos.

- **Aplicativo / Web App**
  - Usado por clientes recorrentes e engajados.
  - Meio principal para consulta de horários e autoagendamento.

---

## 5. Arquitetura de Alto Nível

### 5.1 Componentes

1. **Frontend – App Cliente**
   - Stack sugerida: React Native (mobile) ou Next.js (SPA responsiva).
   - Funcionalidade: agendamento, consulta, cancelamento, histórico.

2. **Frontend – Painel Barbearia**
   - Stack sugerida: Next.js + Tailwind.
   - Funcionalidade: gestão de agenda, serviços, barbeiros, configurações do chatbot.

3. **Backend – Supabase**
   - Banco relacional (Postgres).
   - Auth (e-mail/senha, magic link).
   - Storage (logos, imagens de barbearia – futuro).
   - RLS (Row-Level Security) para multi-tenant.

4. **Orquestrador – n8n**
   - Recebe webhooks de WhatsApp e Instagram.
   - Encaminha mensagens para IA.
   - Chama Supabase (via HTTP ou node oficial).
   - Envia resposta de volta ao canal.

5. **IA – OpenAI (Assistants / Chat Completion)**
   - Inteligência do chatbot.
   - Prompt base + contexto da barbearia (configuração + base de conhecimento).

6. **Provedores de Mensageria**
   - WhatsApp: Gupshup / 360Dialog / Z-API (não-oficial).
   - Instagram: Meta Graph API (Messenger).

### 5.2 Diagrama Lógico (texto)

1. Cliente envia mensagem no **WhatsApp** ou **Instagram**.  
2. Provedor → envia **webhook** → **n8n**.  
3. n8n cria um contexto:
   - detecta barbearia (por número/canal)
   - recupera config da barbearia no Supabase
4. n8n chama **IA** com:
   - mensagem do usuário
   - instruções do chatbot da barbearia
   - contexto de agenda (opcional)
5. IA responde com:
   - texto para o usuário
   - e, opcionalmente, uma **intenção estruturada** (ex: `{"intent":"agendar","service":"Degradê","date":"2025-12-12","time":"15:00"}`)
6. n8n processa:
   - se houver intenção de agendar → consulta horários → grava no Supabase → confirma para usuário.
7. Supabase guarda todos os dados; app e painel lêem diretamente do Supabase.

---

## 6. Requisitos Funcionais

### 6.1 Chatbot – Core

- RF-01: O sistema deve responder automaticamente mensagens recebidas via WhatsApp.
- RF-02: O sistema deve responder automaticamente mensagens recebidas via Instagram DM.
- RF-03: O chatbot deve entender intenções básicas (FAQ, agendar, remarcar, cancelar, falar com humano).
- RF-04: O chatbot deve conseguir criar agendamentos no banco de dados.
- RF-05: O chatbot deve oferecer, de forma sutil, o uso do aplicativo para clientes recorrentes.
- RF-06: Deve ser possível configurar mensagens de boas-vindas por barbearia.
- RF-07: Deve ser possível configurar o tom de voz do chatbot (mais formal ou descontraído).

### 6.2 Agendamentos

- RF-08: O sistema deve registrar agendamento com: cliente, serviço, barbeiro (opcional), data/hora, canal.
- RF-09: O sistema deve impedir **conflito de horários** para o mesmo barbeiro.
- RF-10: Deve ser possível remarcar agendamentos existentes.
- RF-11: Deve ser possível cancelar agendamentos.
- RF-12: O sistema deve registrar o canal de origem (whatsapp, instagram, app).

### 6.3 App Cliente

- RF-13: Permitir login/cadastro simples (e-mail + código ou WhatsApp).
- RF-14: Listar serviços da barbearia.
- RF-15: Exibir horários disponíveis (slot de agenda) por dia.
- RF-16: Criar agendamento pelo app, sincronizado com o mesmo backend.
- RF-17: Cancelar agendamento pelo app.
- RF-18: Exibir histórico básico de agendamentos passados e futuros.

### 6.4 Painel Barbearia

- RF-19: Login do dono/barbeiro.
- RF-20: Visualizar agenda por dia/semana.
- RF-21: Filtrar agenda por barbeiro.
- RF-22: Cadastrar/editar/remover serviços.
- RF-23: Cadastrar/editar barbeiros.
- RF-24: Configurar texto base do chatbot (boas-vindas, mensagem de ausência, horário de funcionamento).

---

## 7. Requisitos Não Funcionais

- RNF-01: Sistema deve suportar múltiplas barbearias (multi-tenant).
- RNF-02: Latência média da resposta do chatbot < 5 segundos.
- RNF-03: Banco de dados com backup diário (via Supabase).
- RNF-04: API e fluxos protegidos por autenticação e RLS.
- RNF-05: Logs de falhas e mensagens críticas em canal próprio (ex.: Slack/Discord).
- RNF-06: Infraestrutura com custos baixos para escalar (Supabase + n8n self-hosted ou cloud).

---

## 8. Modelagem de Dados – Supabase

> Observação: nomes em inglês para facilitar uso com libs, mas textos exibidos podem ser em PT-BR.

### 8.1 Tabela `barbershops`

- `id` (uuid, PK)
- `name` (text)
- `trade_name` (text, opcional)
- `phone_whatsapp` (text)
- `instagram_handle` (text)
- `address` (text)
- `open_time` (time) – hora abertura
- `close_time` (time) – hora fechamento
- `timezone` (text, default: "America/Sao_Paulo")
- `created_at` (timestamptz)
- `owner_user_id` (uuid, FK → auth.users)

### 8.2 Tabela `barbers`

- `id` (uuid, PK)
- `barbershop_id` (uuid, FK → barbershops)
- `name` (text)
- `bio` (text, opcional)
- `is_active` (boolean)
- `created_at` (timestamptz)

### 8.3 Tabela `services`

- `id` (uuid, PK)
- `barbershop_id` (uuid, FK → barbershops)
- `name` (text)
- `description` (text)
- `price_cents` (integer)
- `duration_minutes` (integer)
- `is_active` (boolean)
- `created_at` (timestamptz)

### 8.4 Tabela `customers`

- `id` (uuid, PK)
- `barbershop_id` (uuid, FK → barbershops)
- `name` (text)
- `phone` (text)
- `instagram` (text)
- `preferred_channel` (text, enum: whatsapp/instagram/app)
- `created_at` (timestamptz)

### 8.5 Tabela `appointments`

- `id` (uuid, PK)
- `barbershop_id` (uuid, FK → barbershops)
- `customer_id` (uuid, FK → customers)
- `service_id` (uuid, FK → services)
- `barber_id` (uuid, FK → barbers, opcional)
- `scheduled_at` (timestamptz) – data/hora do serviço
- `channel` (text, enum: whatsapp/instagram/app)
- `status` (text, enum: pending/confirmed/cancelled/completed)
- `notes` (text, opcional)
- `created_at` (timestamptz)
- `cancelled_at` (timestamptz, opcional)

### 8.6 Tabela `bot_configs`

Configuração específica do chatbot por barbearia.

- `id` (uuid, PK)
- `barbershop_id` (uuid, FK → barbershops)
- `welcome_message` (text)
- `tone` (text, enum: casual/formal/divertido)
- `language` (text, default: "pt-BR")
- `assistant_instructions` (text) – prompt base da IA
- `created_at` (timestamptz)
- `updated_at` (timestamptz)

### 8.7 RLS – Ideia Geral

- Todas as tabelas relacionadas a barbearias devem ter política RLS:
  - o usuário só vê registros da `barbershop_id` que ele administra ou pertence.
- App cliente acessa somente dados públicos da barbearia (serviços, horários, etc.) + seus próprios agendamentos.

---

## 9. Fluxos de Integração (n8n)

### 9.1 Fluxo – Mensagem WhatsApp → IA → Resposta

1. **Trigger:** Webhook (mensagem recebida do provedor de WhatsApp).
2. **Node:** Parse JSON.
3. **Node:** Identificar barbearia pelo número de destino (número oficial da barbearia).
4. **Node:** Buscar `bot_configs` + dados básicos da barbearia no Supabase.
5. **Node:** Construir payload de IA:
   - instruções do assistente (assistant_instructions)
   - contexto da barbearia (horário, serviços, etc.)
   - histórico breve da conversa (últimas 5–10 mensagens)
6. **Node:** Chamada OpenAI Chat/Assistants.
7. **Node:** Tratar resposta:
   - texto para exibir
   - (opcional) JSON com intenção detectada.
8. **Node (IF):** Se intenção = agendar/remarcar/cancelar → chamar subfluxo de agendamento.
9. **Node:** Enviar mensagem de volta ao WhatsApp.

### 9.2 Fluxo – Mensagem Instagram DM → IA → Resposta

Mesmo fluxo do WhatsApp, mudando apenas o **trigger** e o **canal de resposta**.

### 9.3 Subfluxo – Criação de Agendamento

Entradas esperadas:
- `barbershop_id`
- `customer` (nome, telefone, instagram)
- `service`
- `date`, `time`
- `channel`

Passos:
1. Normalizar data/hora para timezone da barbearia.
2. Verificar disponibilidade:
   - se barber escolhido → checar conflitos desse barbeiro
   - se barber não escolhido → tentar alocar automaticamente um disponível.
3. Criar/atualizar `customer` no Supabase.
4. Criar `appointment` com status `confirmed`.
5. Retornar resumo do agendamento para o chatbot responder ao cliente.

---

## 10. Lógica de IA e Prompts

### 10.1 Prompt Base (Template)

```text
Você é o assistente virtual da barbearia {{NOME_BARBEARIA}}.

Sua função é:
- responder dúvidas com clareza e simpatia
- oferecer agendamentos sempre que fizer sentido
- nunca inventar informações (se não souber, diga que não sabe)
- manter respostas curtas, diretas e amigáveis

Informações da barbearia:
- Nome: {{NOME_BARBEARIA}}
- Endereço: {{ENDERECO}}
- Horário de funcionamento: {{HORARIO_FUNCIONAMENTO}}
- Telefone/WhatsApp: {{WHATSAPP}}

Regras de atendimento:
- Sempre que o cliente pedir preço, responda com o valor do serviço e ofereça agendamento.
- Para agendar, peça: nome, serviço desejado, dia e horário aproximado.
- Se o cliente não souber o horário, sugira alguns slots disponíveis.
- Se estiver fora do horário de funcionamento, responda normalmente, mas deixe claro que o agendamento será confirmado no próximo horário útil.

Formato de saída:
1. Uma mensagem amigável em português do Brasil para o cliente.
2. Se identificar uma intenção estruturada (agendar, remarcar, cancelar), retorne também um JSON no final da resposta no seguinte formato:

```json
{"intent":"agendar","service":"Degradê","date":"2025-12-12","time":"15:00"}
```

Se não tiver nenhuma intenção clara, não retorne JSON.
```

> TODO (DEV): transformar o JSON de intenção em algo robusto (ex: usar função de tool-calling da OpenAI para extrair esses dados).

### 10.2 Personalização por Barbearia

- O campo `assistant_instructions` em `bot_configs` permite:
  - mudar o tom de voz
  - incluir regras específicas (ex: taxa de no-show, tolerância de atraso)
  - mudar frases padrão

---

## 11. Aplicativo de Agendamento (Cliente Final)

### 11.1 Stack Sugerida

- **Opção A (mobile):** React Native + Expo.
- **Opção B (web):** Next.js responsivo (PWA), com ícone instalável.

### 11.2 Telas Principais

1. **Tela de Boas-Vindas**
   - Logo da barbearia.
   - Call to action: “Entrar com WhatsApp” ou “Entrar com e-mail”.

2. **Tela de Serviços**
   - Lista de serviços com preço e tempo.
   - Botão “Agendar”.

3. **Seleção de Data e Horário**
   - Calendário simples + lista de horários disponíveis.

4. **Confirmação de Agendamento**
   - Resumo: serviço, dia, hora, barbeiro (se definido).
   - Botão “Confirmar”.

5. **Meus Agendamentos**
   - Próximos agendamentos.
   - Botão “Cancelar” (regras: x horas de antecedência).

6. **Perfil**
   - Nome do cliente.
   - Canal preferido de contato.

---

## 12. Painel Web da Barbearia

### 12.1 Stack Sugerida

- Next.js + Tailwind + Supabase Client.

### 12.2 Módulos

1. **Dashboard / Agenda**
   - Agenda do dia, com cards de horários.
   - Filtro por barbeiro.

2. **Serviços**
   - CRUD de serviços.

3. **Barbeiros**
   - CRUD de barbeiros.

4. **Configurações do Chatbot**
   - Texto de boas-vindas.
   - Horário de funcionamento.
   - Instruções adicionais para IA.

---

## 13. Gestão de Multi-Barbearias (SaaS)

- Cada barbearia tem seu registro em `barbershops`.
- Cada número de WhatsApp/Instagram é mapeado para uma barbearia específica.
- O painel de **Admin da Plataforma** pode:
  - Criar nova barbearia.
  - Configurar credenciais de API de WhatsApp/Instagram.
  - Definir owner (usuário Supabase).

Modelo de cobrança:
- `plan_type` em `barbershops`: basic, pro, premium.
- Limitação de features por plano (ex: número de barbeiros, número de mensagens, etc.).

---

## 14. Segurança, Logs e Monitoramento

- Uso de HTTPS em todas as comunicações externas.
- Chaves de API do OpenAI, WhatsApp e Instagram armazenadas em variáveis de ambiente seguras.
- Logs no n8n para:
  - mensagens recebidas
  - erros de integração
  - tempo de resposta da IA
- Possível integração futura com:
  - Sentry (erros)
  - Logtail / Supabase logs (auditoria).

---

## 15. Roadmap de Evolução

### Fase 1 – MVP
- Fluxo WhatsApp completo (pergunta → IA → agendamento).
- Supabase com tabelas principais.
- Painel web simples de agenda.
- App web responsivo para cliente (PWA).

### Fase 2 – Instagram + Melhorias
- Integração com Instagram DM.
- Melhorias na UI do painel.
- Histórico avançado de clientes.

### Fase 3 – Pagamentos e Fidelidade
- Integração com PIX / link de pagamento.
- Programa de pontos.
- Lembretes automáticos de pagamento.

### Fase 4 – Analytics e Escala
- Dashboards gerenciais (no-show, faturamento estimado, etc.).
- Otimizações de custo e performance.
- Internacionalização.

---

**Fim do SPEC – StarCut v1.0**  
Este documento pode ser usado diretamente como base para desenvolvimento (Codex, outras IAs, ou equipe de dev humana).
