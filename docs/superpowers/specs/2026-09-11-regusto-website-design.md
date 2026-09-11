# Regusto — Website Institucional — Design

**Data:** 2026-09-11
**Status:** Aprovado pelo cliente em chat (2026-09-11), pendente de plano de implementação.

## 1. Contexto e objetivo

A Regusto é um restaurante self-service por quilo, com marmitex e grelhados, no bairro Jardim América, Bauru/SP. O projeto atual (`D:\regusto`) é um repositório Next.js vazio (scaffold padrão do `create-next-app` foi removido da working tree; histórico permanece no git) que servirá de base para um site institucional novo, autoral, construído do zero.

**Objetivo do site** (confirmado pelo cliente): site completo — institucional + cardápio + caminho de pedido — não uma landing page única de marca, nem só um cardápio utilitário.

**Filosofia de execução** (diretriz do cliente, resumida): buscar o nível de "estúdio profissional de alto nível" através de melhor direção, composição, tipografia, movimento e UX — não através de acúmulo de efeitos, bibliotecas ou complexidade. Coerência com a marca real é um critério de qualidade explícito, não opcional.

## 2. Pesquisa e fatos reais

Fonte oficial usada como identificador principal, conforme instrução do cliente: **https://regusto.com.br**.

### 2.1 Dados oficiais (site oficial + página de contato)
- **Nome no site oficial:** "Regusto Restaurante - Marmitas, Marmitex, Grelhados e Self Service por kg"
- **Endereço:** Av. Getúlio Vargas, 16-70, Jardim América, Bauru - SP, CEP 17017-339
- **Telefone institucional:** (14) 3204-4151
- **Horário oficial:** Segunda a sábado, 08:00 às 14:20. Domingo fechado.
- **CNPJ:** 27.001.306/0001-19
- **Formas de pagamento:** Dinheiro, PIX, cartões (Visa, Mastercard, Hipercard, Elo), vale-refeição (Alelo, Verocad, Sodexo, Ticket Restaurante)
- **Categorias de cardápio confirmadas:** Grelhados à la Carte, Marmitex (com recorte "Sexta-Feira"), Pastéis (três subcategorias), Salada, Bebidas
- **Plataforma de pedido atual:** Pedizap (pedido via WhatsApp), número WhatsApp em uso: `5511998731881`

### 2.2 Dados públicos de terceiros (não oficiais — usados só como contexto, não como fonte primária)
- Nome mais completo usado em listagens externas (Tripadvisor, Facebook, Solutudo, Social Bauru, Guia Bauru Pocket): **"Regusto — Restaurante e Rotisserie"**
- Instagram oficial: **@regustorestaurante** — bio real: *"Restaurante . Comida caseira de qualidade"* (1.326 seguidores). Conteúdo predominante: pratos do dia rotativos (filé mignon, frango, carne moída), cardápios de marmitex.
- Capacidade estimada ~100 pessoas, estacionamento e delivery disponíveis, nota 4.2 no Google (fonte: agregadores, não verificado diretamente no Google Meu Negócio)
- Contato Facebook associado: perfil pessoal "Paulo Celso Alarcon" — não usado como conteúdo do site sem confirmação do cliente sobre o papel dessa pessoa (dono/gestor)
- Artigo "Inauguração do Regusto Restaurante & Rotisserie" (Social Bauru, 18/03/2017) confirma que o negócio existe nesse endereço desde ao menos 2017; não trouxe texto de história/conceito aproveitável
- Listagem do iFood existe mas bloqueou acesso automatizado (HTTP 403) — itens/preços específicos não puderam ser extraídos por pesquisa

### 2.3 Decisões tomadas a partir da pesquisa
- **Horário exibido no site = o oficial** (08:00–14:20, seg-sáb) — ver regra completa e status "a confirmar" em §2.4.
- **Marca no site = "Regusto"** (curto, como no domínio e no handle do Instagram), com "Comida caseira de qualidade" (frase real já usada pela própria marca no Instagram) como tagline — em vez de inventar uma nova frase ou adotar o nome mais longo de listagens de terceiros.
- **CTA de pedido usa o número de WhatsApp já em produção** (`5511998731881`), não o telefone fixo institucional — telefone fixo não opera WhatsApp; o número móvel é o que a Regusto já usa hoje para receber pedidos reais via Pedizap.
- **"Reserva" (do briefing original) é reinterpretada como "fazer pedido/encomendar"** (marmitex/grelhados para retirada ou entrega via WhatsApp), não como reserva de mesa — o formato self-service por quilo não opera com reserva de mesa tradicional.

### 2.4 Regra permanente: divergência de horário de funcionamento

