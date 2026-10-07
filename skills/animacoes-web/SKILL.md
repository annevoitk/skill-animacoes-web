---
name: animacoes-web
description: Animações e microinterações para a web com acabamento de produto premium. Use sempre que o pedido envolver animação, transição, motion, microinteração, efeito de hover ou clique, entrada de elementos ao rolar a página, modal, gaveta (drawer), popover, abas, toast, troca de conteúdo, cartão que expande, lista que adiciona ou remove itens, ou quando a pessoa disser que a interface "está dura", "sem vida", "travada" ou pedir para "dar mais vida". Funciona em HTML/CSS/JS puro e em React (Motion, antigo Framer Motion).
---

# Animações para a web

Animação boa é aquela que a pessoa sente, mas não percebe. Ela explica o que aconteceu (de onde veio, para onde foi), dá resposta imediata ao toque e some do caminho. Animação ruim chama atenção para si, atrasa a tarefa e cansa na décima vez.

## 1. Antes de animar, pergunte

1. **Qual é o propósito?** Só anime se for para: dar feedback a uma ação, mostrar mudança de estado, manter a orientação espacial (de onde o elemento veio) ou criar um momento de marca que acontece poucas vezes. "Fica bonito" sozinho não é motivo.
2. **Com que frequência a pessoa vê isso?** Quanto mais frequente, mais curta e discreta. Ações repetidas centenas de vezes por dia (atalho de teclado, abrir menu de comando, digitar) **não devem ter animação**.
3. **A pessoa está esperando por isso?** Se a animação fica entre a pessoa e o que ela quer, ela tem que ser rápida o bastante para não ser notada como espera.

## 2. Números que funcionam

| Situação | Duração | Curva |
|---|---|---|
| Hover, foco, troca de cor | 150–200 ms | `ease` ou `ease-out` |
| Pressionar botão (escala) | 100–160 ms | `ease-out` |
| Elemento entrando (dropdown, popover, toast) | 180–250 ms | `ease-out` |
| Elemento saindo | 120–200 ms (mais rápido que a entrada) | `ease-in` ou `ease-out` |
| Modal, gaveta, mudança de layout | 250–400 ms | mola sem quique |
| Elemento grande atravessando a tela | 400–500 ms | `ease-in-out` ou mola |
| Momento de marca (hero, onboarding) | até ~800 ms | livre |

**Regra geral: quase nada passa de 300 ms.** Se parece lento, provavelmente está.

### Curvas (cubic-bezier)

As curvas padrão do navegador são fracas. Prefira estas:

```css
:root {
  --ease-out: cubic-bezier(0.23, 1, 0.32, 1);       /* entrar: começa rápido, assenta suave */
  --ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);   /* mover algo que já está na tela */
  --ease-in: cubic-bezier(0.55, 0, 1, 0.45);         /* sair de cena */
}
```

- **Entrando ou respondendo a um toque: `ease-out`.** O movimento começa rápido, o que dá sensação de resposta imediata.
- **Movendo algo que já está visível de um ponto a outro: `ease-in-out`.**
- **Nunca `linear`** em movimento de interface (só em coisas contínuas, como spinner ou barra de progresso).
- **Evite `ease-in` em entradas.** Começar devagar parece atraso.

### Molas (spring)

Molas soam naturais porque não têm duração fixa: respondem à velocidade. Para interface, use **mola sem quique** (bounce 0). Quique só em momentos lúdicos e raros.

```js
// Motion (React)
transition={{ type: "spring", duration: 0.3, bounce: 0 }}   // padrão para quase tudo
transition={{ type: "spring", duration: 0.45, bounce: 0.15 }} // algo mais expressivo, uso raro
```

## 3. Desempenho (não negociável)

- **Anime só `transform` e `opacity`.** Eles rodam na placa de vídeo e não recalculam o layout. `filter: blur()` também é aceitável em elementos pequenos.
- **Nunca anime** `width`, `height`, `top`, `left`, `margin`, `padding`. Para mudar tamanho, use `scale` ou a técnica de medir a altura (receita 4.3) ou o `layout` do Motion.
- `will-change: transform` só em elementos que realmente vão animar, e só durante a animação.
- Animações em loop infinito: pause quando estão fora da tela (`IntersectionObserver`).
- Teste em celular comum, não só no computador.

## 4. Acessibilidade

