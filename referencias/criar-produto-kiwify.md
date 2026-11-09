# Criar e editar produto na Kiwify (pelo Chrome do Nilton)

Aprendido no Criativos do Reino (★ Cartazes Editáveis, 06/11/2026). Vale para qualquer app/marca do Nilton que venda pela Kiwify.
Primeiro uso: abrir **uma página da Kiwify por vez** (muitas seguidas/iframes deixaram o painel em branco).

## Antes de criar: questionário (sempre perguntar, mesmo que ele diga "cria o produto X")

Use a ferramenta de perguntas. Pergunte só o que não estiver decidido no projeto (veja a memória do projeto):

1. **Cobrança:** assinatura recorrente (anual? mensal?) ou pagamento único.
2. **Planos:** vai ter **Promo**? **Order Bump**? **Afiliado**? (o **Padrão** sempre existe)
3. **Preços** de cada plano (sugira pela página de vendas do produto: Promo e Order Bump mais baratos, Afiliado = Padrão).
4. **Entrega:** área de membros da Kiwify ou **externa** (app próprio do Nilton).
5. Página de vendas (link), garantia (padrão 7 dias), imagem (sugira a do produto no app, recortada 6:5).
6. Se o app precisa liberar algo: **qual grupo** o produto libera.

Mostre a lista final (nome, planos, preços, grupo) e só então crie.

## Padrões

- **Nome do produto** (Criativos do Reino): `★ Nome` = assinatura anual, `☆ Nome` = pagamento único. Não escrever "anual"/"vitalício".
- **Planos** (`Padrão - <produto>`, `Promo - <produto>`, `Order Bump - <produto>`, `Afiliado - <produto>`): regra 11 do SKILL.md.
- **Aba Links:** Padrão, Promo e Order Bump → ⋮ **Esconder dos afiliados**. O "Sales Page" que a Kiwify cria (vem do campo
  Página de vendas) → ⋮ **Editar** → nome **"Afiliado - Página de vendas"** (sem idioma no nome; o Nilton tirou o
  "(Português)" em 06/11/2026). O afiliado vê só o checkout "Afiliado - <produto>" e essa página.
- **Parcelamento:** Padrão, Promo e Afiliado em até 12x (o padrão da Kiwify); os links do site saem com **`?split=1`** (o checkout
  abre mostrando o valor cheio; parcela quem quiser). **Order Bump:** parcelamento máximo que deixa **cada parcela entre R$ 10 e
  R$ 15** (R$ 37,90 → 3x de ~R$ 12,63; R$ 29,90 → 2x) — regra do Nilton, 06/11/2026: o bump não tem `split` e mostra a
  parcela logo de cara. **O parcelamento (e a frequência) de um plano não muda depois de criado:** para trocar, exclua o
  plano (só se não tiver venda) e crie de novo; o checkout dele muda de link e precisa ser escondido dos afiliados de novo.
- Textos das imagens e do checkout: **só o preço** ("R$ 37,90"), sem "/ano" (pedido do Nilton).
- Categoria "Outros" (igual aos produtos antigos), idioma dos e-mails Português.
- **Aba Configurações (todo produto novo, Nilton 06/11/2026):** método de pagamento **"Cartão de crédito e Pix"** (sem
  boleto) e **desligar** a opção de pedir para o comprador **repetir/confirmar o e-mail**. Salvar produto.
- Suporte (e-mail, nome do produtor): **não preencher sem o Nilton passar** (dado pessoal).

## Passo a passo — criar

1. Produtos → **Criar produto** → Tipo de pagamento (Pagamento único | **Assinatura recorrente**) → Entrega (Área de membros
   da Kiwify | **Área de membros externa** | Evento | Só receber) → Continuar.
2. Nome, Descrição (mínimo 100 caracteres), Página de vendas → **Criar produto**. A URL vira `/products/edit/<ID>`:
   **guarde o ID** (é o que o app usa).
