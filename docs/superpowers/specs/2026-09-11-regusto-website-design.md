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
- **Movimento:** GSAP + ScrollTrigger, intensidade "subtle-accent" — fades/reveals com stagger sutil, sem WebGL/3D/partículas/shaders/cursor customizado (salvo caso excepcional justificado durante a implementação — não forçar). `prefers-reduced-motion` respeitado via `gsap.matchMedia()`, sem exceção. **Atualização de 2026-09-11 (ver §11):** parallax/scrub deixou de ser exclusivo do hero — a seção "Como Funciona" ganhou um segundo e único ponto de scrub deliberado, como grande momento interativo do site. Todas as demais seções permanecem no regime original (fade-up disparado uma vez, sem scrub).

## 5. Arquitetura de informação

Página única (`/`) com navegação por âncoras — o escopo do negócio não justifica multi-página no MVP (YAGNI):

1. **Hero** — foto de prato/ambiente em destaque (full-bleed/quase full-bleed, real quando houver asset; placeholder estruturado até lá), headline com a tagline real ("Comida caseira de qualidade"), CTA primário de pedido — composição detalhada em §11.1
2. **Sobre** — história/bairro/proposta (texto real pendente do cliente; placeholder estruturado até lá); storytelling de scroll sutil, ver §11.3
3. **Como Funciona** — apresenta os três formatos (Self-service, Marmitex, Grelhados) como experiência editorial de scroll, não como três cards genéricos — este é o grande momento interativo do site; composição detalhada em §11.2
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
- Qualquer efeito visual WebGL/3D/partículas/shader/cursor customizado, salvo se durante a implementação surgir uma aplicação excepcional e claramente melhor para a Regusto — não deve ser forçado nem buscado ativamente
- Sistema de controle de capacidade de mesas ou reserva real (não se aplica ao formato self-service)

## 10. Rastreabilidade de decisões

Este projeto seguiu o processo de brainstorming arquitetural completo: 6 perguntas de esclarecimento em chat (tipo de negócio, identidade visual disponível, objetivo do site, mecanismo de pedido, manutenção de conteúdo, idioma), pesquisa web para confirmar dados reais da empresa, 3 abordagens de arquitetura apresentadas com trade-offs, e aprovação explícita do cliente antes deste documento. Decisões de julgamento profissional tomadas sem nova pergunta ao cliente (autorizado explicitamente por ele) estão documentadas nas seções 2.3 e 3.

## 11. Pesquisa visual no 21st.dev e ajustes de composição — 2026-09-11

### 11.0 Processo e regra de uso do 21st.dev (permanente)

Antes desta rodada de implementação, foi feita uma pesquisa curada no 21st.dev (MCP) por referências visuais para hero, storytelling de scroll, showcases de imagem, estrutura para os formatos de atendimento, cardápio, contato e microinterações. A pesquisa cobriu ~40 componentes candidatos; a maioria foi descartada por ser genuinamente incompatível com a identidade da Regusto (pricing tables de SaaS, formulários de contato com "trust badges", heroes com glassmorphism/vídeo, qualquer coisa em 3D/WebGL). Um reranking semântico pedindo explicitamente "sem cards genéricos" para os três formatos de atendimento só devolveu bento grids e cards de produto — confirmando que o 21st não tem nada pronto para restaurante, e que Cardápio/Contato precisam continuar autorais (ver §11.4).

**Regra permanente:** o 21st.dev é biblioteca de referência, nunca diretor de arte do projeto. Nenhum componente foi ou deve ser instalado/copiado verbatim. Onde uma referência foi aprovada (abaixo), ela serve como inspiração estrutural/de interação — a implementação real é reconstruída inteiramente com os tokens deste documento e do `2026-09-11-regusto-design-dna.json` (cor, tipografia, easing, paleta de movimento), para que o resultado leia como criação própria da Regusto, nunca como "coleção de componentes do 21st".

### 11.1 Hero — fotografia como protagonista

O `Hero.tsx` originalmente desenhado neste projeto (ver plano de implementação) não incluía nenhuma imagem — apenas fundo de cor sólida (`bg-secondary`). Isso contradizia a própria diretriz deste spec (fotografia como protagonista) e foi corrigido a partir da referência "Editorial Image Hero" pesquisada no 21st (estrutura: foto full-bleed + fade + tagline + headline serifada + CTA — **não copiada**, só usada como inspiração de composição).