- **Sempre respeite `prefers-reduced-motion`.** Para quem pede menos movimento, troque deslocamento e escala por um simples fade curto, ou corte a animação.

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

```js
// Motion (React): desliga movimentos e mantém fades
import { MotionConfig } from "motion/react";
<MotionConfig reducedMotion="user">{app}</MotionConfig>
```

- Nada pisca mais de 3 vezes por segundo.
- Animação nunca pode ser o único jeito de entender uma informação.
- Elementos interativos continuam clicáveis durante a animação (não bloqueie a interface esperando a animação terminar).

## 5. Receitas

Cada receita tem a versão em CSS/JS puro e em React com Motion. Em projeto React, instale `motion` (`npm i motion`) e importe de `motion/react`. Em site estático, use as versões em CSS.

### 5.1 Botão que responde ao toque

O botão afunda levemente ao ser pressionado. É a microinteração com melhor custo-benefício que existe.

```css
.btn { transition: transform 160ms var(--ease-out); }
.btn:active { transform: scale(0.97); }
```

```jsx
<motion.button whileTap={{ scale: 0.97 }} transition={{ duration: 0.12 }}>Salvar</motion.button>
```

Não passe de `0.95`. Não anime a escala no hover em botões de formulário (só em cartões e imagens, e no máximo `1.02`–`1.04`).

### 5.2 Troca de texto ou ícone dentro de um botão

Ex.: "Enviar" → spinner → "Enviado ✓". O conteúdo antigo sai para cima e o novo entra de baixo, ao mesmo tempo.

```jsx
import { AnimatePresence, motion } from "motion/react";

const variants = {
  initial: { opacity: 0, y: -20 },
  visible: { opacity: 1, y: 0 },
  exit: { opacity: 0, y: 20 },
};

<button className="relative overflow-hidden">
  <AnimatePresence mode="popLayout" initial={false}>
    <motion.span
      key={estado}               /* trocar a key é o que dispara a animação */
      variants={variants}
      initial="initial" animate="visible" exit="exit"
      transition={{ type: "spring", duration: 0.3, bounce: 0 }}
    >
      {rotulos[estado]}
    </motion.span>
  </AnimatePresence>
</button>
```

- `mode="popLayout"` tira o elemento que está saindo do fluxo da página, para os dois animarem juntos sem o botão pular de tamanho.
- `initial={false}` evita animar na primeira vez que a tela carrega.
- O botão precisa de `overflow: hidden` e largura estável (ou `min-width`).

### 5.3 Altura que acompanha o conteúdo (gaveta, acordeão, cartão que muda)

Altura `auto` não anima. Meça o conteúdo e anime a altura do recipiente até esse número.

```jsx
import useMeasure from "react-use-measure";

function CaixaFluida({ children }) {
  const [ref, { height }] = useMeasure();
  return (
    <motion.div animate={{ height: height || "auto" }}
                transition={{ type: "spring", duration: 0.35, bounce: 0 }}
                style={{ overflow: "hidden" }}>
      <div ref={ref}>{children}</div>
    </motion.div>
  );
}
```

Em CSS puro, o jeito moderno é o grid:

```css
.acordeao { display: grid; grid-template-rows: 0fr; transition: grid-template-rows 300ms var(--ease-out); }
.acordeao[open] { grid-template-rows: 1fr; }
.acordeao > div { overflow: hidden; }
```

### 5.4 Indicador que desliza entre abas (layout compartilhado)

O destaque da aba ativa "viaja" até a aba clicada, em vez de sumir de uma e aparecer na outra.

```jsx
{abas.map((aba) => (
  <button key={aba} onClick={() => setAtiva(aba)} className="relative px-3 py-1">
    {ativa === aba && (
      <motion.span layoutId="aba-ativa" className="absolute inset-0 rounded-full bg-white/10"
                   transition={{ type: "spring", duration: 0.35, bounce: 0 }} />
    )}
    <span className="relative">{aba}</span>
  </button>
))}
```

O mesmo `layoutId` em dois lugares diferentes faz o Motion animar de um para o outro. Serve para: abas, item selecionado num menu, ponto do slider, "bolinha" de paginação.

Em CSS puro: posicione um único elemento indicador com `transform: translateX()` e `width` calculados via JS no clique (`getBoundingClientRect`), com `transition: transform 300ms var(--ease-out)`.

### 5.5 Cartão que expande para modal (transição compartilhada)

