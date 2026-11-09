# gd-vendas-infoproduto

Skill do Claude Code com o conhecimento comum para ler webhooks de vendas de infoproduto (Hotmart, Kiwify,
Greenn) e mandar Compras à Meta. Usada pelos apps Pixel.Afilic e Zap.Afilic e por qualquer projeto novo.

- `SKILL.md`: regras de ouro, vocabulário único de eventos e onde está cada detalhe.
- `referencias/`: uma página por plataforma, mais rastreio (src/sck/xcod) e API de Conversões da Meta.
- `exemplos/`: avisos reais com os dados pessoais trocados, para testes. **Nunca** salve aqui um aviso com
  e-mail, telefone, CPF ou nome de cliente de verdade.

## Instalar em outro computador

Coloque esta pasta em `~/.claude/skills/gd-vendas-infoproduto` (no Windows,
`C:\Users\<você>\.claude\skills\gd-vendas-infoproduto`). O Claude Code carrega a skill sozinho.

## Atualizar

Ao corrigir ou descobrir algo na leitura de um webhook em qualquer app, registre aqui (e na lista de
pendências, se o outro app ainda precisar da mesma correção).