Decisão final:
- **Camada de foto**: full-bleed/quase full-bleed ocupando a maior parte do hero, real quando o cliente enviar o asset; até lá, `PhotoPlaceholder` no mesmo enquadramento (ponto de troca único, sem retrabalho de layout). Nunca inventar ou usar banco de imagens genérico — pendência de conteúdo já registrada em §6.
- **Camada de scrim**: gradiente sutil (transparente → `secondary`) sobre a foto, garantindo legibilidade do texto claro já definido — esta é a "transição elegante" pedida, resolvida com o parallax de 2 camadas que o Design DNA já previa para o hero (`visual_effects.scroll_effects.parallax`), sem token novo.
- **Rótulo locacional discreto** (novo, pequeno): usa o token tipográfico `caption` já definido no Design DNA (0.8125rem/peso 500), com dado real já existente em `siteContent.address` (ex.: "Jardim América, Bauru — SP"). Sem caixa alta decorativa (evita o tell de "eyebrow label" genérico). Não é uma tagline nova nem informação institucional inventada — é composição de campos já confirmados.
- **Headline**: mantém a tagline real já aprovada ("Comida caseira de qualidade") como elemento tipográfico principal, preservando o mecanismo de *word-reveal* já validado no projeto — mas migrando de divisão manual de string (`split(" ")` + spans manuais) para **GSAP `SplitText`** real, que o próprio Design DNA já apontava como tecnologia-alvo (`text_effects.technology`), com o split manual apenas como fallback teórico, não como implementação principal.
- **Corpo e CTA**: inalterados (`siteContent.description`, `WhatsAppOrderButton`).
- **Ícone-assinatura**: mantido (path-draw já validado, sem mudança).

Nenhuma tagline ou dado institucional novo foi inventado nesta etapa — tudo vem de `siteContent` já existente.

### 11.2 Como Funciona — experiência editorial de scroll (grande momento do site)

A seção "Como Funciona" deixa de ser uma grade de 3 cards genéricos (self-service, marmitex, grelhados) e passa a ser uma **experiência de scroll horizontal editorial**, referência estrutural aprovada: "Horizontal Feature Reveal" do 21st.dev — **não instalada nem copiada**; reconstruída do zero com os tokens da Regusto. Esta seção é declarada, por decisão explícita do cliente, **o grande momento interativo do site** — o único lugar além do hero onde o site usa scroll-scrub/parallax (ver regra de hierarquia de motion em §11.3).

Decisões de adaptação (respondendo ponto a ponto ao que a referência do 21st trazia):
- **Numeração**: a referência original usava "01/02/03" como se fossem etapas de um processo. **Self-service, marmitex e grelhados não são uma sequência** — são três formatos paralelos de atender o cliente. Decisão final: **nenhum numeral**. O nome de cada formato ("Self-service", "Marmitex", "Grelhados" — em type sentence-case, não caixa alta decorativa) é o próprio elemento tipográfico gigante, na mesma escala `display` do hero. Isso evita o tell de numeração-como-processo e reforça a leitura editorial em vez de "onboarding em 3 passos".
- **Tipografia**: `display` (Fraunces) para o nome do formato, `body` (Work Sans) para a descrição — sem introduzir escala nova.
- **Cores**: fundo `secondary` (verde-oliva) / texto `neutral-100`, estendendo a regra de inversão de contraste que o Design DNA já reservava para seções institucionais de destaque (ver DNA JSON, `contrast_strategy`, atualizado nesta data). Cria um ritmo escuro→claro→**escuro (momento alto)**→claro ao longo da página, reforçando esta seção como pico visual e de movimento, não só de movimento.
- **Espaçamento/composição**: cada formato ocupa um painel full-viewport dividido em foto (metade) + texto (metade), seguindo o `focal_strategy` do Design DNA ("uma foto em destaque por vez, sem colagens saturadas") — uma foto real por formato (self-service = balcão/prato montado, marmitex = embalagem pronta, grelhados = prato grelhado), `PhotoPlaceholder` até o cliente enviar os assets.
- **Transições/comportamento (desktop, ≥1024px)**: trilha horizontal fixada (`position: sticky` + `ScrollTrigger` com `scrub`) que avança conforme o scroll vertical — o "grande momento" pedido. Parallax de imagem sutil (10–20%, mesmo `depth_range` já definido no Design DNA para o hero). Distância de scroll dimensionada para 3 painéis (mais curta que a referência original, pensada para 4) — ajustável na implementação conforme sensação real ao testar.
- **Mobile/tablet (<1024px)**: **sem scroll-jack horizontal.** Pilha vertical estática reaproveitando o componente `Reveal` já existente no projeto (fade + translateY sutil, mesmo padrão de entrada do resto do site) — nenhuma mecânica nova só para mobile, o que mantém a experiência do público local (majoritariamente mobile, conforme §7) simples e coerente com o resto do site.
- **Acessibilidade**: ordem do DOM é sempre nome → descrição → foto (independente da disposição visual esquerda/direita, resolvida via CSS, nunca por reordenação física); nenhum listener customizado de wheel/touch é usado (o scroll nativo continua funcionando para teclado/scroll normal); a seção não deve interceptar ou bloquear a rolagem por teclado.
- **Reduced motion**: correção deliberada de uma falha observada na referência do 21st (lá, o `prefers-reduced-motion` desligava só os reveals de texto, mas mantinha o scroll-jack horizontal ativo). Na Regusto, sob `prefers-reduced-motion: reduce`, a seção renderiza **o mesmo layout estático empilhado do mobile**, mesmo em desktop — nunca uma versão "parada" do layout horizontal, que ficaria confusa sem o scroll para guiá-la.

