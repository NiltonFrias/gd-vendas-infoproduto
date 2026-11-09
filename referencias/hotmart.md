# Hotmart — webhook (postback) versão 2.0.0

Exemplo completo: `../exemplos/hotmart-aprovado-pix.json`.

## Autenticação

- Cabeçalho `X-HOTMART-HOTTOK` igual ao hottok do painel da Hotmart (Ferramentas → Webhook → Autenticação).
  Na versão 1 ele vinha no corpo (`hottok`). Compare em tempo constante.
- Um mesmo endereço recebe todos os produtos da conta: identifique o produto por `data.product.id`.

## Configurar na Hotmart (quais eventos marcar)

Caminho: **Ferramentas → Webhook (versão 2.0)** → cadastrar o endereço do app → copiar o **hottok**.

- **Área de membros** (liberar e tirar acesso): Compra aprovada, Compra completa, Reembolso, Chargeback, **Pedido de
  reembolso** e Cancelamento de assinatura. Esqueceu o Pedido de reembolso? O acesso só cai quando o reembolso sai.
- **Apps de vendas/rastreio** (Pixel, WhatsApp): todos os de compra, inclusive Boleto/Pix impresso
  (`PURCHASE_BILLET_PRINTED`), Compra cancelada, Compra atrasada, Abandono de carrinho e Compra expirada.
- Texto pronto da tela (área de membros), dentro de "Como configurar na Hotmart" recolhido: *Na Hotmart:
  **Ferramentas → Webhook (versão 2.0)**. Cole o endereço acima e marque **Compra aprovada, Compra completa, Reembolso,
  Chargeback, Pedido de reembolso e Cancelamento de assinatura**. Copie o **hottok** e cole aqui embaixo.*
- A ordem dos eventos na tela da Hotmart ainda não foi conferida por print (regra 10 do SKILL.md). Quando tiver o print,
  ponha os eventos aqui na ordem da tela.

## Estrutura principal

```
event                      PURCHASE_APPROVED | PURCHASE_BILLET_PRINTED | ...
creation_date              ms epoch do aviso
data.product.id / .name    ID numérico (estável) e nome (pode mudar)
data.buyer                 email, name, first_name, last_name, checkout_phone, checkout_phone_code, document, address{country_iso,...}
data.commissions[]         { source: PRODUCER | MARKETPLACE | AFFILIATE | COPRODUCER, value, currency_value }
data.purchase.transaction  "HP..." — chave da venda (vários avisos com a mesma transação)
data.purchase.status       APPROVED, BILLET_PRINTED, ...
data.purchase.price.value  quanto o comprador pagou (reais, decimal); full_price / original_offer_price também
data.purchase.payment      { type, installments_number, pix_code, pix_qrcode, pix_expiration_date, billet_url, refusal_reason }
data.purchase.approved_date / order_date   ms epoch
data.purchase.order_bump   { is_order_bump, parent_purchase_transaction }
data.purchase.origin       { src, sck, xcod } — rastreio que veio no link do checkout (ver gd-pixel-meta/referencias/rastreio.md)
data.purchase.recurrence_number            > 1 = renovação de assinatura
data.purchase.checkout_country.iso
data.subscription          { status, plan{ id, name }, subscriber{ code } } (quando é assinatura)
```

## Pix × boleto (atenção)

`PURCHASE_BILLET_PRINTED` vale para os dois. `payment.type` pode ser `BILLET`, `PIX` ou `PIX_AUTOMATIC`
(visto em produção em out/2026). Regra: **contém "PIX" → Pix; senão → boleto**. Outros tipos:
`CREDIT_CARD`, `PAYPAL`, `GOOGLE_PAY`… Para agrupar formas de pagamento: Pix = /PIX/, Cartão = /CREDIT|CARD/,
Boleto = /BILLET|BOLETO/.

## Valor

- Faturamento do dono = `commissions` com `source = "PRODUCER"` (já sem taxa da Hotmart). Sem ela, use
  `purchase.price.value`.
- O mesmo vale para o valor mandado à Meta, se a regra do pixel for "comissão".

## Situação da venda (máquina de estados)

Em ordem de chegada, por transação:

- `BILLET_PRINTED` → **aguardando pagamento** (Pix/boleto). Se depois chegar `APPROVED` → paga ("gerado ·
  pago depois"). Se chegar `CANCELED`/`EXPIRED` → **não paga**.
- `APPROVED` → **aprovada**.
- `PROTEST` → **pedido de reembolso**: a venda **sai** do faturamento e das contagens.
  - Se depois chegar `COMPLETE` ou um novo `APPROVED` → o pedido foi cancelado: a venda **volta**.
- `REFUNDED` / `CHARGEBACK` → **reembolsada** / **chargeback**: sai para sempre.
- `COMPLETE` → garantia acabou; a venda já tinha sido contada na aprovação (não conte de novo).
- `CANCELED` no **cartão** → **recusada** (a Hotmart não tem evento próprio de recusa).
- `CANCELED` no **Pix/boleto** → **não pago** (cancelado). Atenção: o Pix não pago também chega com
  `payment.refusal_reason = "Transaction refused"` — **não** use o refusal_reason sozinho para dizer que foi cartão recusado.
- `OUT_OF_SHOPPING_CART` → carrinho abandonado (não tem transação; use e-mail/telefone + produto como chave).

Venda paga e depois reembolsada **não** volta a "pendente".

## Order bump

- O bump é **outra transação** com aviso próprio. `order_bump.is_order_bump = true` e
  `parent_purchase_transaction` = transação da compra principal.
- Compra principal de checkout com bump: `order_bump` existe com `is_order_bump = false` → pode chegar um bump.
- A ordem dos avisos é aleatória. Para somar numa Compra só (Meta), espere ~60 s pela chegada do bump.
- No Pix/boleto, os itens do mesmo checkout saem com o **mesmo** `pix_code`/boleto: ótimo para agrupar.

## Telefone do comprador

- Nos pedidos, o DDD costuma vir separado: `checkout_phone = "999999999"`, `checkout_phone_code = "31"`.
- Às vezes o número já vem com o DDD repetido ou com um 9 a mais ("79 9 99157-1637"): normalize.
- No abandono, `buyer.phone` costuma vir já com DDI.
- Comprador brasileiro: o número final deve ser 55 + DDD + 8/9 dígitos; fora disso, descarte.

## Produto renomeado

`data.product.id` não muda quando o produtor renomeia. Agrupe por ID e mostre o nome do aviso mais recente.

## Outros

- `recurrence_number > 1` = renovação: evento próprio (renovação paga / recusada), não "compra aprovada".
- Hotmart também tem integração própria de pixel/API de Conversões no produto: se estiver ligada junto com o
  seu app, a Meta recebe Compras em dobro (ver gd-pixel-meta/referencias/compra-capi.md).
