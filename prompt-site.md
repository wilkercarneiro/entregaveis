# Prompt para Lovable.dev — Novo site Consultem Contabilidade (v2, com regras anti-bug)

## PROMPT (copiar tudo abaixo desta linha)

Você vai construir o novo site institucional da **Consultem — Gestão Contábil de Empresas e Negócios**, escritório de contabilidade em Conceição do Coité-BA. Já existe um site atual com identidade visual consolidada e arquitetura de informação definida — sua missão é **recriar essa mesma estrutura e identidade, mas com execução visual e técnica de nível muito superior**: animações fluidas, transições modernas, microinterações refinadas. O site atual parece um template de WordPress; o novo deve parecer um produto digital feito sob medida.

Objetivo duplo: autoridade institucional + geração de leads (WhatsApp e formulários).

---

### 0. REGRAS INVIOLÁVEIS — leia antes de qualquer decisão de design

Estas duas categorias de erro ocorreram na geração anterior e são **inaceitáveis**. Trate-as como restrições de nível máximo, acima de qualquer preferência estética:

**A) Contraste de texto — nunca texto branco sobre fundo claro:**
- A cor do texto de CADA seção deve ser definida junto com a cor de fundo daquela seção, nunca herdada globalmente. Regra fixa de pareamento:
  - Fundo navy escuro (`#2E4A62` a `#1F3A50`) → headlines em branco `#FFFFFF`, corpo em `#E8EDF2`, destaques em amarelo `#F7C325`
  - Fundo branco, off-white ou cinza claro (`#EFEFEF`) → headlines em navy `#2E4A62`, corpo em `#3D4B58`, destaques em amarelo escurecido `#C99B12` (o amarelo `#F7C325` puro NÃO tem contraste suficiente para texto sobre fundo claro — use-o em fundo claro apenas em botões/formas, nunca em texto)
  - Fundo amarelo (`#F7C325`, seção Instagram) → todo texto em navy `#2E4A62`
- É PROIBIDO usar `text-white` (ou branco em qualquer forma) em seções com fundo branco/claro. Antes de finalizar, faça uma varredura seção por seção verificando o par fundo × texto de cada uma — inclusive nos estados de hover, nos cards internos e nos textos sobre overlays.
- Texto sobre foto só é permitido com overlay navy em gradiente garantindo contraste AA; nunca texto direto sobre imagem.
- Componentes de UI (shadcn ou similares) que assumem tema escuro/claro global devem ter as cores explicitamente sobrescritas por seção — não confie no default do tema.

**B) Overflow horizontal e responsividade — a página NUNCA pode rolar para o lado:**
- Nenhuma seção pode gerar scroll horizontal na página em NENHUMA largura de tela (teste mental: 320px, 375px, 768px, 1024px, 1440px). Aplique `overflow-x: clip` no wrapper de cada seção que contenha carousel, formas decorativas ou elementos com parallax, e garanta `overflow-x: hidden` no body como rede de segurança final — mas a correção real deve ser feita na causa, não só mascarada no body.
- **Carousel de serviços (seção 3.2) — causa do bug anterior:** o track do carousel deve ficar contido dentro de um viewport próprio com `overflow: hidden`; os cards fora da área visível NÃO podem empurrar a largura da página. Use largura de card fluida (`min()` / porcentagem / clamp), nunca largura fixa em px que some além da viewport. No mobile (<768px), o carousel vira scroll-snap horizontal nativo (`scroll-snap-type: x mandatory`) contido na própria seção, com 1 card + espiar do próximo (`~85vw` por card), sem drag JS complexo.
- Formas decorativas (semicírculos navy, blobs amarelos) que vazam para fora da seção devem estar em containers com `overflow: clip` ou posicionadas com `inset` negativo dentro de wrapper clipado — nunca podem expandir a largura do documento.
- Elementos com parallax/transform não podem criar largura extra: anime apenas `transform` dentro de containers clipados.
- Todas as seções devem ser construídas mobile-first e verificadas nos breakpoints: 320px, 375px, 768px, 1024px e 1440px. Grids de cards: 1 coluna no mobile, 2 no tablet, 3 no desktop. Tipografia com `clamp()` para escalar sem quebrar.
- Ao final da implementação, faça uma passada de QA explícita: (1) verificar par fundo × texto de cada seção; (2) verificar ausência de scroll horizontal em cada breakpoint listado; (3) verificar o carousel de serviços especificamente no mobile.

