# Como funciona

Este documento explica toda a lógica da Floresta da Avelã: o mundo, os sentidos, as ações, as recompensas, o cérebro e como ele aprende. Não é preciso saber IA para acompanhar. Os detalhes técnicos ficam marcados como **(técnico)**.

[← Voltar ao README](../README.md)

---

## Sumário

1. [A ideia geral: aprendizado por reforço](#1-a-ideia-geral-aprendizado-por-reforço)
2. [O mundo](#2-o-mundo)
3. [O esquilo](#3-o-esquilo)
4. [Os sentidos (62 entradas)](#4-os-sentidos-62-entradas)
5. [As ações (6 saídas)](#5-as-ações-6-saídas)
6. [Recompensas](#6-recompensas)
7. [A raposa](#7-a-raposa)
8. [O cérebro: rede neural](#8-o-cérebro-rede-neural)
9. [Como ele aprende: Deep Q-Learning](#9-como-ele-aprende-deep-q-learning)
10. [Curiosidade (exploração)](#10-curiosidade-exploração)
11. [Como os pensamentos são gerados](#11-como-os-pensamentos-são-gerados)
12. [Por que ele demora para aprender certas coisas](#12-por-que-ele-demora-para-aprender-certas-coisas)

---

## 1. A ideia geral: aprendizado por reforço

Aprendizado por reforço (*Reinforcement Learning*) é aprender como um animal de verdade: **tentando, errando e lembrando do que deu certo**.

Ninguém diz ao esquilo o que fazer. Existem só quatro coisas:

| Peça | Na Floresta da Avelã |
|---|---|
| **Agente** | O esquilo, quem decide. |
| **Ambiente** | A floresta, com nozes, árvores, raposa e estações. |
| **Ações** | As 6 coisas que ele pode fazer. |
| **Recompensa** | Um número que diz se a última jogada foi boa ou ruim. |

O objetivo do agente é **maximizar a soma das recompensas ao longo do tempo**, e não só a próxima. Por isso ele aprende que andar até uma noz (que não dá ponto na hora) vale a pena, porque leva a pegar e comer a noz (que dá).

---

## 2. O mundo

- **Tamanho:** 900 × 600 unidades, visto de cima.
- **Tempo:** a simulação anda em *ticks* (passos). Em 1×, roda 1 tick por quadro de tela, ou seja, cerca de 60 ticks por segundo. Os tempos mostrados na página (por exemplo, "vida de 45s") são convertidos considerando 60 ticks = 1 segundo.
- **Árvores:** 12 árvores fixas, geradas aleatoriamente com pelo menos 125 unidades de distância entre si. Cada uma tem raio de 30 a 40. Estar a menos de 62% do raio do centro conta como "em cima da árvore", onde a raposa não alcança.
- **Nozes:** no outono, a cada tick há 2,5% de chance de cair uma noz perto de uma árvore aleatória, até no máximo 16 no chão. A simulação começa com 12.
- **Esconderijos:** criados quando o esquilo enterra uma noz. Enterrar a menos de 16 unidades de um esconderijo existente soma a noz a ele. Um esconderijo só pode ser desenterrado depois de 200 ticks.
- **Estações:**

| Estação | Duração | O que muda |
|---|---|---|
| Outono | 2.400 ticks (~40s em 1×) | Nozes caem das árvores. |
| Inverno | 1.200 ticks (~20s em 1×) | Ao começar, **todas as nozes do chão somem** (a neve cobre). Nenhuma noz nova cai. Só sobrevive quem guardou comida na boca ou nos esconderijos. |

As estações continuam de uma vida para a outra. Uma vida nova pode começar no meio do inverno.

---

## 3. O esquilo

| Atributo | Valor |
|---|---|
| Velocidade andando reto | 2,0 por tick |
| Velocidade virando | 1,7 por tick (85%) |
| Giro por tick ao virar | 0,11 rad (~6,3°) |
| Fome inicial | 30% |
| Fome por tick | 1/1600 (de 0 a 100% em ~27s em 1×) |
| Comer uma noz | reduz a fome em 40 pontos percentuais |
| Nozes na boca | máximo de 3 |
| Pegar noz ou esconderijo | encostar (a menos de 12 unidades) |

**Uma vida termina quando:**
- a fome chega a 100% (**morreu de fome**), ou
- a raposa encosta nele fora de uma árvore (**pego pela raposa**).

Quando uma vida termina, o esquilo renasce num lugar aleatório longe da raposa, com fome de 30%, sem nozes e **sem esconderijos**. O **cérebro continua o mesmo**: é ele que acumula o aprendizado de todas as vidas.

---

## 4. Os sentidos (62 entradas)

Tudo o que o cérebro sabe sobre o mundo chega como 62 números entre −1 e 1.

### 4.1 Visão: 45 números

São 9 raios espalhados num leque de ~207° à frente da cabeça, cada um com alcance de 180 unidades.

Para **cada raio** e **cada tipo de coisa**, a entrada vale:

```
proximidade = 1 − distância / 180     (0 se não vê nada desse tipo no raio)
```

| Tipo | Como é detectado |
|---|---|
| árvore | círculo do "tronco" (62% do raio da copa) |
| noz | círculo de raio 9 |
| esconderijo | círculo de raio 10, só se já pode ser desenterrado |
| raposa | círculo de raio 15 |
| borda | distância até a borda do mapa |

9 raios × 5 tipos = **45 entradas**. Um raio não é bloqueado por objetos: ele informa o objeto mais próximo de *cada tipo* separadamente.

### 4.2 Estado interno: 5 números

| Entrada | Valor |
|---|---|
| fome | 0 a 1 |
| nozes na boca | 0, ⅓, ⅔ ou 1 |
| é inverno? | 0 ou 1 |
| está em cima de árvore? | 0 ou 1 |
| a raposa está me perseguindo? | 0 ou 1 |

### 4.3 Sentidos em volta (360°): 12 números

A visão só cobre a frente. Para ele não ficar "cego" atrás, existem 4 sentidos que apontam para o alvo mais próximo, em qualquer direção:

| Sentido | Alvo | Alcance |
|---|---|---|
| Faro | noz mais próxima | 420 |
| Noção de lugar | árvore mais próxima | 420 |
| Audição | raposa | 300 |
| Memória | esconderijo pronto mais próximo | 500 |

Cada sentido vira 3 números:

```
ângulo_relativo = direção do alvo − direção da cabeça   (de −π a π)
proximidade     = max(0, 1 − distância / alcance)
entradas        = [ sen(ângulo_relativo), cos(ângulo_relativo), proximidade ]
```

O seno diz se o alvo está à esquerda (negativo) ou à direita (positivo). O cosseno diz se está na frente (positivo) ou atrás (negativo). Fora do alcance, os três valem 0.

**Total: 45 + 5 + 12 = 62 entradas.**

> Esses sentidos em volta foram adicionados depois que, nos testes, o esquilo só com visão frontal mal conseguia aprender a fugir da raposa nem a achar comida fora do campo de visão.

---

## 5. As ações (6 saídas)

| # | Ação | Efeito |
|---|---|---|
| 0 | Andar em frente | Anda 2,0 por tick. |
| 1 | Virar à esquerda | Gira 0,11 rad e anda 1,7 por tick. |
| 2 | Virar à direita | Gira 0,11 rad e anda 1,7 por tick. |
| 3 | Enterrar noz | Se tem noz e não está numa árvore, enterra uma no lugar. |
| 4 | Comer noz | Se tem noz na boca, come uma. |
| 5 | Parar e observar | Fica parado. |

**Repetição de ação (técnico):** cada decisão dura **2 ticks**. Enterrar e comer acontecem só no primeiro tick. Isso deixa o aprendizado mais rápido e o movimento mais suave.

---

## 6. Recompensas

A recompensa é somada nos 2 ticks de cada decisão.

| Situação | Recompensa |
|---|---|
| Cada tick vivo | −0,001 (um empurrãozinho para não ficar parado à toa) |
| Pegar uma noz do chão | **+0,15** |
| Comer com fome acima de 20% | **+0,3 + 0,7 × fome** (até +1,0 quando está faminto) |
| Comer sem fome (20% ou menos) | −0,05 |
| Tentar comer sem ter noz | −0,02 |
| Enterrar com fome abaixo de 45% (até 15 esconderijos) | **+0,06** |
| Enterrar com fome alta ou com esconderijos demais | −0,01 |
| Tentar enterrar sem noz ou em cima da árvore | −0,02 |
| Desenterrar um esconderijo no inverno | **+0,2** |
| Desenterrar um esconderijo no outono | +0,05 |
| Chegar numa árvore enquanto é perseguido | **+0,1** |
| Bater na borda do mapa | −0,01 |
| Fome acima de 75% | −0,002 por tick |
| **Morrer (fome ou raposa)** | **−1** e a vida acaba |

### Por que essas regras

- **Comer vale mais quando está com fome.** Isso ensina a guardar comida e comer na hora certa, em vez de comer tudo de cara.
- **Enterrar dá um pouco de ponto só com a barriga cheia.** O benefício de verdade (sobreviver ao inverno) vem muito depois, e o cérebro não enxerga tão longe (veja a [seção 12](#12-por-que-ele-demora-para-aprender-certas-coisas)). Essa recompensa extra (*reward shaping*) dá uma dica.
- **Esconderijos só podem ser desenterrados depois de 200 ticks.** Sem isso, ele aprenderia a "trapacear": enterrar e desenterrar no mesmo lugar sem parar para ganhar pontos.

---

## 7. A raposa

A raposa **não aprende**. Ela segue regras fixas, como uma máquina de estados:

```
           vê o esquilo a menos de 150
           (e o esquilo não está numa árvore)
 ┌─────────┐ ─────────────────────────────▶ ┌──────────┐
 │ PASSEAR │                                │ PERSEGUIR│
 │ vel 0,9 │ ◀───────────────────────────── │ vel 1,45 │
 └─────────┘   esquilo subiu na árvore      └──────────┘
      ▲         (pausa de 160 ticks)              │
      │                                           │ correu 170 ticks
      │        ┌───────────┐                      │
      └─────── │ DESCANSAR │ ◀────────────────────┘
   após 160    │  parada   │
     ticks     └───────────┘
```

- **Passear:** caminha até pontos aleatórios do mapa.
- **Perseguir:** corre direto até o esquilo (com limite de giro, então dá para despistar). Se encostar a menos de 15 unidades, a vida acaba.
- **Desistir:** se o esquilo sobe numa árvore, ela vai passear longe e não volta a perseguir por 160 ticks.
- **Cansar:** depois de 170 ticks correndo, descansa 160 ticks (aparece "z z").
- **Vida nova:** a raposa reaparece a pelo menos 380 unidades do esquilo e não persegue nos primeiros 240 ticks.

O esquilo (2,0) é mais rápido que a raposa (1,45), mas a raposa começa perto e ele perde velocidade ao virar.

---

## 8. O cérebro: rede neural

É uma rede neural *feed-forward* (perceptron multicamadas):

```
entradas (62) ──▶ oculta 1 (48, ReLU) ──▶ oculta 2 (48, ReLU) ──▶ saídas (6, linear)
```

- Cada **saída** é o **valor Q** de uma ação: uma previsão de quantos pontos ele vai somar daqui para frente se fizer essa ação agora e continuar jogando bem depois.
- **ReLU:** `max(0, x)`. É a "ativação" que deixa a rede aprender padrões não lineares.
- **Parâmetros:** 62×48 + 48 + 48×48 + 48 + 48×6 + 6 = **5.622 números** (pesos e vieses). É isso que fica salvo no navegador.

**(técnico)** Inicialização He (`peso ~ N(0, 2/entradas)`). A última camada é multiplicada por 0,3 para começar com valores Q pequenos.

### Escolhendo uma ação

1. Os 62 sentidos entram na rede.
2. Saem 6 valores Q.
3. Com probabilidade *ε* (curiosidade), ele sorteia uma ação. Senão, escolhe a de **maior Q**.

As barras de **Vontade agora** e a **certeza** dos pensamentos são os valores Q passados por uma *softmax* com temperatura 0,06:

```
vontade(a) = exp(Q(a) / 0,06) / Σ exp(Q(b) / 0,06)
```

Isso é só para mostrar na tela. A decisão real usa o maior Q.

---

## 9. Como ele aprende: Deep Q-Learning

O algoritmo é uma versão do **DQN** (*Deep Q-Network*, DeepMind, 2015), com o refinamento **Double DQN**.

### 9.1 A ideia do Q-learning

O valor de uma ação deve ser igual à recompensa imediata **mais** o valor da melhor ação na situação seguinte, com um pequeno desconto:

```
Q(s, a)  ≈  r  +  γ · max Q(s', a')
```

- `s`: o que ele sentia
- `a`: a ação que fez
- `r`: a recompensa que recebeu
- `s'`: o que passou a sentir
- `γ = 0,98`: o desconto. Pontos no futuro valem um pouco menos do que pontos agora.

Se a vida acabou, não existe futuro e o alvo é só `r`.

### 9.2 Memória de experiências (*replay buffer*)

Cada decisão vira uma lembrança `(s, a, r, s', acabou?)`, guardada numa memória circular de **30.000** experiências. Treinar com lembranças sorteadas, em vez de só a última, evita que ele "esqueça" o que aprendeu e deixa o treino estável.

### 9.3 O passo de treino

O treino começa depois de **1.000** lembranças e roda **a cada 2 decisões**:

1. Sorteia **32** lembranças.
2. Para cada uma, calcula o **alvo**:
   - **Double DQN:** a rede principal escolhe a melhor ação seguinte `a* = argmax Q_principal(s', ·)`.
   - A **rede-alvo** (uma cópia congelada) avalia essa ação: `alvo = r + γ · Q_alvo(s', a*)`.
3. Calcula o erro `erro = Q_principal(s, a) − alvo`.
4. Limita o erro entre −1 e +1 (equivalente à **perda de Huber**), para uma experiência muito surpreendente não bagunçar a rede.
5. Faz *backpropagation* só pela saída da ação `a`.
6. Atualiza os pesos com **Adam** (taxa de aprendizado 0,0005).
7. A cada **800** passos de treino, copia a rede principal para a rede-alvo.

### 9.4 Por que duas redes

Se a mesma rede calculasse a previsão e o alvo, ela ficaria "correndo atrás do próprio rabo" e o treino oscilaria. A rede-alvo muda só de vez em quando e dá um alvo estável. O Double DQN ainda evita que ele superestime o valor das ações.

### 9.5 O que o gráfico "erro do cérebro" mostra

É uma média móvel do erro² dos lotes de treino. Ele **não** precisa cair até zero: quando o esquilo aprende algo novo e passa a viver situações novas, o erro pode subir de novo. É a "surpresa" do cérebro.

---

## 10. Curiosidade (exploração)

Se ele sempre fizesse o que acha melhor desde o começo, nunca descobriria coisas novas. Por isso existe o **ε-greedy**:

```
ε = max(0,04 ; 1 − decisões / 40.000 × 0,96)
```

- Nas primeiras decisões, **100%** das ações são sorteadas.
- A curiosidade cai em linha reta até **4%** em 40.000 decisões (uns 22 minutos em 1×, ou bem menos no Turbo).
- Depois disso fica em 4% para sempre.

Na aba Pensamentos, ações sorteadas aparecem como **curiosidade / ação aleatória**.

---

## 11. Como os pensamentos são gerados

Os pensamentos **não** vêm de um modelo de linguagem. São frases montadas a partir do estado real do cérebro e dos sentidos.

### 11.1 Pensamentos de raciocínio

Em 1× (a cada ~1,7s) e em 3× (a cada ~2,6s), no momento da decisão, a função `reasonText()` olha nesta ordem de prioridade:

1. Foi uma ação sorteada? → frase de curiosidade
2. A raposa está perseguindo? → "Corre pra árvore!" (com a direção da árvore)
3. Está na árvore com a raposa perto? → "Aqui em cima ela não me alcança"
4. Escolheu comer ou enterrar? → explica o porquê (fome, inverno chegando)
5. Vê a raposa? → "Melhor ficar de olho"
6. Inverno e vê um esconderijo? → "Hora de desenterrar"
7. Vê uma noz? → "Vou buscar" (com fome) ou "Vou guardar"
8. Lembra de um esconderijo, sente cheiro de noz ou ouve a raposa
9. Situações gerais: boca cheia, borda do mapa, virando, parado

A direção ("à esquerda", "bem na frente", "atrás de mim") vem do raio que viu a coisa ou do ângulo do sentido em volta.

Se a certeza da ação escolhida for menor que 30%, a frase começa com *"Hmm, não tenho certeza..."*.

### 11.2 Pensamentos de acontecimento

Disparados quando algo acontece de fato: pegou noz, comeu, enterrou, desenterrou, a raposa viu, fugiu, morreu, recorde, mudança de estação.

Para não lotar a tela, a aba Pensamentos limita a frequência conforme a velocidade:

| Velocidade | Raciocínio | Acontecimentos | Mortes, recordes, estações |
|---|---|---|---|
| 1× | a cada 1,7s | no máximo 1 a cada 0,2s | 1 a cada 0,15s |
| 3× | a cada 2,6s | 1 a cada 0,2s | 1 a cada 0,15s |
| 10× | não mostra | 1 a cada 0,7s | 1 a cada 0,15s |
| Turbo | não mostra | 1 a cada 1,6s | 1 a cada 0,6s |

O **diário de eventos** não tem esse limite e registra todos os acontecimentos.

---

## 12. Por que ele demora para aprender certas coisas

| Habilidade | Dificuldade | Por quê |
|---|---|---|
| Andar até nozes | Fácil | A recompensa (+0,15) vem logo depois da ação. |
| Comer quando está com fome | Média | Precisa ligar "fome alta" com "comer dá mais ponto". |
| Fugir da raposa | Difícil | Tem que ouvir, achar a árvore, correr para o lado certo e não ser alcançado. Errar custa a vida inteira. |
| Guardar comida para o inverno | Muito difícil | O benefício aparece centenas de passos depois. Com γ = 0,98, uma recompensa a 200 decisões de distância vale só 0,98²⁰⁰ ≈ 1,8% do valor. |

É por isso que existem as recompensas de "dica" (enterrar de barriga cheia, chegar na árvore fugindo). Sem elas, essas habilidades quase nunca apareceriam.

[← Voltar ao README](../README.md)
