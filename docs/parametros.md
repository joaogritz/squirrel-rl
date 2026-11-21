# Parâmetros

Todas as regras ajustáveis ficam como constantes no começo do `<script>` do `index.html`. Esta página lista cada uma, o valor atual e o que acontece quando você muda.

[← Voltar ao README](../README.md)

> **Importante:** ao mudar `NIN`, `NOUT` ou `HID` (o formato da rede), troque também `KEY` (por exemplo para `'avela-cerebro-v3'`). O carregamento já recusa cérebros com formato diferente, mas trocar a chave deixa claro que é um cérebro novo e evita misturar históricos.

---

## Mundo

| Constante | Valor | O que é | Aumentar | Diminuir |
|---|---|---|---|---|
| `W`, `H` | 900, 600 | Tamanho do mapa | Mais espaço; comida mais espalhada | Mais apertado; raposa mais perto |
| `AUTUMN` | 2400 | Duração do outono em ticks (~40s em 1×) | Mais tempo de fartura | Menos tempo para estocar |
| `WINTER` | 1200 | Duração do inverno em ticks (~20s em 1×) | Inverno mais duro; guardar comida fica essencial | Inverno quase irrelevante |

Valores fixos dentro do código (procure por eles se quiser mudar):

| Onde | Valor | O que é |
|---|---|---|
| `genTrees` | 12 árvores, distância mínima 125, raio 30–40 | Floresta |
| `envTick` | `nuts.length < 16`, chance `0.025` por tick | Limite e ritmo de nozes no outono |
| boot | 12 nozes | Nozes no começo |
| `t.r * 0.62` | 62% do raio | Área "em cima da árvore" |
| `tick - c.born > 200` | 200 ticks | Tempo até um esconderijo poder ser desenterrado |

---

## Esquilo

| Constante | Valor | O que é | Aumentar | Diminuir |
|---|---|---|---|---|
| `SPD` | 2.0 | Velocidade por tick | Foge e coleta mais fácil | Mais vulnerável |
| `TURN` | 0.11 | Giro por tick (rad) ao virar | Curvas fechadas | Curvas abertas, difícil mirar |
| `HUNGER_RATE` | 1/1600 | Fome por tick | Morre de fome mais rápido; comida mais valiosa | Fome quase não importa |

Valores fixos: fome inicial `0.3`, comer reduz `0.4`, máximo de `3` nozes na boca e raio de coleta `12`.

---

## Visão

| Constante | Valor | O que é | Aumentar | Diminuir |
|---|---|---|---|---|
| `RAYS` | 9 | Número de raios | Visão mais detalhada, rede maior (5 entradas por raio) | Visão grosseira |
| `FOV` | π × 1,15 (~207°) | Abertura do leque | Enxerga mais dos lados | "Visão de túnel" |
| `RAYLEN` | 180 | Alcance dos raios | Vê de mais longe | Míope |

Os alcances dos sentidos em volta ficam em `observe`: faro `420`, árvore `420`, audição `300`, memória `500`.

> `NIN` é calculado como `RAYS * 5 + 5 + 12`. Se mudar `RAYS`, ele se ajusta sozinho, mas troque `KEY`.

---

## Raposa

| Constante | Valor | O que é | Aumentar | Diminuir |
|---|---|---|---|---|
| `FOX_CHASE` | 1.45 | Velocidade perseguindo | Muito mais mortal (acima de 2,0 ela alcança sempre em linha reta) | Quase inofensiva |
| `FOX_WALK` | 0.9 | Velocidade passeando | Cobre mais área | Fica mais parada |
| `FOX_SIGHT` | 150 | Distância em que nota o esquilo | Persegue mais vezes | Persegue raramente |
| `FOX_STAMINA` | 170 | Ticks correndo antes de cansar | Perseguições longas | Basta correr um pouco |

Valores fixos: captura a menos de `15`, descanso de `160` ticks, pausa de `160` ticks depois que o esquilo sobe na árvore, `240` ticks de carência no começo de cada vida e nascimento a pelo menos `380` do esquilo.