---

### 1. Identidade visual (MANTER — é a marca existente)

**Paleta:**
- Azul petróleo/navy: `#2E4A62` (variações até `#1F3A50` para fundos escuros) — cor institucional principal
- Amarelo dourado: `#F7C325` — cor de acento, CTAs e destaques em headlines
- Cinza claro: `#EFEFEF` para fundo do header e seções neutras
- Branco e off-white para respiros
- Texto sobre fundo escuro: branco puro nos headlines, `#E8EDF2` no corpo

**Tipografia:**
- Manter a família geométrica arredondada do site atual (Poppins) para continuidade da marca, MAS elevar o tratamento tipográfico: hierarquia mais dramática (headlines maiores, 56–72px desktop), tracking ajustado, pesos contrastantes (Bold nos títulos, Regular/Light no corpo), line-height respirado (1.6–1.7 no corpo)
- Padrão de destaque nos headlines: palavras-chave em amarelo dentro de frase branca/navy (ex: "Uma **gestão contábil** transparente e responsável") — manter esse padrão, é assinatura da marca

**Linguagem de formas (assinatura visual do site atual — manter e refinar):**
- Formas circulares e orgânicas grandes: semicírculos navy nas bordas das seções, blobs amarelos atrás de fotos, molduras em arco para retratos
- Padrão de ondas/pontos sutis em fundos navy (como no hero atual)
- Refinar: essas formas devem ter **parallax sutil no scroll** (velocidades diferentes do conteúdo), criando profundidade

**Logo:** "C" circular em navy com detalhe amarelo + wordmark "CONSULTEM / GESTÃO CONTÁBIL DE EMPRESAS E NEGÓCIOS". Use placeholder fiel a isso; a logo real será enviada depois.

### 2. Estrutura de navegação (MANTER)

Navbar: **Serviços** (dropdown) | **Segmentos** (dropdown) | **Conteúdos** (dropdown) | **Abrir conta PJ** | **Sobre nós** | botão destacado **ÁREA VIP** (navy, canto direito).

Melhorias na navbar:
- Sticky com transição: começa sobre fundo cinza claro; ao rolar, encolhe suavemente e ganha fundo com blur (glassmorphism sutil) e sombra leve
- Dropdowns com animação de entrada (fade + slide 8px, stagger nos itens)
- Indicador de página ativa com underline animado em amarelo

### 3. Estrutura de seções (mesma do site atual, execução elevada)

**3.1 Hero**
- Fundo navy escuro com padrão de ondas/pontos animado sutilmente (movimento lento, quase imperceptível)
- Headline: "Uma **gestão contábil** transparente e responsável" (destaques em amarelo) — entrada com split-text animado por palavra (stagger, easing suave), NUNCA com palavras se sobrepondo
- Subtítulo: "Cada empresa é única, e na Consultem, acreditamos que suas soluções contábeis devem refletir essa singularidade."
- CTA amarelo "CONHEÇA A CONSULTEM" com hover magnético sutil e microanimação de preenchimento
- À direita: retrato do contador em composição orgânica (círculo navy + blob amarelo), com parallax leve e entrada com máscara circular revelando a foto
- Usar placeholder de retrato profissional; fotos reais serão substituídas depois