Existe uma divergência real entre o horário do site oficial (08:00–14:20, seg-sáb) e o horário mostrado por agregadores externos (ex: 11:00–15:00). Essa divergência **não é resolvida por esta pesquisa** e não deve ser resolvida por invenção. Regra explícita do cliente para o projeto:

1. O horário do **site oficial é a informação principal** usada em todo o site, enquanto a divergência não for esclarecida.
2. Essa informação é marcada como **"a confirmar"** — deve ser revisada com o cliente antes do lançamento/publicação final, não é definitiva.
3. Dados de agregadores **nunca substituem automaticamente** o horário oficial, nem agora nem em atualização futura de conteúdo.
4. O horário vive em **um único local editável** da aplicação (ver §6) — nenhum componente deve ter o horário hardcoded separadamente, exatamente para que essa confirmação futura (ou qualquer correção) seja uma edição em um lugar só.

## 3. Decisão de arquitetura técnica

Três abordagens foram apresentadas ao cliente; **Approach A foi aprovada**:

**Conteúdo em código + pedido via WhatsApp.** Next.js (App Router) + TypeScript + Tailwind CSS + GSAP/ScrollTrigger, deploy na Vercel. Cardápio/textos/dados práticos vivem em arquivos de conteúdo tipados no próprio repositório (não em CMS headless). O CTA de pedido é um link `wa.me` com mensagem pré-preenchida, isolado num componente próprio.

Alternativas descartadas e por quê:
- **CMS headless (Sanity, etc.):** adiciona complexidade real (projeto externo, tokens, modelagem de conteúdo) não justificada — o cliente não indicou necessidade de autoatendimento de conteúdo por equipe não-técnica com urgência suficiente para pagar esse custo.
- **Formulário próprio de reserva com API route + e-mail:** seria "teatro de reserva" para um negócio self-service (não controla mesas nem capacidade real), e não reflete como a Regusto já opera pedidos hoje (WhatsApp/Pedizap).

## 4. Identidade visual (Design DNA)

Perfil completo nos três eixos (design_system, design_style, visual_effects) está em [`2026-09-11-regusto-design-dna.json`](./2026-09-11-regusto-design-dna.json), gerado por julgamento profissional a partir da pesquisa (não há fotos de referência medidas nesta fase — nenhuma imagem foi fornecida). Resumo:

- **Paleta:** terracota queimada `#A8432B` (marca/CTA), verde-oliva escuro `#2F3B2E` (contraste institucional), dourado-mostarda `#E8A33D` (destaque pontual), escala neutra de creme `#FBF6EE` a quase-preto quente `#1E1A16`.
- **Tipografia:** Fraunces (serifada editorial, variable) para display/headings/nomes de prato; Work Sans para corpo/UI/preços (tabular-nums).
- **Tom:** acolhedor, caseiro-com-orgulho, apetitoso, honesto, contemporâneo — editorial gastronômico, não fine-dining, não fast-food.
- **Fotografia:** protagonista visual; luz lateral quente, sem filtro sintético, foco em prato/detalhe/textura/ambiente. Fachada real serve só como referência de atmosfera/materiais, não precisa aparecer na composição final.
- **Movimento:** GSAP + ScrollTrigger, intensidade "subtle-accent" — fades/reveals com stagger sutil, parallax leve só no hero, sem WebGL/3D/partículas/shaders/cursor customizado. `prefers-reduced-motion` respeitado via `gsap.matchMedia()`. Essa contenção é intencional: efeitos pesados não servem à identidade caseira e arriscam performance no público local (muitos em conexão móvel).

## 5. Arquitetura de informação

Página única (`/`) com navegação por âncoras — o escopo do negócio não justifica multi-página no MVP (YAGNI):

1. **Hero** — foto de prato/ambiente em destaque, headline com a tagline real ("Comida caseira de qualidade"), CTA primário de pedido
2. **Sobre** — história/bairro/proposta (texto real pendente do cliente; placeholder estruturado até lá)
3. **Como Funciona** — explica o formato self-service por kg + marmitex + grelhados para quem não conhece
4. **Cardápio** — por categoria real (Grelhados à la Carte, Marmitex, Pastéis, Saladas, Bebidas), com nota visível direcionando ao WhatsApp/Instagram para o prato do dia (que muda diariamente e não é praticável manter estático — ver §6)
5. **Ambiente** — galeria de fotos (placeholder até receber fotos reais)
6. **Informações Práticas** — endereço com mapa embutido, horário oficial, formas de pagamento, estacionamento
7. **Contato/Pedido** — CTA de WhatsApp repetido, telefone institucional, endereço

