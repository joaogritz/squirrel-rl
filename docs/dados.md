# Dados e exportação

Esta página descreve os dados que a Floresta da Avelã registra: o histórico de vidas, o diário de eventos, o gráfico de treino e o que fica salvo no navegador.

[← Voltar ao README](../README.md)

---

## Exportando

Na seção **Estatísticas e diário**, abaixo da floresta:

- **Copiar vidas (CSV):** copia o histórico das vidas que estão na memória (até 400).
- **Copiar diário (CSV):** copia todos os eventos do diário (até 1.500).

Os dados vão para a área de transferência. Cole numa planilha (Excel, Google Sheets, LibreOffice) ou num arquivo `.csv`. Se o navegador bloquear a cópia, a página mostra o texto numa caixa já selecionada para você copiar com Ctrl+C.

Formato: separado por vírgula, primeira linha com os nomes das colunas, números com ponto decimal. Textos com vírgula ou aspas ficam entre aspas.

> No Google Sheets, colar um CSV pode colocar tudo numa coluna só. Use **Dados → Dividir texto em colunas** com o separador "vírgula".

---

## CSV de vidas

Uma linha por vida.

| Coluna | Tipo | Descrição |
|---|---|---|
| `vida` | inteiro | Número da vida. |
| `duracao_s` | inteiro | Duração em segundos (60 ticks = 1s). |
| `causa` | texto | `fome` ou `raposa`. |
| `nozes_pegas` | inteiro | Nozes pegas do chão. |
| `comidas` | inteiro | Nozes comidas. |
| `enterradas` | inteiro | Nozes enterradas. |
| `desenterradas` | inteiro | Nozes tiradas de esconderijos. |
| `perseguicoes` | inteiro | Vezes em que a raposa começou a perseguir. |
| `fugas` | inteiro | Perseguições que terminaram sem captura (subiu na árvore ou a raposa cansou). |
| `pontuacao` | decimal | Soma das recompensas da vida, já com o −1 da morte. |
| `curiosidade` | 0–1 | Fração das decisões que foram sorteadas. |
| `fome_media` | 0–1 | Fome média ao longo da vida. |
| `tempo_na_arvore` | 0–1 | Fração das decisões tomadas em cima de árvore. |
| `estacao_da_morte` | texto | `outono` ou `inverno`. |
| `acao_andar_em_frente` … `acao_parar_e_observar` | inteiro | Quantas vezes cada uma das 6 ações foi escolhida. |

As colunas de ação ficam assim (acentos e espaços viram `_`):

```
acao_andar_em_frente, acao_virar_esquerda, acao_virar_direita,
acao_enterrar_noz, acao_comer_noz, acao_parar_e_observar
```

> Vidas salvas por versões antigas da página só têm `duracao_s` e `causa`. As outras colunas aparecem como `0`.

### Exemplo

```csv
vida,duracao_s,causa,nozes_pegas,comidas,enterradas,desenterradas,perseguicoes,fugas,pontuacao,curiosidade,fome_media,tempo_na_arvore,estacao_da_morte,acao_andar_em_frente,acao_virar_esquerda,acao_virar_direita,acao_enterrar_noz,acao_comer_noz,acao_parar_e_observar
187,71,raposa,11,8,3,1,2,1,2.84,0.04,0.41,0.08,outono,1102,340,402,61,44,180
188,22,raposa,3,2,0,0,1,0,-0.31,0.04,0.38,0,inverno,410,96,120,6,9,25
```

### Análises interessantes

- **Curva de aprendizado:** `duracao_s` por `vida`, com média móvel.
- **O que mata:** contagem de `causa` em grupos de 20 vidas.
- **Estratégia:** relação entre `enterradas` e sobrevivência no inverno (`estacao_da_morte`).
- **Comportamento:** como a fração de `acao_parar_e_observar` muda entre vidas iniciais e recentes.
- **Fuga:** `fugas / perseguicoes` ao longo do tempo.

---

## CSV do diário

Uma linha por evento.