---

## Cérebro e treino

| Constante | Valor | O que é | Aumentar | Diminuir |
|---|---|---|---|---|
| `HID` | `[48, 48]` | Neurônios por camada oculta | Mais capacidade, treino mais lento por passo | Aprende menos coisas |
| `GAMMA` | 0.98 | Desconto do futuro | Planeja mais longe, treino mais instável | Pensa só no imediato |
| `LR` | 5e-4 | Taxa de aprendizado do Adam | Aprende mais rápido, mas pode "desaprender" | Aprende devagar e com estabilidade |
| `BATCH` | 32 | Lembranças por passo de treino | Gradiente mais estável, passo mais caro | Mais ruído |
| `REPLAY` | 30000 | Tamanho da memória | Lembra de mais coisas antigas | Foca no recente, pode esquecer |
| `SYNC` | 800 | Passos de treino entre cópias para a rede-alvo | Mais estável, aprende mais devagar | Mais rápido, pode oscilar |
| `EPS_DECAY` | 40000 | Decisões até a curiosidade chegar ao mínimo | Explora por mais tempo | Para de explorar cedo, pode "travar" num jeito ruim |
| `EPS_MIN` | 0.04 | Curiosidade mínima permanente | Mais ações aleatórias para sempre | Mais "certinho", explora menos |

Valores fixos: o treino começa com `1000` lembranças, roda a cada `2` decisões e cada decisão dura `2` ticks (`phase === 2`).

---

## Recompensas

Ficam dentro de `envTick`. Veja a tabela completa em [como-funciona.md](como-funciona.md#6-recompensas). Os pontos mais sensíveis:

| Recompensa | Valor | Efeito de mudar |
|---|---|---|
| Pegar noz | +0,15 | Mais alto: vira "colecionador" e esquece de comer. Mais baixo: demora a achar comida. |
| Comer | 0,3 + 0,7 × fome | É o que mais motiva a sobrevivência. |
| Enterrar de barriga cheia | +0,06 | Mais alto: enterra tudo compulsivamente. Zero: quase nunca enterra. |
| Morrer | −1 | Mais negativo: fica medroso e prefere ficar na árvore. |
| Cada tick | −0,001 | Mais negativo: fica "apressado". |

> Cuidado com recompensas que podem ser repetidas sem custo. O esquilo **vai** achar o atalho (por isso existe a espera de 200 ticks para desenterrar).

---

## Interface

| Constante ou valor | Onde | O que é |
|---|---|---|
| `KEY` | constants | Chave do `localStorage` (`'avela-cerebro-v2'`) |
| `0.06` | `softmax` | Temperatura das barras de vontade e da certeza |
| `1700` / `2600` ms | `simTick` | Intervalo dos pensamentos de raciocínio em 1× e 3× |
| `20000` ms | `frame` | Intervalo do salvamento automático |
| `13` ms | `frame` | Tempo de simulação por quadro no Turbo |
| `70` | `think` | Pensamentos guardados na aba |
| `400` / `300` | `endLife` / `save` | Vidas na memória / vidas salvas |
| `1500` / `300` | `logEvent` / `save` | Eventos no diário / eventos salvos |

---

## Receitas de experimento

| Experimento | Mudanças |
|---|---|
| **Modo difícil** | `FOX_CHASE = 1.7`, `FOX_SIGHT = 200`, `WINTER = 2000` |
| **Modo zen** (sem raposa) | `FOX_SIGHT = 0` |
| **Esquilo míope** | `RAYLEN = 80` (troque `KEY`) e veja quanto ele depende do faro |
| **Sem faro** | Em `observe`, comente a linha `sense(out, b + 5, nearest(nuts), ...)` e compare o gráfico de nozes por vida |
| **Cérebro minúsculo** | `HID = [8]` (troque `KEY`) e veja o limite do que ele consegue aprender |
| **Aprendiz apressado** | `LR = 3e-3`, `SYNC = 100` e observe a instabilidade no gráfico de duração |

[← Voltar ao README](../README.md)
