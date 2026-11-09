---
name: gd-vendas-infoproduto
description: Como ler e tratar webhooks de vendas de infoproduto (Hotmart, Kiwify, Greenn) — eventos, Pix × boleto, reembolso, order bump, comissão, rastreio (src/sck/xcod) e envio de Compra à Meta (CAPI). Use sempre que o código receber, interpretar, guardar ou reagir a um aviso de venda dessas plataformas, ou quando um app mostrar/mandar algo errado sobre uma venda (ex.: "Pix virou boleto", "venda reembolsada contando", "produto aparecendo duas vezes"). Use também para criar ou editar produto na Kiwify (planos Padrão/Promo/Order Bump/Afiliado, checkout, links de afiliado, oferta no app) — sempre com questionário antes.
---

# Vendas de infoproduto: como ler os webhooks

Conhecimento comum aos apps do Nilton (Pixel.Afilic, Zap.Afilic e os próximos). O que já foi aprendido na
prática vale para todos: ao corrigir a leitura num app, atualize esta skill (e o outro app).

## Regras de ouro

1. **Guarde o aviso bruto** (corpo inteiro) antes de interpretar. A plataforma pode mandar o mesmo aviso
   mais de uma vez e em qualquer ordem: trate cada aviso como idempotente (plataforma + transação + evento).
2. **Uma venda = uma transação**, e vários avisos ao longo do tempo (Pix gerado → aprovada → pedido de
   reembolso → reembolsada). A situação atual sai da sequência de avisos, não de um aviso isolado.
3. **Pix × boleto:** na Hotmart os dois chegam como `PURCHASE_BILLET_PRINTED`. Decida pelo
   `purchase.payment.type` procurando **"PIX" dentro do texto** — existem `PIX` **e `PIX_AUTOMATIC`**.
   Comparar com `=== "PIX"` transforma Pix em boleto (bug real visto no Zap.Afilic).
