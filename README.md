# gd-vendas-infoproduto

Skill do Claude Code com o conhecimento comum para ler webhooks de vendas de infoproduto (Hotmart, Kiwify,
Greenn) e mandar Compras à Meta. Usada pelos apps Pixel.Afilic e Zap.Afilic e por qualquer projeto novo.

- `SKILL.md`: regras de ouro, vocabulário único de eventos e onde está cada detalhe.
- `referencias/`: uma página por plataforma (Hotmart, Kiwify, Greenn) e como criar produto na Kiwify.
- `exemplos/`: avisos reais com os dados pessoais trocados, para testes. **Nunca** salve aqui um aviso com
  e-mail, telefone, CPF ou nome de cliente de verdade.

A skill cita outras duas skills GD que não estão neste repositório: `gd-acesso-membros` (o que a venda libera
para o comprador) e `gd-pixel-meta` (rastreio e Compra na API de Conversões da Meta).

## Instalar

No terminal, baixe a pasta para dentro das skills do Claude Code:

```bash
git clone https://github.com/NiltonFrias/gd-vendas-infoproduto.git ~/.claude/skills/gd-vendas-infoproduto
```

No Windows (PowerShell):

```powershell
git clone https://github.com/NiltonFrias/gd-vendas-infoproduto.git "$env:USERPROFILE\.claude\skills\gd-vendas-infoproduto"
```

Abra o Claude Code de novo: ele carrega a skill sozinho. Para usar, peça algo como "vamos integrar a Hotmart
usando a skill gd-vendas-infoproduto".

## Atualizar

Para receber as novidades: `git pull` dentro da pasta da skill.

Ao corrigir ou descobrir algo na leitura de um webhook em qualquer app, registre aqui (e na lista de
pendências, se o outro app ainda precisar da mesma correção).
