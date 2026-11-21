# Arquitetura do código

Todo o projeto está em um único arquivo, `index.html`, sem dependências além das fontes do Google Fonts. Este documento mostra como o código está organizado e por onde começar para mudar alguma coisa.

[← Voltar ao README](../README.md)

---

## Visão geral do arquivo

```
index.html
├── <head>
│   ├── metadados, título, fontes
│   └── <style>            ← todo o CSS (cores em variáveis, tema claro e escuro)
└── <body>
    ├── <header>           ← título e controles (pausar, velocidade, visão, cérebro novo)
    ├── <main class="layout">
    │   ├── <section class="stage">   ← canvas da floresta + legenda
    │   └── <aside class="mind">      ← abas Pensamentos / Cérebro / Progresso
    ├── <section class="board">       ← estatísticas, gráficos e diário
    └── <script>           ← toda a lógica, dentro de uma IIFE com 'use strict'
```

O JavaScript fica dentro de `(() => { ... })()`, então nada vaza para o escopo global.

---

## Seções do `<script>`

O script é dividido por comentários `// ---------- nome ----------`, nesta ordem:

| Seção | Responsabilidade |
|---|---|
| **constants** | Todas as constantes do jogo e do treino. Veja [parametros.md](parametros.md). |
| **neural network** | Rede neural: criação, *forward*, cópia e otimizador Adam. |
| **replay memory** | Memória de experiências e o passo de treino (`trainBatch`). |
| **world** | Estado do mundo, geração de árvores e nozes, esquilo, raposa e a função `envTick`. |
| **senses** | Raios de visão e sentidos em volta (`observe`). |
| **thoughts** | Fluxo de pensamentos (`think`) e geração de frases (`reasonText`). |
| **agent loop** | Escolha de ação, gravação de experiências e fim de vida (`simTick`, `endLife`). |
| **persistence** | Salvar e carregar o cérebro no `localStorage`. |
| **rendering: world** | Desenho da floresta no canvas. |
| **rendering: panel** | Atualização das abas laterais (barras, feed, radar, neurônios, gráfico). |
| **statistics board** | Indicadores, 7 gráficos, diário e exportação CSV. |
| **controls** | Botões, abas, cliques no mapa e eventos da página. |
| **boot** | Inicialização e o laço `requestAnimationFrame`. |

---

## Fluxo de um quadro de tela

```
requestAnimationFrame(frame)
│
├── se rodando:
│     ├── velocidade 1/3/10 → chama simTick() 1, 3 ou 10 vezes
│     └── Turbo → chama simTick() em blocos de 20 até gastar ~13 ms
│
├── render(t)                    ← desenha a floresta
├── a cada 110 ms: updatePanel() ← aba lateral ativa
├── a cada 1 s:   updateStats()  ← indicadores, gráficos (se mudaram) e diário
└── a cada 20 s:  save()         ← cérebro no localStorage
```

A simulação e o desenho são independentes: no Turbo a tela continua sendo desenhada umas 60 vezes por segundo, mas centenas ou milhares de ticks rodam entre um quadro e outro.

---

## Fluxo de uma decisão (`simTick`)

Cada decisão ocupa 2 ticks. A variável `phase` diz em qual tick estamos.

```
phase 0 (início da decisão)
│  observe(obs)               ← 62 sentidos (se ainda não tiver)
│  forward(online, obs)       ← 6 valores Q
│  ε-greedy                   ← escolhe curAct
│  estatísticas da vida       ← contagem de ações, curiosidade, fome média
│  talvez gerar pensamento    ← reasonText()
│
├─ envTick(curAct, primeiro=true)   ← move, come, enterra, raposa, fome...
phase 1
├─ envTick(curAct, primeiro=false)
│
fim da decisão (ou morte no meio)
│  observe(obs2)
│  remember(obs, curAct, recompensa, obs2, acabou)
│  decisions++ ; a cada 2000 → ponto no gráfico de treino
│  a cada 2 decisões (com 1000+ lembranças) → trainBatch()
└─ se acabou → endLife(causa)  senão obs ← obs2
```

---

## Funções principais

### Rede neural

| Função | O que faz |
|---|---|
| `makeNet()` | Cria uma rede `NIN → HID... → NOUT` com pesos, vieses, gradientes e momentos do Adam em `Float32Array`. |
| `forward(net, x, acts)` | Calcula a saída. Grava a ativação de cada camada em `acts` (reaproveitado no *backprop*). |
| `copyNet(src, dst)` | Copia pesos (usado para a rede-alvo). |
| `adam(net, n)` | Aplica um passo do Adam usando os gradientes acumulados, divididos por `n`. |
| `initBrain(saved)` | Cria a rede principal (opcionalmente com pesos salvos), a rede-alvo e os buffers. |

### Treino

| Função | O que faz |
|---|---|
| `remember(s, a, r, s2, d)` | Grava uma experiência na memória circular `RB`. |
| `trainBatch()` | Sorteia `BATCH` experiências, calcula alvos (Double DQN), faz *backprop* com erro limitado (Huber) e chama `adam`. Sincroniza a rede-alvo a cada `SYNC` passos. |
| `epsilon()` | Curiosidade atual, que cai linearmente até `EPS_MIN`. |

### Mundo

