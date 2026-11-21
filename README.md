# 🐿️ Floresta da Avelã

**Um esquilo virtual com um cérebro de rede neural que aprende sozinho a sobreviver numa floresta simulada, e você vê o que ele está pensando em tempo real.**

A Avelã nasce sem saber nada. No começo anda sem rumo, bate nas bordas, ignora as nozes e é pega pela raposa. Ela aprende por tentativa e erro: ganha pontos quando come e perde quando morre. Depois de algumas centenas de vidas, passa a buscar nozes, enterrar comida para o inverno e subir nas árvores quando a raposa aparece.

Tudo roda **no navegador**, num único arquivo HTML, sem servidor, sem instalação e sem bibliotecas de IA. A rede neural e o algoritmo de aprendizado foram escritos do zero em JavaScript puro.

---

## Sumário

- [O que tem no projeto](#o-que-tem-no-projeto)
- [Como rodar](#como-rodar)
- [Como usar](#como-usar)
- [Como funciona (resumo)](#como-funciona-resumo)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Documentação completa](#documentação-completa)
- [Personalizando](#personalizando)
- [Perguntas frequentes](#perguntas-frequentes)
- [Limitações conhecidas](#limitações-conhecidas)
- [Ideias para evoluir](#ideias-para-evoluir)

---

## O que tem no projeto

| Parte | O que faz |
|---|---|
| **Floresta 2D** | Mapa visto de cima com 12 árvores, nozes caindo, esconderijos, uma raposa e estações do ano (outono e inverno). |
| **Esquilo com sentidos** | 9 raios de visão à frente, faro (direção da noz mais próxima), noção de onde fica a árvore mais perto, audição (de onde vem a raposa) e memória dos esconderijos. |
| **Cérebro** | Rede neural de 62 entradas → 48 → 48 → 6 saídas, treinada com *Deep Q-Learning* (DQN). |
| **Aba Pensamentos** | Traduz o que o cérebro está "sentindo" e decidindo em frases, com o nível de certeza. |
| **Aba Cérebro** | Radar do que os olhos captam e os neurônios acendendo em tempo real. |
| **Aba Progresso** | Gráfico rápido da duração de cada vida e números gerais. |
| **Estatísticas e diário** | 6 indicadores, 7 gráficos interativos e um diário com até 1.500 eventos, com filtros e exportação em CSV. |
| **Memória persistente** | O cérebro é salvo no navegador (`localStorage`). Você pode fechar e continuar outro dia. |

---

## Como rodar

### Opção 1: abrir o arquivo (mais fácil)

1. Baixe ou clone o repositório.
2. Dê dois cliques em `index.html`.
3. Pronto, a simulação começa sozinha.

```powershell
git clone https://github.com/SEU-USUARIO/floresta-da-avela.git
```

```powershell
start .\floresta-da-avela\index.html
```

> As fontes vêm do Google Fonts. Sem internet a página funciona do mesmo jeito, só com as fontes padrão do sistema.

### Opção 2: servidor local (opcional)

Não é obrigatório, mas se preferir rodar por `http://`:

```powershell
npx serve .
```

ou, com Python:

```powershell
python -m http.server 8000
```

Depois abra `http://localhost:8000`.

---

## Como usar

### Controles

| Controle | O que faz |
|---|---|
| **Pausar / Continuar** | Congela a simulação (o treino também para). |
| **1× / 3× / 10×** | Velocidade da simulação. Em 1× e 3× aparecem os pensamentos do dia a dia. |
| **Turbo** | Roda o máximo que o computador aguentar. Use para treinar rápido. |
| **Mostrar visão** | Liga ou desliga os raios de visão desenhados no mapa. |
| **Cérebro novo** | Apaga tudo o que ele aprendeu e recomeça do zero. Pede um segundo clique para confirmar. |
| **Clique no mapa** | Joga uma noz no lugar clicado. |

### O que aparece no mapa

- **Raios coloridos:** o que cada raio de visão está tocando.
  🟩 árvore · 🟨 noz · 🟪 esconderijo · 🟥 raposa · ⬜ borda
- **Anel em volta do esquilo:** quanto falta para a fome chegar a 100%. Fica amarelo acima de 50% e vermelho acima de 75%.
- **Nozinhas na boca:** quantas nozes ele está carregando (máximo de 3).
- **Montinho de terra com X:** um esconderijo. Fica mais forte quando já pode ser desenterrado.
- **Linha vermelha tracejada:** a raposa está perseguindo.
- **"z z" sobre a raposa:** ela cansou e está descansando.
- **Canto superior esquerdo:** estação atual e quanto falta para acabar.
- **Canto superior direito:** número da vida e tempo de vida atual.

### Abas do painel

**Pensamentos**
- *Fome*, *Na boca*, *Esconderijos*, *Estação* e o estado atual (decidindo, explorando, fugindo, em cima da árvore).
- *Vontade agora*: quanto ele quer fazer cada uma das 6 ações neste instante.
- *Pensamentos*: o fluxo de frases. Cada uma mostra a vida, o segundo e a **certeza**. Quando a ação foi sorteada para explorar, aparece *ação aleatória*.

**Cérebro**
- *O que os olhos captam*: os 9 raios vistos de cima.
- *Neurônios acendendo*: as 62 entradas, as duas camadas ocultas e as 6 ações. Quanto mais forte a cor, mais ativo o neurônio.

**Progresso**
- Gráfico de barras com a duração das últimas 80 vidas e números gerais.

### Estatísticas e diário (abaixo da floresta)

- **Indicadores:** vidas vividas, recorde, média recente com tendência ▲/▼, nozes comidas por vida, taxa de fuga da raposa e principal causa de morte.
- **Gráficos** (passe o mouse para ver detalhes):
  1. Duração de cada vida
  2. Do que ele morre
  3. Nozes por vida
  4. Pontuação por vida
  5. Taxa de fuga da raposa
  6. Curiosidade e erro do cérebro
  7. O que ele faz: antes e agora
- **Diário de eventos:** tabela com filtros por tipo.
- **Copiar vidas (CSV)** e **Copiar diário (CSV)**: copiam os dados para colar no Excel ou no Google Sheets. O formato está em [docs/dados.md](docs/dados.md).

### Dica para ver o aprendizado

1. Deixe uns 2 a 5 minutos no **Turbo**.
2. Olhe o gráfico **Duração de cada vida**: a linha preta deve subir.
3. Volte para **1×** e leia os pensamentos. Ele agora decide com mais certeza.

---

## Como funciona (resumo)

```
 ┌────────────┐   sentidos (62 números)   ┌──────────────┐
 │  Floresta  │ ────────────────────────▶ │   Cérebro    │
 │ (ambiente) │                           │ rede neural  │
 │            │ ◀──────────────────────── │   62→48→48→6 │
 └────────────┘     ação escolhida        └──────────────┘
        │                                        ▲
        │ recompensa (+ comeu, − morreu)         │ ajusta os pesos
        ▼                                        │
 ┌────────────────────────────┐         ┌────────────────┐
 │ Memória de experiências    │ ──────▶ │ Treino (DQN)   │
 │ (últimas 30.000 jogadas)   │ sorteia │ Adam, lotes 32 │
 └────────────────────────────┘         └────────────────┘
```

1. **Sentir:** a cada decisão o esquilo transforma o que vê, cheira e ouve em 62 números.
2. **Decidir:** a rede neural dá uma nota (valor Q) para cada uma das 6 ações. Normalmente ele escolhe a maior. Às vezes sorteia uma ação qualquer (curiosidade).
3. **Agir:** a ação dura 2 passos da simulação.
4. **Receber recompensa:** pegar e comer nozes dá pontos, morrer tira 1 ponto.
5. **Lembrar:** a jogada (o que via, o que fez, quanto ganhou, o que passou a ver) vai para uma memória de 30.000 experiências.
6. **Aprender:** a cada 2 decisões, ele sorteia 32 lembranças e ajusta a rede para prever melhor quanto cada ação vale a longo prazo.

**As 6 ações:** andar em frente, virar à esquerda, virar à direita, enterrar noz, comer noz, parar e observar.

**Os pensamentos não vêm de uma IA de linguagem.** São frases montadas a partir dos números reais do cérebro: o que os sensores captam, a fome, a ação escolhida e a certeza dela. Por isso refletem de verdade o que está acontecendo lá dentro.

A explicação completa, com fórmulas, está em [docs/como-funciona.md](docs/como-funciona.md).

---

## Estrutura do repositório

```
floresta-da-avela/
├── index.html              ← o projeto inteiro (HTML + CSS + JavaScript)
├── README.md               ← este arquivo
└── docs/
    ├── como-funciona.md    ← aprendizado por reforço, DQN, recompensas, sentidos
    ├── arquitetura.md      ← organização do código, funções e fluxo de dados
    ├── parametros.md       ← todas as constantes ajustáveis e o efeito de cada uma
    └── dados.md            ← formato do CSV, do diário e do que fica salvo no navegador
```

---

## Documentação completa

| Documento | Para quem | Conteúdo |
|---|---|---|
| [Como funciona](docs/como-funciona.md) | Quem quer entender a IA | Aprendizado por reforço, DQN passo a passo, sentidos, ações, tabela de recompensas, raposa, estações e como os pensamentos são gerados. |
| [Arquitetura do código](docs/arquitetura.md) | Quem quer mexer no código | Seções do `index.html`, laço principal, funções principais, estado global e como adicionar coisas novas. |
| [Parâmetros](docs/parametros.md) | Quem quer fazer experimentos | Todas as constantes com valor atual, o que fazem e o que acontece se aumentar ou diminuir. |
| [Dados e exportação](docs/dados.md) | Quem quer analisar os resultados | Colunas do CSV, tipos de evento do diário e formato salvo no `localStorage`. |

---

## Personalizando

Todas as regras ficam em constantes no começo do `<script>` do `index.html`. Alguns experimentos legais:

| Quer... | Mude |
|---|---|
| Raposa mais perigosa | `FOX_CHASE` para `1.7` ou `FOX_SIGHT` para `200` |
| Inverno mais longo e difícil | `WINTER` para `2400` |
| Esquilo com mais fome | `HUNGER_RATE` para `1 / 1100` |
| Cérebro maior | `HID` para `[64, 64]` |
| Aprender mais rápido (e mais instável) | `LR` para `1e-3` |
| Explorar por mais tempo | `EPS_DECAY` para `80000` |

> Quando mudar `NIN` ou `HID`, troque também `KEY` (por exemplo para `'avela-cerebro-v3'`). Assim o navegador não tenta carregar um cérebro salvo com outro formato. Detalhes em [docs/parametros.md](docs/parametros.md).

---

## Perguntas frequentes

**Ele está mesmo aprendendo ou é tudo programado?**
Ele está aprendendo. Nenhuma regra do tipo "se vir noz, vá até ela" foi escrita. O código só define o que ele sente, o que pode fazer e quando ganha ou perde pontos. O comportamento sai do treino da rede neural. Clique em **Cérebro novo** para ver como ele age sem treino.

**Por que às vezes ele faz uma besteira mesmo depois de treinado?**
Duas razões. Primeiro, ele sempre guarda no mínimo 4% de curiosidade e às vezes sorteia uma ação. Segundo, a rede é pequena e o mundo tem acaso: nozes caem em lugares aleatórios e a raposa anda sem padrão.

**Quanto tempo leva para ele ficar bom?**
No Turbo, uns 2 a 5 minutos, dependendo do computador. Em testes, a média de vida passou de cerca de 15–29 segundos para cerca de 40 segundos, e o recorde passou de 5 minutos.

**Perdi o progresso ao abrir em outro navegador.**
O cérebro é salvo só no navegador onde você treinou. Outro navegador, outro computador ou uma janela anônima começam do zero.

**Os pensamentos são gerados pelo ChatGPT ou pelo Claude?**
Não. São modelos de frases preenchidos com os números do cérebro. Veja a seção *Pensamentos* em [docs/como-funciona.md](docs/como-funciona.md).

**Funciona no celular?**
Funciona. O layout se adapta, mas o treino no Turbo é mais lento em aparelhos fracos.

---

## Limitações conhecidas

- **Horizonte curto:** com `GAMMA = 0.98`, o cérebro planeja bem uns 50 passos à frente. Enterrar comida para o inverno é longo demais para ele descobrir sozinho, por isso existe uma pequena recompensa extra por enterrar quando está de barriga cheia.
- **A raposa continua sendo a maior causa de morte.** Fugir exige ouvir a raposa, achar a árvore mais próxima e correr na direção certa, tudo junto. É a habilidade que ele mais demora a aprender.
- **Sem GPU:** o treino roda na thread principal do navegador. Em computadores lentos o Turbo treina menos por segundo.
- **Dados locais:** histórico e cérebro ficam só no `localStorage` do navegador. Não há sincronização.

---

## Ideias para evoluir

- Mover o treino para um **Web Worker** e deixar o Turbo bem mais rápido.
- Ter **vários esquilos** ao mesmo tempo, com cérebro compartilhado ou competindo.
- **Evolução (neuroevolução):** os que vivem mais "têm filhotes" com cérebros parecidos.
- **Ciclo de dia e noite**, com a raposa mais ativa à noite.
- **Exportar e importar o cérebro** em JSON para trocar com amigos.
- Uma versão com **pensamentos gerados por um modelo de linguagem** a partir dos mesmos números.

---

Feito com JavaScript puro, Canvas 2D e muita paciência de esquilo.
