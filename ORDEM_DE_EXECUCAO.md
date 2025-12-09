# Ordem precisa de execução

Guia sequencial para implementar o projeto, indicando o que o Codex executa automaticamente e o que você precisa fazer manualmente em cada fase.

## 1) Fase 0 — Preparação e fundação
1. **Criar projeto no Supabase (você)** e guardar `SUPABASE_URL`, `anon`, `service_role`.
2. **Configurar workspace n8n (você)** e testar URL pública de webhook.
3. **Gerar estrutura de repositório (Codex)**: pastas `/apps/web-admin`, `/apps/app-client`, `/services/n8n`, `/infra/supabase`, `.env.example`, `README_SETUP.md`.
4. **Aplicar SQL inicial (você)** no Supabase: tabelas, PKs/FKs/indexes, RLS/policies multi-tenant, seeds.
5. **Criar função `getBarbershopByChannel` (Codex)** e validar retorno no Supabase.
6. **Configurar credenciais no n8n (você)**: Supabase, OpenAI, WhatsApp, Instagram; criar webhook de teste apontando para n8n.

## 2) Fase 1 — MVP (WhatsApp + App Cliente + Painel)
7. **Fluxo WhatsApp→IA→Agendamento (Codex)**: workflow `wa_inbound` (normalização, prompt, validação, disponibilidade, agendamento, resposta, logs).
8. **Conectar provedor de WhatsApp ao webhook do n8n (você)** e validar mensagens reais.
9. **Backend Supabase (Codex)**: funções `checkAvailability`, `listSlots`, `bookAppointment` e tabelas/queries de apoio.
10. **Painel da barbearia (Codex)**: dashboard de hoje, agenda semanal, CRUD serviços/barbeiros, config do chatbot, auth Supabase.
11. **App cliente PWA (Codex)**: rotas `/auth`, `/services`, `/slots`, `/appointments` integradas ao Supabase.
12. **Configurar variáveis de ambiente e deploy web/app (você)**: Vercel/Netlify, chaves Supabase e tokens externos.
13. **QA ponta a ponta (você)**: login, agendar via WhatsApp e app, verificar painel refletindo agenda.

## 3) Fase 2 — Instagram
14. **Fluxo Instagram→IA→Agendamento (Codex)**: workflow similar ao WhatsApp, suporte a mídia.
15. **Configurar app Meta e webhooks (você)**: tokens, autorização de páginas, apontar webhooks para n8n, testar DMs reais.

## 4) Fase 3 — Pagamentos e fidelidade
16. **API PIX e lógica de pontos (Codex)**: endpoint de QR Code, vínculo com agendamento, tabela `loyalty_points`, regra de pontos e páginas “Meus Pontos”.
17. **Credenciais de pagamento (você)**: criar conta MercadoPago/Gerencianet, inserir `client_id/client_secret` no `.env`, configurar callbacks de confirmação.
18. **Testes de pagamento (você)**: gerar QR, pagar, verificar confirmação e atualização de pontos/agendamentos.

## 5) Fase 4 — Analytics e escala
19. **Métricas e dashboards (Codex)**: tabela `analytics_events`, coleta de métricas (no-show, fluxo, origem), painéis para barbearias e admin SaaS, caching de slots, suporte multi-idioma.
20. **Validação e branding (você)**: ajustar charts/cores, conferir dados reais, preparar para produção.

## 6) Finalização e operação contínua
21. **Auditoria de segurança e RLS (você + Codex)**: revisar políticas, testar perfis de acesso.
22. **Monitoramento e logs (Codex)**: garantir logs do chatbot e app; configurar alertas no n8n/Supabase (você).
23. **Checklist de produção (você)**: domínios configurados, backups habilitados, testes finais de WhatsApp/Instagram/pagamentos.
24. **Ciclo de evolução (você + Codex)**: priorizar melhorias, abrir issues, planejar sprints.
