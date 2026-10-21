# Carro, piloto ou estratégia? A Fórmula 1 na era híbrida (2014–2025)

Projeto Baseado em Problemas (PjBL) da disciplina de **Data Analytics — PUCPR**.

## Sobre

Na era híbrida da Fórmula 1 (2014–2025), quanto do resultado de uma corrida depende do carro, do piloto, da estratégia e da confiabilidade? E o regulamento de 2022 mudou esse equilíbrio?

O repositório tem duas entregas, no mesmo notebook:

- **RA1 — Ingestão e preparação dos dados:** download, diagnóstico de qualidade (Q1–Q14), limpeza justificada (D1–D13) e um conjunto de dados validado.
- **RA2 — Processamento em streaming:** Spark Structured Streaming com chegada simulada de voltas e uma janela *tumbling* de 5 minutos (tempo médio de volta por piloto).

## Perguntas analíticas

| ID | Pergunta |
|---|---|
| P1 | Em quais circuitos a posição de largada mais determina a posição final, e essa dependência diminuiu após o regulamento de 2022? |
| P2 | Quais pilotos superam o companheiro de equipe de forma consistente, tanto na classificação quanto na corrida? |
| P3 | A estratégia de pit stop (número de paradas e volta da 1ª parada) está associada a ganho ou perda de posições em relação à largada? |
| P4 | Como a taxa de abandono por falha mecânica evoluiu por equipe, e equipes menos confiáveis perderam posições no campeonato por isso? |
| P5 | Quanto o ritmo de volta se degrada ao longo de um stint e essa degradação varia entre circuitos? |

No RA1, as perguntas são apenas definidas. As respostas ficam para os próximos RAs.

## Estrutura

```
├── notebooks/
│   └── pjbl_f1.ipynb          # notebook executado (RA1 + RA2)
├── docs/
│   ├── RA1_Documentacao_F1.docx   # documentação do RA1 no template da disciplina
│   └── pjbl_f1.pdf                # notebook executado no Colab, em PDF
├── DECISOES.md                # registro das decisões de limpeza (D1–D13)
└── README.md
```

Os dados não ficam no repositório: o notebook baixa e gera tudo em `/content/data/` quando roda no Colab.

## Requisitos

- Uma conta Google para usar o **Google Colab**. Python e Java já vêm instalados.
- A primeira célula do notebook instala o que falta: `pyspark==3.5.3` e `kagglehub==1.0.2`.
- Acesso à internet para o download. O dataset é público e não precisa de login no Kaggle.

## Como executar (Google Colab)

1. Abra https://colab.research.google.com.
2. Carregue o notebook `notebooks/pjbl_f1.ipynb`: **Arquivo > Abrir notebook > GitHub**, com o link do repositório, ou **Arquivo > Fazer upload de notebook**.
3. Use **Ambiente de execução > Executar tudo**.
4. Confira os resultados:
   - a primeira célula deve imprimir **"Rodando no Colab: True"**;
   - a seção 2.1 deve mostrar o download do dataset e **"9 arquivos em data/raw"**;
   - a seção 5.4 deve imprimir **"Todas as validações passaram"**;
   - `/content/data/processed/` deve ter os 4 CSVs (painel **Arquivos**, à esquerda);
   - a seção 11 deve mostrar 4 tabelas do streaming, uma após cada lote.

Os arquivos de `/content/` são apagados quando a sessão do Colab termina. Para gerar tudo de novo, basta executar o notebook outra vez.

## Saídas geradas

| Arquivo (em `/content/`) | Uma linha representa | Linhas × colunas |
|---|---|---|
| `data/processed/resultados_limpo.csv` | um piloto em uma corrida | 5.105 × 21 |
| `data/processed/classificacao_limpo.csv` | um piloto na classificação de uma corrida | 5.089 × 8 |
| `data/processed/pit_stops_limpo.csv` | uma parada | 9.181 × 7 |
| `data/processed/voltas_limpo.csv` | uma volta de um piloto | 274.801 × 7 |

## Fonte dos dados e versão

- **Dataset:** *Formula 1 World Championship (1950–2026)*, de Atharv Ranjan, no Kaggle. Os dados vêm do padrão Ergast/Jolpica-F1.
  https://www.kaggle.com/datasets/atharvranjan/formula-1-world-championship-1950-present
- **Versão usada:** **9**. Ela está fixada no código (`.../versions/9`), porque a fonte é atualizada a cada corrida.
- **Licença:** CC0: Public Domain.
- **Tabelas usadas:** races, circuits, results, drivers, constructors, status, qualifying, pit_stops, lap_times.