O cartão cresce até virar o modal, em vez de o modal surgir do nada. Cada parte (imagem, título, descrição, botão) recebe um `layoutId` próprio, igual na versão pequena e na grande.

```jsx
// versão pequena
<motion.div layoutId={`card-${id}`} onClick={() => setAberto(id)}>
  <motion.img layoutId={`img-${id}`} src={img} />
  <motion.h3 layoutId={`titulo-${id}`}>{titulo}</motion.h3>
</motion.div>

// versão grande (dentro de AnimatePresence)
{aberto === id && (
  <motion.div layoutId={`card-${id}`} className="modal">
    <motion.img layoutId={`img-${id}`} src={img} />
    <motion.h3 layoutId={`titulo-${id}`}>{titulo}</motion.h3>
    <motion.p initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }}
              transition={{ duration: 0.1 }}>{texto}</motion.p>
  </motion.div>
)}
```

- Conteúdo que só existe na versão grande entra com fade curto, sem `layoutId`.
- Use `layoutId` únicos por item (com o id), senão itens diferentes se confundem.
- Feche com Esc e com clique fora (receita 5.7).

### 5.6 Passos que deslizam conforme a direção (wizard, carrossel, onboarding)

Avançar desliza para a esquerda, voltar desliza para a direita. A direção é passada para as variantes.

```jsx
const variants = {
  initial: (dir) => ({ x: `${110 * dir}%`, opacity: 0 }),
  visible: { x: "0%", opacity: 1 },
  exit: (dir) => ({ x: `${-110 * dir}%`, opacity: 0 }),
};

<MotionConfig transition={{ type: "spring", duration: 0.4, bounce: 0 }}>
  <div style={{ overflow: "hidden" }}>
    <AnimatePresence mode="popLayout" custom={direcao} initial={false}>
      <motion.div key={passo} custom={direcao} variants={variants}
                  initial="initial" animate="visible" exit="exit">
        {conteudo[passo]}
      </motion.div>
    </AnimatePresence>
  </div>
</MotionConfig>
```

`direcao` é `1` ao avançar e `-1` ao voltar. `custom` precisa ir tanto no `AnimatePresence` quanto no filho, para a saída usar a direção atual.

### 5.7 Modal ou gaveta com fundo escurecido

```jsx
<AnimatePresence>
  {aberto && (
    <>
      <motion.div className="fundo" onClick={fechar}
                  initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }}
                  transition={{ duration: 0.2 }} />
      <motion.div role="dialog" aria-modal="true" className="gaveta"
                  initial={{ y: "100%" }} animate={{ y: 0 }} exit={{ y: "100%" }}
                  transition={{ type: "spring", duration: 0.4, bounce: 0 }}>
        {conteudo}
      </motion.div>
    </>
  )}
</AnimatePresence>
```

- O fundo usa só opacidade e duração curta. A gaveta usa mola.
- Modal no centro: entre com `scale: 0.96 → 1` + opacidade, nunca de `scale: 0`.
- Feche com Esc, com clique no fundo e devolva o foco ao botão que abriu.
- Em CSS puro, use `<dialog>` com `@starting-style` e `transition-behavior: allow-discrete` para animar a abertura e o fechamento.

### 5.8 Popover que nasce do botão

O popover deve crescer a partir do ponto de onde saiu, não do centro.

```css
.popover {
  transform-origin: var(--origem, top center);
  transition: transform 180ms var(--ease-out), opacity 180ms var(--ease-out);
}
.popover:not(.aberto) { opacity: 0; transform: scale(0.96); pointer-events: none; }
```

Em React, outra opção elegante: o botão e o popover compartilham um `layoutId` no recipiente, e o rótulo do botão (com seu próprio `layoutId`) vira o título do popover.

### 5.9 Itens entrando e saindo de uma lista

```jsx
<AnimatePresence initial={false}>
  {itens.map((item) => (
    <motion.li key={item.id} layout
               initial={{ opacity: 0, scale: 0.9 }} animate={{ opacity: 1, scale: 1 }}
               exit={{ opacity: 0, scale: 0.9, transition: { duration: 0.15 } }}
               transition={{ type: "spring", duration: 0.3, bounce: 0 }}>
      {item.nome}
    </motion.li>
  ))}
</AnimatePresence>
```

O `layout` faz os outros itens deslizarem para ocupar o espaço, em vez de pularem. A saída é mais rápida que a entrada.

### 5.10 Imagem que aparece com foco (blur-in)