4. **Produto = ID da plataforma**, não o nome. O produtor pode renomear o produto (ex.: "Kit Editável Gospel
   App" → "Kit Cartaz Gospel", mesmo ID 8647293): agrupe e filtre pelo ID e mostre o nome mais recente.
5. **Valor que fica para o dono** = comissão do produtor (`commissions[source=PRODUCER]` na Hotmart), não o
   preço pago. Use o preço só quando não houver comissão.
6. **Reembolso tira a venda das contas**, inclusive quando é só pedido (veja a máquina de estados em
   `referencias/hotmart.md`).
7. **Order bump é outra transação**, com aviso próprio que chega em ordem aleatória (pode vir antes da
   principal). Ligue pela transação "pai" quando a plataforma informar.
   Nas telas que agrupam por venda, **agrupe por compra (checkout)**, não por transação: o bump fica no grupo da
   principal, marcado "(bump)" (Hotmart: `order_bump.parent_purchase_transaction`; Kiwify: mesmo comprador em até
   10 min, regra em `referencias/kiwify.md`). Pedido do Nilton, 18/10/2026.
8. Datas: Hotmart em **milissegundos (epoch)**; Kiwify em texto no horário de Brasília. Valores: Hotmart em
   reais (decimal); Kiwify em **centavos**.
9. **Nome da pessoa no padrão** antes de mostrar ou usar em mensagem (WhatsApp, e-mail, painel): o comprador
   digita como quer ("VIVIANE DE OLIVEIRA", "joão dos santos"). Mostre "Viviane de Oliveira": primeira letra de
   cada palavra maiúscula; `de, da, do, das, dos, e` minúsculas no meio; vale depois de hífen e apóstrofo
   (Ana-Maria, D'Ávila); romanos ficam maiúsculos (Pedro II). **Só mexa em palavra toda maiúscula ou toda
   minúscula** — quem escreveu "McDonald" fica como está. Guarde o aviso bruto sem mexer; trate na leitura.
   Para chamar pelo primeiro nome numa mensagem, use a primeira palavra já tratada. Referência:
   `nomeProprio` em `src/util.js` do Pixel.Afilic (com testes em `test/hotmart.test.js`).

10. **Ao dizer quais eventos marcar no webhook, cite só os que marcar, na ORDEM DA TELA da plataforma** (Nilton,
   06/11/2026; enxugado em 07/11/2026 no FluxoPost e levado ao Criativos do Reino em 09/11/2026): a pessoa vai descendo e
   marcando sem procurar. **Na tela do app**, isso fica num "Como configurar na <plataforma>" **recolhido** (`ajuda()` da
   skill `gd-layout-apps`), numa frase só: onde clicar → "marque só: A, B, C… Os outros eventos ficam desmarcados" → onde
   está o token. **Não** desenhe a lista inteira de eventos (Boleto gerado, Pix gerado, Carrinho abandonado…) com ✓: ocupa a
   tela e confunde. Em passo a passo ou mensagem, pode citar os desmarcados em uma linha, se ajudar.
   Eventos a marcar em cada plataforma: `referencias/<plataforma>.md` (Kiwify confirmada pelo print; Hotmart: nomes
   usados nos apps, ordem ainda sem print; Greenn: sem estudo).

11. **Nome dos planos/ofertas** (padrão do Nilton, 06/11/2026): cada produto com 5 planos — **Padrão - <produto>**
   (link normal), **Promo - <produto>** (recuperação no WhatsApp, dentro da área de membros), **Order Bump - <produto>**
   (só para o bump) **Afiliado - <produto>** (checkout só dos afiliados) e **Pop-up - <produto>** (Padrão − R$ 10, desconto do pop-up de
   saída do checkout). Assim dá para saber de qual link veio cada
   venda. Produtos novos do Criativos do Reino: ★ no nome = assinatura anual, ☆ = pagamento único.
   Na aba **Links** da Kiwify: Padrão, Promo, Order Bump e Pop-up ficam **escondidos dos afiliados** (⋮ → Esconder dos afiliados);
   o afiliado vê só o checkout "Afiliado - <produto>" e a página de vendas, renomeada para **"Afiliado - Página de vendas"**
   (o "Sales Page" automático, ⋮ → Editar; sem idioma no nome).
   **Criar ou editar produto na Kiwify** (questionário antes, passo a passo, edição): `referencias/criar-produto-kiwify.md`.

12. **Oferta de assinatura na Kiwify** (confirmado 06/11/2026): `Product.product_offer_name` vem **null**; o plano vem
   em `Subscription.plan` (`name`, `id`) e o link em `checkout_link` (só o código, ex. "S7yH7oK"). Para separar planos,
   case pelo nome do plano **ou** pelo código do checkout. Cada **order bump** chega como **venda separada** (outro
   `order_id`, outro produto); reembolsar o principal manda "assinatura cancelada" do bump, mas não reembolsa o bump.

13. **Página de links da equipe** (`criativosdoreino.com/ofertas/`, noindex, branca e simples): cada produto com a página
   e os checkouts Padrão/Promo (com `?split=1`), botão Copiar, e gerador de link rastreado (`src` = canal, `sck` =
   palavra-chave sem acento). Fonte dos links: aba **Links** de cada produto na Kiwify (o preço não vem no HTML do
   checkout). Arquivo `Paginas de Vendas/Links/pagina/index.html` (lista `PRODUTOS` no script).

## Vocabulário único (use estes nomes no código novo)

| Evento normalizado | Significado | Hotmart | Kiwify |
|---|---|---|---|
| `carrinho_abandonado` | saiu do checkout sem comprar | `PURCHASE_OUT_OF_SHOPPING_CART` | corpo com `status: "abandoned"` e sem `order_id` |
| `pix_gerado` | gerou o Pix, ainda não pagou | `PURCHASE_BILLET_PRINTED` + tipo contém PIX | `pix_created`, ou `waiting_payment` sem ser boleto |
| `boleto_gerado` | gerou o boleto, ainda não pagou | `PURCHASE_BILLET_PRINTED` + tipo BILLET | `billet_created`, ou `waiting_payment` + `payment_method: boleto` |
| `aprovada` | pagamento confirmado | `PURCHASE_APPROVED` | `order_approved` / status `paid` |
| `recusada` | cartão recusado | `PURCHASE_CANCELED` **com pagamento em cartão** (nunca Pix/boleto) | `order_rejected` / `refused` |
| `cancelada` | cancelada | `PURCHASE_CANCELED` | — |
| `expirada` | Pix/boleto venceu sem pagar | `PURCHASE_EXPIRED`, `PURCHASE_DELAYED` | — |
| `pedido_reembolso` | cliente pediu o dinheiro de volta | `PURCHASE_PROTEST` | status `refund_requested` |
| `reembolsada` | dinheiro devolvido | `PURCHASE_REFUNDED` | `order_refunded` / `refunded` |
| `chargeback` | contestação no cartão | `PURCHASE_CHARGEBACK` | `chargeback` / `chargedback` |
| `garantia_encerrada` | acabou o prazo de reembolso (venda confirmada) | `PURCHASE_COMPLETE` | — |

Avisos que **não são compra nova** (ignorar nas automações de compra): Hotmart `SUBSCRIPTION_CANCELLATION`,
`SWITCH_PLAN`, `UPDATE_SUBSCRIPTION_CHARGE_DATE`, `CLUB_*`; Kiwify `subscription_*` (trazem o pedido antigo
com status `paid` — não podem disparar "compra aprovada").

**Renovação de assinatura** (plano mensal/anual, inclusive order bump com recorrência) é evento próprio —
`renovacao_paga` e `renovacao_recusada` —, nunca "compra aprovada"/"cartão recusado". Hotmart:
`purchase.recurrence_number > 1` (e `PURCHASE_DELAYED` com recorrência > 1 = renovação recusada). Kiwify:
`Subscription.charges.completed` tem cobrança `paid` de **outro** `order_id`. Confirmado no Zap.Afilic
(24/10/2026): a Kiwify cria um pedido novo a cada tentativa e, com o cartão recusando, tenta **todo dia**
(~8h) — sem isso o cliente recebeu "quer ajuda para finalizar?" 5 dias seguidos, e quem renovou recebeu de
novo as boas-vindas de primeiro acesso. Aviso de renovação recusada: no máximo um por semana por contato;
renovação paga cancela o aviso pendente e não conta como cliente novo nos painéis.

Nomes para mostrar ao usuário (pt-BR): Compra aprovada · Pix/boleto gerado (ou "Pix gerado" / "Boleto
gerado") · Pedido de reembolso · Reembolsada · Chargeback · Cancelada · Pagamento expirado · Garantia
encerrada · Carrinho abandonado.

## Onde está cada detalhe

- `referencias/criar-produto-kiwify.md` — **criar e editar produto na Kiwify**: questionário antes (afiliado? promo? order bump?
  preços, entrega, grupo), padrões de nome, planos, links de afiliado, oferta no app, webhook.

- O que a venda **muda no acesso** do comprador (criar conta, grupos, prazo, bloqueio) fica na skill
  `gd-acesso-membros`.

- `referencias/hotmart.md` — campos do webhook 2.0.0, autenticação, Pix/boleto, comissão, order bump,
  reembolso, telefone, renovação, produto renomeado.
- `referencias/kiwify.md` — eventos, centavos, Pix no campo do boleto, assinatura HMAC, carrinho abandonado.
- `referencias/greenn.md` — ainda sem aviso real estudado (o que fazer até lá).
- Rastreio (src / sck / xcod, ID do visitante) e a Compra na API de Conversões ficam na skill **`gd-pixel-meta`**
  (`referencias/rastreio.md` e `referencias/compra-capi.md` de lá).
- `exemplos/` — avisos reais com os dados pessoais trocados. Use nos testes.

## Onde isso está implementado hoje

- Pixel.Afilic (Node/Express, JS): `src/adaptadores/hotmart.js`, `kiwify.js`, `greenn.js`; situação da venda
  em `src/painel/vendas.js`; reembolso nas contas em `src/painel/dashboard.js`; produto↔campanha em
  `src/produto-campanha.js`.
- Zap.Afilic (Next/Prisma, TS): `src/lib/sales/hotmart.ts`, `kiwify.ts`, `types.ts` (formato `SalesEvent`),
  `order-context.ts` (produto + order bump no mesmo checkout).

## Pendências conhecidas

- Feito e publicado no Zap.Afilic (17/10/2026): `hotmart.ts` usa `payType.includes("PIX")` (pega `PIX_AUTOMATIC`);
  `PURCHASE_CANCELED` só vira `refused` quando o tipo contém `CARD` — a Hotmart manda `refusal_reason:
  "Transaction refused"` também no **Pix não pago** (5 de 5 casos reais em out/2026); nome no padrão em
  `src/lib/name.ts` (`properName`/`firstName`, com testes em `src/lib/name.test.ts`), aplicado na leitura
  Hotmart/Kiwify/outra e também em `{nome}`/`{primeiro_nome}` das mensagens (`src/lib/template.ts`), para
  contatos antigos gravados em maiúsculas.
- Zap.Afilic: vendas Pix antigas já gravadas como `refused` continuam assim no banco (o painel as conta como
  "Cartão recusado"); a correção vale para os avisos novos.
- Pixel.Afilic: adaptadores da Kiwify e da Greenn só guardam o aviso (não traduzem a venda ainda).
- Feito no Pixel.Afilic (17/10/2026): telefone pronto para a Meta (`telefoneDe` em `src/adaptadores/hotmart.js`,
  DDD separado e 9 a mais; número que não fecha fica vazio), nomes "Pix gerado" / "Boleto gerado" / "Recusada"
  pelo tipo de pagamento (`varianteDe` em `src/painel/layout.js`). Nos avisos reais, 24 de 25 telefones já vêm com
  DDD (11 dígitos); 1 veio com 12 dígitos (o 9 a mais).