**3.2 Serviços em destaque**
- Headline: "Transformamos seu negócio com soluções contábeis"
- Subtítulo: "Nosso compromisso é proporcionar maior flexibilidade e eficácia para o crescimento do seu negócio"
- Carousel de cards de serviço (ex: "Equiparação hospitalar — Mantenha seu hospital em conformidade e otimize seus processos") em card navy com CTA amarelo "CONHEÇA MAIS"
- Elevar: carousel com drag/swipe fluido, transição com easing físico, indicadores de progresso animados, cards vizinhos parcialmente visíveis com escala reduzida
- OBRIGATÓRIO: seguir as regras de contenção da seção 0-B — track dentro de viewport com `overflow: hidden`, larguras fluidas, scroll-snap nativo no mobile. Este componente causou overflow horizontal na geração anterior e não pode quebrar de novo
- Foto em moldura de arco ao lado, com parallax

**3.3 Segmentos**
- Headline: "O suporte contábil que seu segmento merece"
- Cards: **Saúde** ("Aumente a eficiência da sua prática médica otimizando sua saúde financeira"), **Comércio** ("Controle preciso das finanças, garantindo uma visão clara da gestão do seu comércio"), **Advogado** ("Revolucione seu escritório de advocacia e aumente a precisão financeira")
- Elevar: em vez de 3 cards brancos genéricos com sombra, use cards com borda fina, hover que eleva sutilmente + linha amarela animada na base + ícone com microanimação. Entrada em viewport com stagger

**3.4 CTA — Abertura de ME**
- Banner full-width com foto da equipe e overlay navy escuro (gradiente, não overlay chapado)
- Headline: "Não espere mais, **abra sua ME** com a **segurança** da Consultem"
- ATENÇÃO: no site atual há um bug em que as palavras animadas do headline se sobrepõem, ficando ilegível. A animação aqui deve ser simples e à prova de falha: fade + translateY por linha, sem trocar palavras no mesmo espaço
- Texto: "Entendemos que abrir uma Microempresa é crucial para seu sucesso. Por isso, oferecemos um serviço ágil e personalizado para atender a todos os requisitos legais e fiscais."
- CTA amarelo "ABRIR MINHA ME AGORA"
- Efeito: leve parallax na foto de fundo (fixa mais lenta que o scroll)

**3.5 Benefícios**
- Headline: "Transforme desafios financeiros em oportunidades"
- 3 pilares: **Gestão a um toque** ("Gerencie suas finanças de forma rápida e fácil com nosso app"), **Eficiência operacional** ("Simplifique a administração financeira com processos contábeis otimizados e automatizados"), **Visão clara e acessível** ("Acesse relatórios financeiros detalhados e atualizados instantaneamente")
- Elevar: ícones em linha (stroke) com animação de desenho (draw-on) ao entrar em viewport, no lugar dos ícones em círculo azul chapado

**3.6 Instagram**
- Fundo amarelo (manter, é assinatura da marca) com formas orgânicas navy
- Headline: "Nos siga no Instagram!"
- Texto: "Acompanhe nosso perfil para obter insights valiosos e ver o que estamos criando diariamente. Nossa página é o lugar ideal para se conectar e aproveitar o melhor que temos a oferecer."
- CTA navy "SEGUIR A CONSULTEM" → https://instagram.com/consultemcontabilidade
- Mosaico de posts (placeholders) com rotações sutis e hover que endireita/eleva o card — sensação de "prints espalhados na mesa", com parallax entre eles

**3.7 Blog / Conteúdos**
- Headline: "Aprofunde seu conhecimento com nossos artigos"
- Grid de cards de artigo (imagem + título + data + "LER AGORA") — hover com zoom sutil na imagem e underline animado no título

**3.8 CTA final**
- Banner com foto da equipe + overlay navy em gradiente
- Headline: "A **Consultem** está pronta para **impulsionar seu sucesso**"
- Texto: "Conte com uma abordagem estratégica e eficiente para todas as suas necessidades contábeis."
- CTA amarelo "INICIAR CONSULTORIA HOJE"