Bom para galerias e imagens que carregam: começa desfocada, levemente maior e transparente, e assenta.

```css
.revela { animation: revela 500ms var(--ease-out) both; }
@keyframes revela {
  from { opacity: 0; filter: blur(10px); transform: scale(1.08); }
  to   { opacity: 1; filter: blur(0); transform: scale(1); }
}
```

```jsx
<motion.img initial={{ opacity: 0, filter: "blur(10px)", scale: 1.08 }}
            animate={{ opacity: 1, filter: "blur(0px)", scale: 1 }}
            transition={{ duration: 0.5, ease: [0.23, 1, 0.32, 1] }} />
```

Use em poucos elementos por vez. Blur em muitos elementos grandes pesa no celular.

### 5.11 Entrada ao rolar a página (site estático)

```css
.reveal { opacity: 0; transform: translateY(16px); transition: opacity 600ms var(--ease-out), transform 600ms var(--ease-out); }
.reveal.visivel { opacity: 1; transform: none; }
.reveal.d1 { transition-delay: 80ms; } .reveal.d2 { transition-delay: 160ms; }
```

```js
const io = new IntersectionObserver((entradas) => {
  for (const e of entradas) if (e.isIntersecting) { e.target.classList.add("visivel"); io.unobserve(e.target); }
}, { rootMargin: "0px 0px -10% 0px" });
document.querySelectorAll(".reveal").forEach((el) => io.observe(el));
```

- Anima **uma vez só** (`unobserve`). Repetir ao subir e descer cansa.
- Deslocamento pequeno (12–24 px). Nada vindo de 100 px de distância.
- Escalonamento (stagger) de 50–80 ms entre itens, no máximo 5–6 itens. Depois disso, entra tudo junto.
- **Nunca esconda conteúdo importante atrás de animação**: se o JS falhar, o conteúdo tem que aparecer. Adicione a classe que esconde via JS (`document.documentElement.classList.add("js")` e `.js .reveal { opacity: 0 }`).

### 5.12 Toast / notificação

Entra de baixo (ou do canto) com `y: 100% → 0` + opacidade em mola de ~0.35 s, sai mais rápido com opacidade. Vários toasts empilham com `layout` para os de cima se reacomodarem. Pausa o tempo de fechar quando o mouse está em cima.

## 6. O que evitar

- Animar tudo. Escolha poucos momentos e faça-os bem.
- Durações longas "para ficar suave". Suave vem da curva, não da duração.
- Entradas de `scale: 0`. Comece de `0.9`–`0.96`.
- Quique (bounce) em interface séria: formulários, painéis, finanças, saúde.
- Animação que bloqueia: a pessoa não consegue clicar até terminar.
- Parallax pesado e scroll sequestrado (`scroll-jacking`).
- Spinners para esperas menores que ~300 ms (pisca e some). Mostre só depois desse tempo.
- Mudar o layout da página enquanto a pessoa lê (conteúdo que empurra outro conteúdo).
- Animar ao carregar a página elementos que estão abaixo da dobra.

## 7. Checklist antes de entregar

- [ ] Cada animação tem um propósito claro (feedback, estado, orientação ou marca).
- [ ] Ações frequentes são rápidas (≤ 200 ms) ou não animam.
- [ ] Só `transform`, `opacity` (e `filter` com moderação) estão sendo animados.
- [ ] Entradas usam `ease-out` ou mola sem quique. Nada `linear` em movimento.
- [ ] Saídas são mais rápidas que entradas.
- [ ] `prefers-reduced-motion` respeitado.
- [ ] Interface continua utilizável durante a animação e sem JavaScript.
- [ ] Testado no celular e com rolagem rápida.
- [ ] Modais e gavetas fecham com Esc e clique fora, e devolvem o foco.

## 8. Ao aplicar num projeto existente

1. Leia o CSS/JS do projeto e **reaproveite** as curvas, durações e variáveis que já existem. Não crie um segundo sistema de animação.
2. Se o projeto é estático (sem React), use as versões em CSS/JS puro. Não instale Motion só para isso.
3. Se o projeto já usa outra biblioteca de animação (GSAP, Motion, Anime.js), siga com ela.
4. Centralize durações e curvas em variáveis (`--ease-out`, `--dur-rapida`) para ajustar tudo de uma vez.
5. Ao terminar, descreva para a pessoa, em linguagem simples, o que passou a se mover e por quê.