| Coluna | Tipo | Descrição |
|---|---|---|
| `decisao` | inteiro | Contador global de decisões quando o evento aconteceu. Serve como "relógio". |
| `vida` | inteiro | Número da vida. |
| `tempo_s` | inteiro | Segundos desde o começo daquela vida. |
| `tipo` | texto | Categoria do evento (tabela abaixo). |
| `evento` | texto | Frase descrevendo o que aconteceu. |

### Tipos de evento

| Tipo (no CSV) | Tag interna | Quando acontece |
|---|---|---|
| comida | `comida` | Pegou uma noz do chão ou comeu. |
| guardando | `guardar` | Enterrou ou desenterrou. |
| perigo | `perigo` | A raposa começou a perseguir. |
| a salvo | `salvo` | Subiu na árvore a tempo ou a raposa cansou. |
| fim da vida | `morte` | Morreu de fome ou foi pego. |
| recorde | `recorde` | Uma vida terminou com novo recorde de duração. |
| estação | `estacao` | Começou o outono ou o inverno. |
| pensando | `pensa` | Mensagens do sistema (nasceu, acordou com cérebro salvo, cérebro novo). |

Os pensamentos de raciocínio do dia a dia ("Noz à esquerda, vou buscar") **não** entram no diário, porque seriam milhares por minuto. Eles aparecem só na aba Pensamentos.

---

## Gráfico "Curiosidade e erro do cérebro"

A cada 2.000 decisões, um ponto é gravado em `series`:

| Campo | Descrição |
|---|---|
| `d` | Número da decisão. |
| `eps` | Curiosidade (0–1). |
| `loss` | Média móvel exponencial (fator 0,01) do erro² dos lotes de treino. |
| `avg` | Média de duração das últimas 20 vidas, em segundos. |

No gráfico, o erro aparece **relativo**: dividido pelo maior erro registrado, para caber na mesma escala de 0–100% da curiosidade.

---

## O que fica salvo no navegador

A página salva tudo em `localStorage`, na chave `avela-cerebro-v2`:

- a cada 20 segundos;
- quando você troca de aba ou minimiza (`visibilitychange`);
- ao fechar a página (`pagehide`).

Formato (JSON):

```jsonc
{
  "v": 1,                       // versão do formato
  "layers": [                   // pesos da rede principal, camada por camada
    { "W": [ /* 62×48 números */ ], "b": [ /* 48 */ ] },
    { "W": [ /* 48×48 */ ],        "b": [ /* 48 */ ] },
    { "W": [ /* 48×6 */ ],         "b": [ /* 6 */ ] }
  ],
  "decisions": 125980,          // decisões totais (controla a curiosidade)
  "trainSteps": 62491,          // passos de treino
  "history": [ /* últimas 300 vidas */ ],
  "record": 10009,              // recorde em ticks
  "totals": { "ate": 478, "buried": 188 },
  "life": 190,                  // número da próxima vida
  "series": [ /* últimos 300 pontos do gráfico de treino */ ],
  "log": [ /* últimos 300 eventos do diário */ ]
}
```

O que **não** é salvo, e por isso recomeça ao recarregar a página:

- a memória de experiências (30.000 lembranças): o treino recomeça depois de juntar 1.000 novas;
- o otimizador Adam e a rede-alvo (a rede-alvo é recriada como cópia da principal);
- as posições do mundo, a estação atual e a vida em andamento.

Os pesos são arredondados para 5 casas decimais. O arquivo salvo fica com uns 100 a 200 KB.

### Apagando os dados

- Pela página: **Cérebro novo** → confirmar.
- Pelo navegador: DevTools (F12) → **Application** → **Local Storage** → apague a chave `avela-cerebro-v2`.

### Levando o cérebro para outro navegador (manual)

1. No navegador de origem, abra o console (F12) e rode:

   ```js
   copy(localStorage.getItem('avela-cerebro-v2'))
   ```

2. No navegador de destino, com a página aberta, rode no console (cole o texto copiado entre as crases):

   ```js
   localStorage.setItem('avela-cerebro-v2', `COLE_AQUI`)
   ```

3. Recarregue a página **sem** deixar ela salvar antes. Se ela salvar por cima, repita o passo 2 e recarregue logo em seguida.

[← Voltar ao README](../README.md)