3. Geral: Categoria, Garantia, **Imagem** (upload no campo de arquivo; recomendado 300×250 → mande 600×500).
4. Preços → **Adicionar plano** (um por plano): Nome, Preço (digite só os números: 5790 = 57,90), Frequência (Anual…),
   Parcelamento (padrão 12), Renovação "Até o cliente cancelar" → Adicionar plano. Depois **Salvar produto**.
   Plano criado já gera o checkout dele na aba Links.
5. Aba **Links**: esconder dos afiliados e renomear o Sales Page (acima). Anote os checkouts (`pay.kiwify.com.br/...`).
6. Área externa não tem campo de link na Kiwify: o acesso vem do **webhook** do app. Página de obrigado → `/login` do app
   só quando o e-mail de login do app estiver configurado.
7. **No app** (Grupos e ofertas → Nova oferta): plataforma, **ID do produto**, grupo, prazo **Automático**. Uma oferta por
   produto (qualquer plano libera o mesmo grupo); "Separar por oferta" só se planos liberarem grupos diferentes.
8. Webhook: se já existe um "Todos que sou produtor" apontando para o app, o produto novo já entra; senão, criar (eventos
   na ordem da tela, regra 10). Token e endereço secreto: **o Nilton cola** (Claude não digita segredos).
9. Anote tudo no arquivo de produtos do projeto (ex.: `Area de Membros/Kiwify - produtos novos (Clube).md`) e na memória.

## Passo a passo — editar (ex.: "coloca os links de afiliado no Cartazes")

- **Novo plano** (ex.: Afiliado): Preços → Adicionar plano com o nome no padrão → Salvar produto → aba Links: o checkout
  novo aparece; esconda dos afiliados os que não são "Afiliado".
- **Renomear plano:** clique no nome do plano na tabela → Salvar alterações (o checkout continua o mesmo).
- **Mudar preço:** a Kiwify bloqueia mudar a frequência de plano existente; para preço/frequência novos, crie outro plano
  e desative o antigo na aba Links (⋮ Desabilitar) — avise o Nilton antes (quem assinou no antigo continua nele).
- **Esconder/mostrar para afiliados:** aba Links → ⋮ → Esconder dos afiliados / Mostrar aos afiliados (confira pelo menu:
  escondido mostra "Mostrar aos afiliados").
- Depois de editar: confira a aba Links (nome, tipo, preço, status) e atualize o arquivo de produtos.

## Jeito de mexer na página (o que funcionou)

- Selects da Kiwify: mudar `value` e disparar `change` (o "Área de membros externa" e "Quero apenas receber" têm o mesmo
  value: use o índice). Campos de texto: clique + digitar. Botões: `find` e clicar pela referência.
- O ⋮ abre um menu pequeno ao lado; feche com Esc. Toast "Link atualizado"/"As alterações do produto foram salvas" confirma.
- "Testar Webhook" da Kiwify pode não chegar no app; para conferir o app, mande um aviso sem assinatura (deve dar 401 e
  aparecer no registro como recusado). A prova final é uma compra de verdade (Promo) + reembolso.

## Checkout padrão (aprovado no ★ Cartazes, 06/11/2026)

Nome do checkout: **"Padrão - <produto>"**. Fundo claro (o cinza padrão da Kiwify). Nada de imagens escuras na lateral nem o
cabeçalho com foto do produto. Referência do Nilton: pay.kiwify.com.br/4KncqEA.

- **Divisão do trabalho (Nilton, 06/11/2026, para economizar):** eu crio o produto, os planos, a oferta no app e **só um
  checkout**: o "Checkout A" renomeado para **"Padrão - <produto>"** (aba Checkout → ⋮ → Configurações). O Nilton monta o
  visual (imagens, selos, pop-up) e depois ele mesmo cria o "Geral - <produto>" (cópia sem pop-up, para Promo/Order
  Bump/Pop-up). Quando ele avisar "checkouts configurados", eu entro em cada um e **só preencho os itens da Lista** do
  produto (abaixo) e salvo. Não mexo em mais nada do visual.
