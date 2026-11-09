# Kiwify — webhook

Fonte: leitor do Zap.Afilic (`src/lib/sales/kiwify.ts`), testado com avisos reais.

## Autenticação

- Assinatura HMAC-SHA1 do **corpo bruto** com o token do webhook, no parâmetro de URL `?signature=`.
  Calcule sobre o texto exatamente como chegou (antes de fazer JSON.parse) e compare em tempo constante.

## Pedido

```
order_id                 chave da venda
webhook_event_type       order_approved | pix_created | billet_created | order_rejected | order_refunded | chargeback | subscription_*
order_status             paid | waiting_payment | refused | refund_requested | refunded | chargedback
payment_method           boleto | pix | credit_card ...
Product                  { product_id, product_name, product_offer_name }
Customer                 { full_name, first_name, email, mobile, CPF, cnpj }
Commissions.charge_amount  valor em CENTAVOS
boleto_URL               link do boleto — ou, no Pix, o código "copia e cola" (começa com 000201)
pix_code                 costuma vir vazio (o Pix vem em boleto_URL)
checkout_link            às vezes só o código (ex.: "gqHVNDn") → https://pay.kiwify.com.br/<código>
access_url / Subscription.access_url
approved_date, created_at, updated_at   texto "2025-10-09 14:33" no horário de Brasília (ou ISO)
Subscription.plan.name   nome do plano  ⚠ na ASSINATURA o Product.product_offer_name vem null: ler o plano aqui (e o código em checkout_link) — confirmado 06/11/2026 no Clube
```

Regras:

- Evento: primeiro `webhook_event_type`; se não houver e `order_status = waiting_payment`, é Pix ou boleto
  conforme `payment_method`; senão use `order_status`.
- `subscription_*` traz o pedido antigo com status `paid`: **não** é compra nova.
- Pix "copia e cola" = texto que começa com `000201` (padrão do Banco Central).
- **Order bump: a Kiwify não informa.** Regra do Nilton (18/10/2026): pedidos aprovados do **mesmo comprador (e-mail)
  com até 10 min de diferença** são um checkout só. 1) Oferta com a palavra **"Bump"** no nome é bump (o Nilton vai
  nomear as ofertas: "Bump - …" para bump, o nome do produto para principal). 2) Sem "Bump" no nome, **o de maior
  valor é o principal** e os mais baratos são bumps. O valor do bump entra na Compra do principal (uma Compra só); a
  Compra espera 1 minuto pelo bump. Pixel.Afilic: `juntarBumpKiwify` em `src/fila.js`.

## Carrinho abandonado

Formato próprio, **sem** `order_id`: `{ id, status: "abandoned", name, first_name, email, phone, cpf,
product_id, product_name, checkout_link, created_at }`.
O nome vem em `name` (não em `Customer.full_name`): converta também o carrinho para o formato que o painel lê, senão
o cliente aparece sem nome (erro real no Pixel.Afilic, 18/10/2026). O ID do produto da Kiwify é um UUID (com letras):
ao esconder o ID na tela, não use só "número · nome".

## Pixel.Afilic (18/10/2026)

Leitor em `src/adaptadores/kiwify.js` (testes em `test/kiwify.test.js`), montado desta referência e do Zap.Afilic:
- O aviso é guardado no **formato do aviso da Hotmart** (evento `PURCHASE_*`, `data.purchase` / `data.buyer` /
  `commissions`), com o original em `kiwify`: o painel (Resumo, Vendas, Eventos) lê as duas plataformas igual.
- Corpo bruto guardado no `express.json({ verify })` para a assinatura (`req.corpoBruto`); Token cifrado no painel
  (Integrações → Webhooks → Kiwify), como o hottok da Hotmart.
- Sem `xcod` na Kiwify: ele é montado dos `utm_*` do parâmetro de URL (`utm_campaign` = "nome|id"…).
- **Confirmado no aviso de teste da Kiwify (18/10/2026):**
  - `Commissions`: `charge_amount` (preço pago), `product_base_price`, `kiwify_fee`, **`my_commission`** (a comissão
    do produtor, já sem a taxa), `settlement_amount` (**= preço cheio, não é a comissão**), `commissioned_stores`
    [{ type: producer | coproducer | affiliate, value (texto, centavos) }]. Comissão = `my_commission`; senão a do
    `type: producer`; senão o preço.
  - `Customer`: full_name, first_name, email, mobile, cnpj/CPF, **ip, city, state, zipcode**, street, number…
  - `TrackingParameters`: src, sck, utm_source/medium/campaign/content/term e **s1, s2, s3**.
    Os utm_* chegam também **sem** "nome|id" quando a visita não veio de anúncio (caso real 18/10/2026: `utm_source=ig`,
    `utm_medium=social`, `utm_content=link_in_bio` = link na bio do Instagram, venda de afiliado). Não descarte: é a
    origem orgânica da venda.
  - `Customer.ip`: IP do comprador no checkout (preenchido nos avisos reais). Serve para a Compra (client_ip_address) e
    para achar a visita na página quando o sck não trouxe o identificador.
  - Também: `product_type` (membership…), `card_type`, `card_last4digits`, `installments`, `sale_type`,
    `refunded_at`, `Subscription { id, start_date, next_payment, status, plan, charges }`, `subscription_id`.
  - **Datas:** o teste veio em **UTC** (created_at = hora do envio em UTC), não em Brasília. Leia como Brasília;
    se cair no futuro, leia como UTC; e nunca depois de agora (a Meta recusa evento no futuro). Confirme na venda real.