### 11.3 Regra permanente: hierarquia de motion do site

A partir desta data, o site segue uma hierarquia de intensidade de movimento explícita — nenhuma seção fora do Hero e do Como Funciona deve introduzir scrub, pin ou parallax novo sem atualizar esta regra:

| Seção | Papel do motion |
|---|---|
| Hero | Impacto — parallax de 2 camadas + reveal de texto ao carregar |
| Como Funciona | Grande momento — único outro ponto de scrub/pin do site |
| Sobre / Ambiente | Storytelling sutil — fade-up simples, sem scrub (referência de scroll sticky do 21st, "Scroll 01", pode inspirar um crossfade de imagem sutil aqui, mas sem competir com Como Funciona) |
| Cardápio | Clareza e conversão — sem efeito de destaque, foco em leitura e ação |
| Contato | Ação — foco no CTA de WhatsApp, sem decoração de movimento |

Não transformar todas as seções em experiências altamente animadas é uma decisão de identidade, não uma limitação técnica.

### 11.4 Cardápio e Contato/Localização — permanecem autorais

Confirmado: o 21st.dev não tem solução adequada para nenhum dos dois (só pricing tables de SaaS, cards de produto de app de delivery e formulários de contato com captura de lead). Nenhum componente externo deve ser forçado nessas seções. Prioridade do Cardápio, em ordem: **experiência visual + legibilidade + descoberta + facilidade de pedido** — a estrutura já definida em §5.4 (categorias reais, nota de prato do dia, preços tabulares) e §6 permanece válida; refinamentos de composição ficam a critério da implementação, sempre autorais.

### 11.5 Dependências técnicas

Nenhuma biblioteca nova além do que já estava decidido (`gsap`, `@gsap/react`). **GSAP `SplitText`** passa a ser usado deliberadamente (não é dependência nova — plugin do próprio pacote `gsap`, gratuito para todos desde a mudança de licenciamento GreenSock/Webflow) para o reveal do headline do hero e dos nomes de formato em Como Funciona, substituindo a divisão manual de string por spans. **Framer Motion não entra no projeto** — onde uma referência do 21st usava Framer Motion (ex.: "Scroll 01"), a ideia é reconstruída em GSAP + ScrollTrigger, que já é a stack de movimento do projeto.

### 11.6 WebGL/3D/partículas/shaders — reafirmação

Continuam fora de escopo (§9), reafirmado após a pesquisa no 21st ter encontrado várias referências tecnicamente interessantes nessa linha (galeria 3D, coverflow, hero com Three.js) — todas descartadas por incompatibilidade com a identidade caseira e o público majoritariamente mobile. Só devem ser reconsideradas se, durante a implementação, surgir uma aplicação excepcional e claramente melhor para a Regusto — nunca por padrão, nunca forçado.