- **Planos novos num produto que já existe:** numerar pela ordem de criação: "Padrão 2 - <produto>", "Order Bump 2 -
  <produto>". Produto novo com só um preço inicial: "Padrão 1 - X" + "Order Bump 1 - X".
- **Configuração matriz** (ajustada pelo Nilton no ★ Cartazes, 06/11/2026 — copiar igual nos próximos):
  - **Computador** — linha do topo com 2 colunas (2/1) (aba Linhas): à esquerda o **banner**; à direita, de cima para baixo:
    1. **Imagem** `selo-garantia-horizontal.png`, pequena (~160 px de largura), alinhada à esquerda com a lista;
    2. **Lista**: "Ícone check", ícone **laranja da marca**, texto preto, fundo cinza-claro, sem título, **16 px**, alinhada à esquerda;
    3. **Selo nativo da Kiwify** (o Nilton preferiu ao nosso de compra segura): tipo "Privacy", título "Segurança",
       subtítulo "Garantida", **cor principal cinza #555555**, cor do título branca, centralizado.
  - **Celular** (botão do celular no topo do editor; ajustar lá, não só "Copiar do desktop"): banner em cima, formulário, e
    depois selo de garantia (~180 px, **centralizado**), Lista **centralizada** (16 px, cabe cada item numa linha) e o Selo.
  - Não usar: o card "Vantagens" da Kiwify nem `selo-compra-vertical.png` / privacidade (ficam guardados).
  - **Lista** = o volume da oferta da página de vendas, um item por linha, até 10. Cartazes: "+700 cartazes editáveis",
    "App no celular e no PC", "Cartazes novos todo mês", "Aulas rápidas e suporte", "Bônus: Fotos com IA",
    "Bônus: Títulos 3D com IA", "Bônus: Fundos com IA", "Bônus: Removedor de fundo", "Bônus: Calendário da Igreja".
    Feitas 06/11 (computador e celular): Carrosséis Pós-Culto, Conteúdo, Identidade e Telão — itens tirados do bloco
    "Tudo o que entra no seu acesso" da página. Celular e computador têm listas separadas: trocar nas duas versões
    (botão do celular no topo do editor) e salvar em cada uma. Itens curtos (~30 letras). **Atenção:** o preço da página
    é o "Hoje você paga só", não o primeiro "R$" que aparecer (o Telão tem bônus de R$ 19,90 e preço R$ 10,00).
    **Sempre o nome do produto, nunca "modelos"/"artes"** ("Cartazes novos todo mês", "Carrosséis novos todo mês").
    **★ Clube (06/11):** checkout "Padrão - Combo Geral" (serve para todos os planos por enquanto; depois vem um só de packs). Lista: 5 packs de artes no app · +700 cartazes editáveis · Carrosséis e identidade · Telas para o telão · 5 cursos e treinamentos · Curso de IA na criação · Fotos e fundos com IA · Títulos 3D com IA · Removedor de fundo · Fatiador e Calendário. Banner com as cores da página (degradê lima→ouro→laranja), `Produtos/Clube/Kiwify/checkout/banner.py`.
    Pela automação: setar o value com o setter nativo + evento `input` (digitar campo a campo embaralha a ordem).
  - **Nome do checkout:** o campo de nome dentro do editor NÃO grava (volta para "Checkout A"). Renomear em Produto → aba
    **Checkout** → ⋮ → **Configurações** → Nome → Salvar alterações.
- **Banner** (~1600×740): fundo + mockup da oferta feitos no **Codex** (o leque da página de vendas, sem texto, lado esquerdo
  livre); texto escrito por cima em HTML (`banner.py`): título em duas linhas ("+700 Cartazes" / "Editáveis" no degradê),
  uma frase curta, 3 pontos de confiança com ✓ alinhados à esquerda (Acesso imediato no app · Compra 100% segura ·
  Garantia de 7 dias) e "kiwify | COMPRA SEGURA" embaixo. **Sem logo**, só o preço (nunca "/ano"), sem sobra em cima/embaixo.
  Arquivos no produto: `Produtos/Packs/<pack>/Kiwify/checkout/` (gerar/prompt-banner.txt, banner.py, banner.jpg).
