# Skill: Animações para a web

Uma skill para o [Claude Code](https://claude.com/claude-code) que ensina o Claude a criar animações e microinterações com acabamento de produto premium: rápidas, com propósito, leves no celular e acessíveis.

Ela entra sozinha quando você pede algo como "anima esse botão", "faz o menu abrir suave", "coloca efeito ao rolar a página" ou "o site está sem vida". Também dá para chamar direto com `/animacoes-web`.

## O que tem dentro

- **Quando animar (e quando não animar):** propósito, frequência e expectativa de quem usa.
- **Números de referência:** durações por tipo de interação, curvas de movimento e molas.
- **Desempenho e acessibilidade:** o que pode ser animado sem travar e como respeitar quem pede menos movimento.
- **12 receitas, em CSS puro e em React (Motion):** botão que afunda ao clicar, troca de texto no botão, altura que acompanha o conteúdo, indicador que desliza entre abas, cartão que vira modal, passos com direção, modal e gaveta, popover, lista que adiciona e remove itens, imagem com blur-in, entrada ao rolar a página e toast.
- **O que evitar** e um **checklist** antes de entregar.

O conteúdo completo está em [`skills/animacoes-web/SKILL.md`](skills/animacoes-web/SKILL.md).

## Como instalar

### Opção 1: como plugin (recomendado)

No Claude Code, digite:

```
/plugin marketplace add annevoitk/skill-animacoes-web
/plugin install animacoes-web@animacoes-web
```

Depois rode `/reload-plugins` (ou reinicie o Claude Code).

### Opção 2: copiando o arquivo

1. Baixe o arquivo [`skills/animacoes-web/SKILL.md`](skills/animacoes-web/SKILL.md).
2. Crie a pasta `~/.claude/skills/animacoes-web/` (no Windows: `C:\Users\SEU_USUARIO\.claude\skills\animacoes-web\`).
3. Coloque o `SKILL.md` dentro dela.
4. Reinicie o Claude Code.

Para usar só em um projeto, coloque a pasta em `.claude/skills/animacoes-web/` dentro do projeto.

## Como testar

Abra um projeto e peça: *"adiciona uma animação no botão de enviar do formulário"*. O Claude deve usar a skill, escolher uma duração curta e respeitar `prefers-reduced-motion`.

---

Criado por **Anne Lise Rosales** · [Flume](https://studioflume.com)