- **Confirmado em avisos reais (Pix gerado, 18/10/2026, Criativos do Reino):** datas no **horário de Brasília** (só o
  teste vem em UTC); `TrackingParameters` com `src` = nome da campanha e `utm_campaign/medium/content` = "nome|id",
  `utm_term` = posicionamento (o xcod montado dos utm funciona); `sck` nulo quando o cliente não passou por página com
  o script. Ofertas vistas: "Order - Carrossel" e "Order - Pack Canva" chegando no mesmo minuto (possível order bump:
  a Kiwify não informa; combinar com o Nilton como reconhecer).
- **Testes de todos os eventos (18/10/2026):** pix_created, billet_created, order_rejected (`card_rejection_reason`,
  ex.: possible_fraud_ip), order_refunded (`refunded_at`), subscription_renewed (status paid) e carrinho abandonado
  (chaves: checkout_link, country, cnpj, created_at ISO, email, id, name, offer_name, phone, product_id, product_name,
  status, store_id, subscription_plan, TrackingParameters) — todos lidos certo.
- **Confirmado (24/10/2026):** renovação chega como `order_approved` / `order_rejected` com `order_id` NOVO e
  `Subscription.charges.completed` listando a cobrança paga anterior (outro `order_id`). Recusada = nova
  tentativa (pedido novo) todo dia por volta das 8h. Primeira compra: `completed` vazio ou só o próprio pedido.

## Vários webhooks (um por produto)

A Kiwify não deixa escolher alguns produtos num webhook: é "todos que sou produtor", "todos que sou afiliado" ou um
produto. Quem separa produtos em projetos (ex.: Brasil × gringa) cria **um webhook por produto**, todos com o mesmo
endereço, e **cada webhook tem um token próprio**. O app guarda uma lista de tokens (com o nome do produto, cifrados)
e aceita o aviso cuja assinatura HMAC bater com qualquer um; marca qual webhook recebeu (último aviso de cada um).
Na tela: escolha "Um webhook para todos os produtos" × "Um webhook por produto" antes do endereço.
Pixel.Afilic (05/11/2026): `cadastro.webhooksKiwify()`, `src/adaptadores/kiwify.js` → `autenticar`, tela em
`src/painel/plataformas.js`.

## Próxima cobrança e várias contas (Criativos do Reino, 06/11/2026)

- `Subscription.next_payment` (texto no horário de Brasília, ex.: "2027-11-06 10:00") = até quando o acesso vale; o app
  soma 3 dias de folga. Renovação (pedido novo, `order_approved`) traz a data nova.
- Várias contas da Kiwify no mesmo app: cada conta com endereço próprio e os seus tokens (skill `gd-acesso-membros`,
  "Plataformas de venda como contas"). Na Kiwify, a área de membros externa não tem campo de link: o acesso vem do
  webhook; a página de obrigado pode levar ao /login do app.

## Eventos do webhook: ordem da tela (print do Nilton, 06/11/2026)

Na tela "Criar webhook" da Kiwify, de cima para baixo: Boleto gerado · Pix gerado · Carrinho abandonado · Compra recusada ·
**Compra aprovada** · **Reembolso** · **Chargeback** · **Assinatura cancelada** · Assinatura atrasada · **Assinatura renovada**.
Área de membros (liberar/tirar acesso): marque os 5 em negrito. A renovação já chega como Compra aprovada (pedido novo), mas
"Assinatura renovada" é a garantia (pedido do Nilton, 06/11/2026): no Criativos do Reino ela só empurra o vencimento para a
próxima cobrança (`Subscription.next_payment` + folga), sem somar dias, então repetir não estraga. Apps de vendas/rastreio (Pixel.Afilic, Zap.Afilic) usam
mais (Pix e boleto gerados, carrinho abandonado, recusada): diga sempre nessa ordem.