- **Selos oficiais** (os mesmos em todo checkout, não um por produto): `Produtos/Checkout Kiwify/selos/` (`selos.py` →
  um PNG transparente por selo em duas versões, `selo-<garantia|compra|privacidade>-horizontal.png` (ícone à esquerda) e
  `-vertical.png` (ícone em cima, texto embaixo), + `selos-linha.png`/`selos-linha-vertical.png` e ícones SVG). Formato da
  referência do Nilton: ícone chapado cinza-escuro com recortes **vazados** (transparentes, por máscara) + texto em duas linhas:
  Garantia / **de 7 dias** (troféu) · Compra / **Segura** ($ com escudo) · Privacidade / **Protegida** (cadeado) — linha de
  cima normal, linha de baixo em **negrito**; na vertical, uma 3ª linha fina e menor, complementar, numa linha só:
  "Experimente sem riscos" · "Pagamento criptografado" · "Seus dados em sigilo". No checkout entra só o de **garantia horizontal**; a segurança é o selo nativo da Kiwify em cinza. Recusados (06/11/2026):
  selo 3D/"cartoon" do Codex, selo redondo com texto em anel, selo de escudo com faixa, card "Vantagens" da Kiwify.
- **Pop-up de saída** (Componentes extra → Exit Popup, ligar a chave; só na visão do computador): "Imagem + texto",
  Ação ao clicar **"Oferecer desconto"** → Selecionar oferta = plano **"Pop-up - <produto>"** = Padrão **menos R$ 10**
  (Cartazes 57,90 → 47,90 = 17% OFF; o Promo é mais barato e fica para WhatsApp/campanhas). Plano anual, 12x, escondido
  dos afiliados. Textos curtos (a pessoa já sabe o que leva): título **"Espera, antes de sair!"** (preço no título quebra a
  linha); texto "Condição especial para você começar hoje: todo o material, com R$ 10 de desconto. De R$ 57,90 por
  R$ 47,90."; botão "Quero por R$ 47,90". Tom: condição especial, sem "você foi premiado"/"selecionamos você" (forçado).
  Imagem retrato 600×900 (`popup.py` → popup.jpg): selo "17% OFF · R$ 10 DE DESCONTO", nome do produto, leque da oferta,
  "de R$ X" riscado e o preço do Pop-up grande no degradê. Trocar imagem: lixeira da imagem no painel → upload → Salvar corte.
  Sem WhatsApp no checkout.
- Order bumps: Produto → aba **Configurações** → seção **Order bump** → "Adicionar order bump" (Produto + Oferta "Order Bump - X" + Call to action + Título + Descrição; salva sozinho, até 5; o último adicionado fica em 1º). Feito 06/11: Cartazes (Telão, Identidade, Conteúdo, Carrosséis) e Carrosséis (Cartazes, Conteúdo, Identidade, Telão). Texto: "Sim, quero levar junto!" / nome do pack / "Leve também ...". O Nilton desligou o pop-up de saída por enquanto (desconto aparecia ao mexer o mouse).

**Editor de checkout por automação:** é SortableJS — arrastar com o mouse não funciona; simular `pointerdown`/`mousedown` no
componente (`#component_drag_<tipo>`: text, image, advantages, seal, header, list, countdown, review, video) + `DragEvent`
`dragstart` → `dragenter`/`dragover` até a zona → `drop` → `dragend`, com um `DataTransfer`. Linhas: o mesmo, a partir do
desenho da linha na aba Linhas. Imagem: componente → upload no input de arquivo do painel → "Salvar corte" (a imagem inteira
já vem marcada). Sempre "Salvar checkout" e esperar "Tudo certo!" (o upload leva alguns segundos). Uma aba de editor por vez.
