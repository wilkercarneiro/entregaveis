# Prompt para Lovable — Sistema de Coleta de Títulos a Pagar (v2)

## Contexto e Objetivo
Sistema web com dois perfis de usuário, ambos autenticados:
1. **Cliente** — cria conta/faz login, envia títulos a pagar pelo painel dele e visualiza tudo que já enviou, com filtros.
2. **Admin** — visualiza envios de todos os clientes e gera arquivo **.txt** em layout bancário para download.

## Stack
- React + TypeScript + Tailwind CSS
- Supabase (Postgres, Auth, RLS)
- Supabase Auth com e-mail/senha para TODOS os usuários (cliente e admin)

---

## ⚠️ CORREÇÕES OBRIGATÓRIAS (erros da versão anterior — não repetir)

Na versão anterior o sistema foi entregue com estes erros em produção:

1. **`POST /rest/v1/rpc/has_role → 404`**: o frontend chamava uma função RPC `has_role` que NUNCA foi criada no banco. Regra: **toda função RPC referenciada no frontend DEVE ser criada via migration SQL antes**. Crie a função `has_role` no schema public (definição completa abaixo) e garanta que a migration seja executada.

2. **`POST /rest/v1/envios → 401 + erro 42501 "new row violates row-level security policy"`**: as tabelas tinham RLS habilitado mas sem policy de INSERT válida para o fluxo real. Regra: **para cada tabela com RLS, escreva explicitamente as policies de SELECT, INSERT, UPDATE e DELETE** cobrindo cada perfil (cliente e admin), e valide o fluxo completo de ponta a ponta.

3. **Login não funcionava**. Garanta que: o signup crie automaticamente o registro em `profiles` via trigger; o e-mail de confirmação esteja DESABILITADO no Supabase Auth (auto-confirm ativado) para o login funcionar imediatamente em ambiente de teste; e o fluxo login → redirect por role funcione.

Antes de considerar entregue, TESTE: criar conta de cliente → logar → enviar títulos → deslogar → logar como admin → ver o envio → baixar TXT. Nenhum erro no console.

---

## Roles e Autenticação

**Tabela `profiles`**
- id (uuid, pk, referencia auth.users.id)
- nome (text)
- telefone (text)
- cnpj_empresa (text)
- role (text, default 'cliente') — valores: 'cliente' | 'admin'
- created_at (timestamptz, default now())

**Trigger obrigatória**: ao criar usuário em `auth.users`, inserir automaticamente linha em `profiles` (função `handle_new_user` com `security definer`, lendo nome/telefone/cnpj do `raw_user_meta_data`).

**Função `has_role` (criar via migration, obrigatório):**
```sql
create or replace function public.has_role(_user_id uuid, _role text)
returns boolean
language sql
stable
security definer
set search_path = public
as $$
  select exists (
    select 1 from public.profiles
    where id = _user_id and role = _role
  );
$$;
```

**Usuários de teste (criar via seed/migration ou instrução de setup):**
- Admin: `admin@teste.com` / senha `Admin@123` → role 'admin'
- Cliente: `cliente@teste.com` / senha `Cliente@123` → role 'cliente', nome "Cliente Teste", CNPJ "[CNPJ do Cliente (com pontos)]"

Se não for possível criar usuários via migration, crie uma Edge Function de setup ou documente o SQL exato para eu rodar no SQL Editor do Supabase (incluindo o update da role para admin).

---

## Estrutura de rotas
- `/login` → login (único para cliente e admin; redireciona por role após logar)
- `/cadastro` → criação de conta do cliente (nome, telefone com máscara, CNPJ com validação, e-mail, senha)
- `/cliente` → painel do cliente (dashboard com seus envios)
- `/cliente/enviar` → formulário de envio de títulos
- `/cliente/envios/:id` → detalhe de um envio do próprio cliente
- `/admin` → dashboard admin (todos os envios)
- `/admin/envios/:id` → detalhe de envio com edição e download do TXT
- `/admin/configuracoes` → parâmetros do TXT (cod_agencia, cod_conta)

Proteção de rotas: `/cliente/*` exige usuário logado com role 'cliente' (ou admin); `/admin/*` exige role 'admin'. Usuário logado acessando `/login` é redirecionado para seu painel.

---

## Modelo de dados

**Tabela `envios`**
- id (uuid, pk)
- user_id (uuid, fk → auth.users.id) — dono do envio
- created_at (timestamptz, default now())
- status (text, default 'pendente') — pendente / processado

(nome, telefone e CNPJ agora vêm do `profiles` do usuário, não precisam ser repetidos por envio)

**Tabela `titulos`**
- id (uuid, pk)
- envio_id (uuid, fk → envios.id, on delete cascade)
- banco (text)
- fornecedor (text)
- numero_titulo (text, opcional)
- valor (numeric)
- data_vencimento (date)
- data_pagamento (date)
- obs (text, opcional)
- dpcta (text, opcional)
- created_at (timestamptz, default now())

**Tabela `configuracoes`**
- id (uuid, pk)
- cod_agencia (text, default '[Preencha com o código da agência padrão (ex: 3790)]')
- cod_conta (text, default '[Preencha com o número da conta padrão (ex: 5428)]')
- Inserir uma linha default via migration.

---

## Policies RLS (escrever TODAS explicitamente via migration)

**profiles:**
- SELECT: usuário vê o próprio profile; admin (`has_role(auth.uid(),'admin')`) vê todos
- UPDATE: usuário atualiza o próprio profile (exceto o campo role); admin atualiza qualquer um
- INSERT: feito pela trigger `security definer` (não precisa policy de insert para o usuário)