**3.9 Footer**
- Logo + colunas de Contato e Endereço:
  - E-mail: contato@consultemcontabilidade.com.br
  - Telefone: (75) 99153-5103
  - Endereço: Avenida Padre Madureira, nº 29, 1º Andar, Madureira, CEP 48730-000, Conceição do Coité-BA
  - Instagram: @leonardoocontador e @consultemcontabilidade
  - Link: Política de privacidade
- Barra final navy: "© Todos os direitos reservados – Consultem Gestão Contábil"

**Elementos persistentes:**
- Botão flutuante de WhatsApp (verde, canto inferior direito) com animação de pulso discreta e mensagem pré-preenchida contextual
- Widget de chat/convite ("Quer consultar nosso melhor preço para o serviço que está buscando?") aparecendo após alguns segundos com animação suave, com botão de fechar

### 4. Efeitos e transições (o grande diferencial do novo site)

- **Smooth scroll** global (estilo Lenis)
- **Reveals com stagger** em todas as seções ao entrar em viewport (fade + translateY 24–32px, easing cubic-bezier suave) — moderado, nem tudo anima
- **Split-text** no headline do hero (por palavra) e headlines de seção (por linha)
- **Parallax em camadas**: formas orgânicas de fundo, fotos e conteúdo em velocidades diferentes
- **Contadores animados** se houver números/KPIs
- **Hover states ricos**: botões com preenchimento animado, cards com elevação + linha amarela, links com underline que desliza
- **Transições entre páginas** (Serviços, Segmentos, Sobre nós): fade/slide suave com curtain navy ou crossfade — nunca corte seco
- **Navbar reativa ao scroll** (item 2)
- **Ícones com draw-on animation** na seção de benefícios
- Respeitar `prefers-reduced-motion`: reduzir/desativar animações para quem tem a preferência ativa
- Performance mobile: animações via transform/opacity apenas, lazy-load de imagens fora da primeira dobra, nada de jank no scroll

### 5. Geração de leads

- Todos os CTAs amarelos levam a: formulário de contato segmentado OU WhatsApp (75) 99153-5103 com mensagem pré-preenchida contextual (ex: no CTA de ME: "Olá! Quero abrir minha ME com a Consultem")
- Formulário segmentado em /contato: primeiro pergunta o perfil (Quero abrir empresa / Já tenho empresa e quero trocar de contador / Sou profissional da saúde / Outro) e adapta os campos seguintes
- Página "Abrir conta PJ" como landing focada em conversão, seguindo a mesma identidade

### 6. Páginas

Home completa (seções acima) + páginas internas: Serviços (índice + template de serviço individual), Segmentos (Saúde, Comércio, Advogado — template único parametrizado), Sobre nós, Conteúdos (blog index + template de artigo), Abrir conta PJ, Contato. Todas seguem a mesma identidade e sistema de animação.

### 7. O que evitar

- Texto branco sobre fundo branco/claro em qualquer seção, card ou estado de hover (bug da geração anterior — inaceitável; ver regras da seção 0-A)
- Qualquer scroll horizontal na página, em qualquer breakpoint — especialmente causado pelo carousel de serviços ou por formas decorativas vazando da seção (bug da geração anterior — inaceitável; ver regras da seção 0-B)
- Sobreposição de textos em headlines animados (bug do site atual — inaceitável)
- Cards brancos genéricos com sombra pesada e cantos muito arredondados
- Ícones em círculo azul chapado estilo clipart
- Animações bounce/spring exageradas
- Overlay preto chapado sobre fotos (usar gradientes navy)
- Fotos de banco de imagens genéricas onde couber placeholder neutro (as fotos reais da equipe serão inseridas depois)
- Qualquer seção que pareça template de WordPress/Elementor

O resultado deve ser inconfundivelmente a Consultem — mesma marca, mesmas cores, mesma estrutura — mas com a execução de um estúdio digital premium.