| Função | O que faz |
|---|---|
| `genTrees()` | Gera 12 árvores com distância mínima entre elas. |
| `spawnNut(x?, y?)` | Cria uma noz perto de uma árvore aleatória, ou numa posição dada (clique). |
| `newSquirrel()` | Cria o esquilo e zera as estatísticas da vida. |
| `resetFox()` | Recria a raposa longe do esquilo, com tempo de carência. |
| `envTick(act, first)` | **Coração da simulação.** Avança 1 tick: estações, nozes, ação, árvore, coleta, esconderijos, raposa e fome. Retorna `{ r, done, cause? }`. |
| `steer(o, tx, ty, sp, maxTurn)` | Gira um objeto em direção a um ponto, com limite de giro, e anda. Usado pela raposa. |

### Sentidos

| Função | O que faz |
|---|---|
| `observe(out)` | Preenche as 62 entradas. Também atualiza `rayVis` (para desenhar os raios), `percept` (melhor percepção por tipo) e `around` (sentidos em volta), usados pelos pensamentos. |
| `rayCircle(...)` | Interseção analítica entre um raio e um círculo. |
| `sense(out, i, alvo, alcance, slot)` | Escreve `[sen, cos, proximidade]` de um alvo. |

### Pensamentos e estatísticas

| Função | O que faz |
|---|---|
| `think(texto, tag, prio, extra)` | Adiciona ao fluxo de pensamentos. `prio`: 0 = raciocínio, 1 = acontecimento, 2 = importante. Acontecimentos (`prio ≥ 1`) também vão para o diário via `logEvent`. |
| `reasonText(ação, explorou)` | Monta a frase de raciocínio a partir dos sentidos e da ação. |
| `endLife(causa)` | Grava a vida no `history`, gera pensamentos de morte e recorde e renasce o esquilo. |
| `logEvent(tag, texto)` | Grava no diário (`logbook`, até 1.500 itens). |
| `updateStats()` | Atualiza indicadores, redesenha os gráficos se `statsDirty` e o diário se `logDirty`. |
| `plot(...)` / `strokeLine(...)` | Ajudantes de gráfico: eixos com escala "redonda", grade e linhas. |
| `livesCSV()` / `logCSV()` | Geram o CSV para copiar. |

---

## Estado global (variáveis importantes)

| Variável | Conteúdo |
|---|---|
| `online`, `target` | Rede principal e rede-alvo. |
| `RB` | Memória de experiências (`Float32Array` contíguos). |
| `trees`, `nuts`, `caches` | Objetos do mundo. |
| `sq` | O esquilo: posição, ângulo, fome, nozes e estatísticas da vida atual. |
| `fox` | A raposa: posição, modo (`wander` / `chase` / `rest`) e temporizadores. |
| `season`, `seasonT` | Estação e ticks desde que ela começou. |
| `life`, `lifeT`, `record` | Número da vida, ticks da vida atual e recorde. |
| `decisions`, `trainSteps`, `lossAvg` | Contadores do treino. |
| `history` | Últimas 400 vidas, com todas as estatísticas. |
| `series` | Pontos do gráfico de treino (1 a cada 2.000 decisões, até 400). |
| `logbook` | Diário de eventos (até 1.500). |
| `feed` | Pensamentos mostrados na aba (até 70). |
| `speed`, `running` | Velocidade (0 = Turbo) e pausa. |

---

## Desempenho

- Todos os vetores da rede e da memória são `Float32Array`, alocados uma vez só. O treino não cria objetos novos por passo.
- `forward` e o *backprop* são laços simples sobre arrays, que o motor JavaScript otimiza bem.
- O fundo da floresta é desenhado uma vez em dois canvases fora da tela (outono e inverno) e depois apenas copiado.
- Os gráficos de estatística só são redesenhados quando algo muda (`statsDirty`), no máximo uma vez por segundo.

---

## Temas e estilo

- As cores ficam em variáveis CSS no `:root` (`--bg`, `--ink`, `--accent` etc.).
- O tema escuro é aplicado por `prefers-color-scheme: dark` ou pelo atributo `data-theme="dark"` na tag `<html>`.
- Os canvases do painel leem as cores do tema com `getComputedStyle`, então os gráficos também mudam de tema.
- A floresta tem cores próprias (verde e neve), iguais nos dois temas.
- Fontes: **Bricolage Grotesque** (títulos), **Atkinson Hyperlegible** (texto) e **JetBrains Mono** (números).

---

## Como adicionar coisas novas

### Uma nova ação (ex.: "pular")

1. Adicione o nome em `ACTIONS` e aumente `NOUT`.
2. Trate o novo número em `envTick` (bloco `// actions`).
3. Adicione uma cor em `ACT_COLORS` e um rótulo curto no array `short` de `drawNeurons`.
4. Troque `KEY` para invalidar cérebros salvos.

### Um novo sentido (ex.: "frio")

1. Aumente `NIN`.
2. Escreva o valor em `observe` nas novas posições.
3. Ajuste o texto de quantidade de entradas na aba Cérebro.
4. Troque `KEY`.

### Um novo tipo de recompensa

Some em `r` dentro de `envTick`. Se quiser que apareça no diário, chame `think(texto, tag, 1)`.

### Um novo gráfico

1. Copie um `<figure class="card" data-chart="...">` no HTML.
2. Crie uma função `chartAlgo()` usando `plot()` e `strokeLine()`.
3. Chame-a dentro de `updateStats()`.
4. Para ter tooltip, defina `P.o.meta = { padL, bw, n, tip: i => '...' }`.

[← Voltar ao README](../README.md)