Uma rota dedicada `/cardapio` fica fora do escopo do MVP, mas a estrutura de conteúdo (§6) é feita para permitir extrair essa seção para sua própria página depois, sem retrabalho, caso valha a pena por SEO local.

## 6. Modelo de conteúdo e estratégia de placeholder

Conteúdo vive em arquivos TypeScript tipados dentro do repositório (ex: `content/site.ts`, `content/menu.ts`), não em banco de dados nem CMS. Cada campo de conteúdo real ainda não recebido é marcado explicitamente como placeholder (não silenciosamente inventado como se fosse real), para ser trocado depois sem mudança de schema.

**Decisão importante:** o prato do dia (rotativo, visto no Instagram) **não** é modelado como conteúdo do site — replicar isso exigiria redeploy quase diário, incompatível com o modelo de manutenção "desenvolvedor edita código" escolhido pelo cliente (Approach A). O site exibe as categorias estáveis do cardápio oficial e direciona explicitamente para WhatsApp/Instagram para o prato do dia. Isso é uma decisão de honestidade sobre a natureza do negócio, não uma limitação escondida.

**Horário de funcionamento — fonte única (ver regra em §2.4):** o horário não é repetido como texto solto em cada componente que o exibe (hero, rodapé, seção de informações práticas, dados estruturados de SEO). Vive como um único valor exportado em `content/site.ts`, algo como:

```ts
export const businessHours = {
  display: "Segunda a sábado: 08:00–14:20 · Domingo: fechado",
  source: "site oficial (regusto.com.br)",
  confirmedForLaunch: false, // divergência com agregadores externos (~11:00–15:00) — confirmar com o cliente antes de publicar
};
```

Todo componente que mostra horário importa `businessHours.display` — nunca reescreve o texto. Corrigir o horário (ou marcar `confirmedForLaunch: true` após confirmação do cliente) é uma edição nesse único arquivo, sem tocar em componentes.

### Pendências reais de conteúdo (bloqueiam apenas o polimento final, não a estrutura)
- Itens específicos do cardápio com preços (categorias confirmadas; itens/preços não — listagem do iFood bloqueou acesso automatizado)
- Fotografia real de pratos e ambiente (não extraível de Instagram/site por ferramenta automatizada; precisa vir do cliente)
- Texto de "Sobre" / história (nenhuma fonte pesquisada trouxe texto aproveitável)
- Confirmação se algum papel de pessoa (ex: "Paulo Celso Alarcon") deve aparecer como chef/proprietário na seção Sobre

## 7. Acessibilidade e performance

- Contraste de texto verificado contra os tokens de cor do Design DNA (dark-on-light dominante, quase-preto quente sobre creme)
- Toda animação via `gsap.matchMedia()` respeitando `prefers-reduced-motion`
- Imagens via `next/image` com `alt` descritivo (comida/ambiente), carregamento otimizado para conexões móveis mais lentas (público de bairro)
- Fontes via `next/font` (Fraunces + Work Sans), sem web font externa bloqueante
- HTML semântico, navegação por teclado nas âncoras e no CTA de WhatsApp

## 8. Testes

- Verificação visual manual no navegador (skill `webapp-testing` / `run`) cobrindo: caminho de pedido via WhatsApp, leitura do cardápio, navegação por âncoras, responsividade mobile (prioridade, dado o público local) e desktop, `prefers-reduced-motion` ativado
- Nenhum teste automatizado de unidade é necessário no MVP — é um site de conteúdo/marketing sem lógica de negócio complexa; interações não triviais (CTA de WhatsApp, animações scroll-triggered) são verificadas visualmente

## 9. Fora de escopo (explícito)

- CMS headless / painel de edição para equipe não-técnica
- Formulário próprio de reserva/pedido com backend e e-mail
- Multi-idioma (site é só pt-BR)
- Rota `/cardapio` dedicada (estrutura permite adicionar depois)
- Qualquer efeito visual WebGL/3D/partículas/shader/cursor customizado
- Sistema de controle de capacidade de mesas ou reserva real (não se aplica ao formato self-service)

## 10. Rastreabilidade de decisões

Este projeto seguiu o processo de brainstorming arquitetural completo: 6 perguntas de esclarecimento em chat (tipo de negócio, identidade visual disponível, objetivo do site, mecanismo de pedido, manutenção de conteúdo, idioma), pesquisa web para confirmar dados reais da empresa, 3 abordagens de arquitetura apresentadas com trade-offs, e aprovação explícita do cliente antes deste documento. Decisões de julgamento profissional tomadas sem nova pergunta ao cliente (autorizado explicitamente por ele) estão documentadas nas seções 2.3 e 3.