**envios:**
- INSERT: usuário autenticado insere com `user_id = auth.uid()` (with check)
- SELECT: dono do envio (`user_id = auth.uid()`) OU admin
- UPDATE: admin (para marcar como processado); dono NÃO edita após enviar
- DELETE: apenas admin

**titulos:**
- INSERT: usuário autenticado, com check de que o `envio_id` pertence a um envio com `user_id = auth.uid()`
- SELECT: dono do envio pai OU admin
- UPDATE/DELETE: apenas admin

**configuracoes:**
- SELECT/UPDATE: apenas admin

---

## Painel do Cliente (`/cliente`)

**Dashboard:**
- Cards de resumo: total de envios, total de títulos, valor total enviado no mês atual
- Lista dos envios do próprio cliente: data, quantidade de títulos, valor total, status (pendente/processado)
- **Filtros**: por mês/ano (select), por banco, por fornecedor (busca texto), por status, por intervalo de datas de vencimento/pagamento
- Também uma visão "Todos os títulos" (tabela achatada de todos os títulos de todos os envios do cliente) com os mesmos filtros, para ele localizar rapidamente um título específico
- Clicar em um envio abre `/cliente/envios/:id` com a tabela completa (somente leitura)

**Formulário de envio (`/cliente/enviar`):**
Tabela editável dinâmica (botão "+ Adicionar título", opção de remover linha), colunas:
- Banco (select: Banco do Brasil, Bradesco, Caixa, Sicoob, Cresol, Caixinha + "Outro" com campo livre)
- Fornecedor (texto, obrigatório)
- Nº Título (texto, opcional)
- Valor (máscara de moeda R$, obrigatório)
- Data de Vencimento (date picker dd/mm/aaaa, obrigatório)
- Data de Pagamento (date picker dd/mm/aaaa, obrigatório)
- Obs (texto, opcional)
- DPCTA (texto curto, opcional)

Botão "Enviar" → cria 1 registro em `envios` (user_id = usuário logado) + N em `titulos` → tela de confirmação com resumo do que foi enviado.

Design mobile-first, validação inline, feedback de loading.

---

## Painel Admin (`/admin`)

- Dashboard: lista de envios de TODOS os clientes (nome do cliente, CNPJ, telefone, data, qtd. títulos, valor total, status)
- **Filtros**: por cliente (nome/CNPJ), mês/ano, banco, status, período
- Detalhe do envio (`/admin/envios/:id`): tabela completa de títulos com **edição inline** (corrigir erros de digitação do cliente antes de exportar)
- Botão **"Baixar TXT"** no detalhe do envio
- Seleção múltipla no dashboard + **"Baixar TXT consolidado"** (vários envios do MESMO cliente em um arquivo; se envios de clientes/CNPJs diferentes forem selecionados, gerar um arquivo por CNPJ ou avisar)
- Ao baixar, marcar envio(s) como "processado"
- `/admin/configuracoes`: editar cod_agencia e cod_conta

---

## Geração do arquivo TXT

Layout posicional separado por `|`:

**Cabeçalho (uma vez, topo do arquivo):**
```
|0000|{CNPJ_SOMENTE_NUMEROS}|
```

**Para cada título (na ordem de cadastro), dois blocos:**
```
|6000|X||||
|6100|{DATA_PAGAMENTO}|{COD_AGENCIA}|{COD_CONTA}|{VALOR}||{DESCRICAO}||||
```

Onde:
- `{CNPJ_SOMENTE_NUMEROS}` = CNPJ do profile do cliente, sem pontuação
- `{DATA_PAGAMENTO}` = formato dd/mm/aaaa
- `{COD_AGENCIA}` / `{COD_CONTA}` = da tabela `configuracoes` (não vêm do cliente)
- `{VALOR}` = vírgula decimal, sem "R$" (ex.: 810,61)
- `{DESCRICAO}` = campo Obs; se vazio, montar "{FORNECEDOR} {NUMERO_TITULO}", em MAIÚSCULAS e sem acentos/caracteres especiais

Manter exatamente a quantidade de barras `|` de cada linha, mesmo com campos vazios entre elas.

Arquivo gerado client-side, nome `remessa_{CNPJ}_{DATA}.txt`, UTF-8.

**Exemplo de saída esperada:**
```
|0000|[CNPJ_DO_CLIENTE_APENAS_NUMEROS]|
|6000|X||||
|6100|01/06/2026|[COD_AGENCIA]|[COD_CONTA]|810,61||CONTABILIDADE||||
|6000|X||||
|6100|01/06/2026|[COD_AGENCIA]|[COD_CONTA]|279,55||[FORNECEDOR_DE_EXEMPLO_LTDA]||||
```

---

## Checklist final antes de entregar (executar e confirmar)
1. Migrations rodadas: tabelas, trigger `handle_new_user`, função `has_role`, todas as policies RLS, linha default em `configuracoes`
2. Auto-confirm de e-mail ativado no Supabase Auth
3. Usuários de teste criados (admin@teste.com e cliente@teste.com) com roles corretas
4. Fluxo cliente: cadastro → login → enviar títulos → ver na lista → filtrar por mês e banco → SEM erros no console
5. Fluxo admin: login → ver envio do cliente → editar um título → baixar TXT → conferir formato → envio marcado como processado
6. Cliente NÃO consegue ver envios de outros clientes; cliente NÃO acessa /admin

## Observações
- Datas e moeda sempre em pt-BR
- Validar CNPJ (dígito verificador) no cadastro
- Loading/feedback visual em todas as ações assíncronas
