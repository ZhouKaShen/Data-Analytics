# Decisões de limpeza e preparação

Este arquivo reúne as decisões tomadas para limpar e preparar os dados do RA1. Ele serve de base para a seção 04 do documento do RA1. Os números são da execução com a versão 9 do conjunto de dados.

Cada decisão tem um código, de D1 a D13, e aponta para o problema que a motivou no diagnóstico, de Q1 a Q14. As seções citadas são as do notebook.

**Termos de Fórmula 1 usados aqui:**
- **Grid:** a posição em que o piloto larga.
- **Pit stop:** a parada nos boxes para trocar pneus. **Pit lane** é a faixa por onde o carro entra e sai dos boxes.
- **Classificação:** a sessão que define a ordem de largada. Ela tem três fases eliminatórias, chamadas Q1, Q2 e Q3.
- **Bandeira vermelha:** a corrida é interrompida e os carros esperam parados nos boxes.
- **Carro de segurança:** carro que entra na pista depois de um acidente e obriga todos a andar devagar.
- **Retardatário:** piloto que terminou a corrida, mas uma ou mais voltas atrás do vencedor.

## D1 — Recorte das temporadas de 2014 a 2025
- **Problema relacionado:** Q11, porque a temporada de 2026 ainda está acontecendo e só tem 15 corridas. Também Q14 e Q5, porque quase todos os erros de duração das paradas estão em 2026.
- **O que foi afetado:** o ano das corridas, na tabela `races`, e todas as tabelas ligadas a ela pelo código da corrida, `raceId`.
- **O que fizemos:** ficamos só com as corridas de 2014 a 2025 e mantivemos nos resultados, na classificação, nas paradas e nas voltas apenas as linhas dessas corridas.
- **Por quê:** 2014 é o início da era híbrida, quando os carros passaram a usar motores que combinam combustível e energia elétrica, e 2025 é a última temporada completa. Uma temporada pela metade distorceria a comparação entre períodos.
- **Resultado:** corridas de 1.164 para 252, resultados de 27.568 para 5.105, classificação de 11.298 para 5.089, paradas de 12.783 para 9.181 e voltas de 632.937 para 274.801.
- **Como conferimos:** um `assert` garante que todos os anos estão entre 2014 e 2025, num total de 12 temporadas.
- **Onde está:** seção 4.1.

## D2 — Correção dos tipos de dados
- **Problema relacionado:** Q3.
- **O que foi afetado:** a data da corrida, que estava como texto, e as colunas de posição final e número do carro, que foram lidas como números decimais só porque têm valores vazios.
- **O que fizemos:** transformamos a data em data de verdade e as duas colunas em números inteiros que aceitam vazio.
- **Por quê:** com a data no tipo certo dá para ordenar e comparar corridas, e posição é sempre um número inteiro. Nenhuma coluna numérica estava como texto, porque já avisamos ao pandas, na leitura, que o texto `\N` significa vazio.
- **Resultado:** 3 colunas corrigidas e nenhuma linha removida.
- **Como conferimos:** mostramos os tipos depois da conversão, e um `assert` na seção 5.4 confere o tipo da data.
- **Onde está:** seção 4.2.

## D3 — Tempos convertidos para segundos
- **Problema relacionado:** Q3 e Q4.
- **O que foi afetado:** os tempos das três fases da classificação, que estavam em texto, e os tempos das voltas e das paradas, que existem em milissegundos.
- **O que fizemos:** criamos a função `tempo_para_segundos`, que transforma um texto como 1:27.452 em 87,452 segundos, e criamos as colunas `q1_s`, `q2_s` e `q3_s`. Nas voltas e nas paradas, dividimos os milissegundos por 1.000.
- **Por quê:** um tempo guardado como texto não permite contas, e as perguntas P2 e P5 dependem de comparar tempos. Nas voltas e nas paradas preferimos os milissegundos, que já são números e batem com o texto em todos os casos do período, em vez de interpretar textos com formatos diferentes.
- **Resultado:** 5 colunas novas.
- **Como conferimos:** a quantidade de vazios é a mesma antes e depois da conversão: 78 no Q1, 1.364 no Q2 e 2.650 no Q3. As colunas novas são numéricas.
- **Onde está:** seção 4.3.

## D4 — Vazios com significado foram mantidos
- **Problema relacionado:** Q1 e Q7.
- **O que foi afetado:** 78 tempos vazios no Q1, 1.364 no Q2, 2.650 no Q3 e 776 posições finais vazias.
- **O que fizemos:** **mantivemos** os vazios, sem preencher nada.
- **Por quê:** aqui o vazio é uma informação. O piloto eliminado no Q1 não disputa o Q2, então não tem tempo no Q2. O piloto que abandonou a corrida não tem posição oficial. Preencher esses vazios, por exemplo com uma média, criaria um tempo ou uma posição que nunca existiram. Para ordenar todos os pilotos continua existindo a coluna `positionOrder`, que não tem vazios.
- **Resultado:** nenhuma alteração.
- **Como conferimos:** um `assert` confere que os vazios são os mesmos antes e depois da conversão. As 776 posições vazias são exatamente as 776 linhas com uma letra no lugar da posição.
- **Onde está:** seção 4.3.

## D5 — Status agrupados em 5 categorias
- **Problema relacionado:** Q8, porque existem 141 formas diferentes de terminar uma corrida, e 68 aparecem no período.
- **O que foi afetado:** a coluna `status`, que diz como cada piloto terminou a corrida. Criamos a coluna nova `status_grupo`.
- **O que fizemos:** separamos em 5 grupos:
  - **Terminou:** o status Finished.
  - **Retardatário:** os status que começam com o sinal de mais, como +1 Lap, e o status Lapped.
  - **Acidente:** Accident, Collision, Collision damage, Spun off, Damage, Debris e Puncture.
  - **Outro:** Disqualified, Did not qualify, Did not start, Withdrew, Not classified, Excluded, Retired, Illness, Driver unwell e 107% Rule.
  - **Mecânico:** todo o resto, que são as quebras do carro, como motor, câmbio e freios.

  A coluna `status` original continua na tabela.
- **Por quê:** muitos status dizem quase a mesma coisa, como Engine, Power Unit e Turbo, que são todos problemas de motor. Sem agrupar, não dá para calcular a taxa de quebras por equipe da P4. Manter o original permite que qualquer pessoa confira o agrupamento.
- **Resultado:** Terminou com 2.733, Retardatário com 1.511, Mecânico com 407, Acidente com 315 e Outro com 139.
- **Como conferimos:** revisamos à mão a tabela que mostra cada status ao lado do seu grupo, e a regra definida no início foi mantida. Alguns casos são duvidosos e ficam registrados como limitação: quebra de asa, com 6 casos, e falha de pneu, com 4, ficaram em Mecânico, mas podem ter sido causadas por um toque em outro carro. Pneu furado, com 7 casos, ficou em Acidente. Retired, com 101 casos, ficou em Outro porque não diz o motivo do abandono.
- **Onde está:** seção 4.4.

## D6 — Marcação de quem largou dos boxes
- **Problema relacionado:** Q6.
- **O que foi afetado:** os 79 resultados do período com grid igual a 0.
- **O que fizemos:** criamos a coluna `largou_box`, verdadeira quando o grid é 0 e o piloto não tem o código W. Nenhuma linha foi removida.
- **Por quê:** grid igual a 0 não é erro. Na Fórmula 1 isso indica que o piloto largou da saída dos boxes, e não do grid, por punição ou por troca de peças. Apagar esses casos distorceria as perguntas P1 e P3, que comparam largada e chegada. Os 6 pilotos com código W desistiram antes da largada, então não contam como largada dos boxes. Esse ajuste foi feito depois de olhar os dados.
- **Resultado:** 73 resultados marcados e nenhum removido.
- **Como conferimos:** contagem de 73 verdadeiros e 5.032 falsos, e um `assert` garante que nenhum grid é negativo.
- **Onde está:** seção 4.5.

## D7 — Marcação das paradas longas
- **Problema relacionado:** Q12.
- **O que foi afetado:** as paradas com mais de 60 segundos, 509 das 9.181 do período, ou 5,5%. Uma parada normal leva de 20 a 30 segundos.
- **O que fizemos:** criamos a coluna `parada_longa` e mantivemos todas as linhas.
- **Por quê:** essas paradas se concentram em corridas interrompidas por bandeira vermelha, quando os carros ficam esperando nos boxes. A Austrália em 2023 teve 49, o Brasil em 2016 teve 36 e a Arábia Saudita em 2021 teve 35. São eventos reais, e cada análise pode decidir se filtra ou não.
- **Resultado:** 509 paradas marcadas e nenhuma removida.
- **Como conferimos:** contagem de verdadeiros e falsos.
- **Onde está:** seção 4.5.

## D8 — Marcação das voltas fora do normal
- **Problema relacionado:** Q13.
- **O que foi afetado:** as 274.801 voltas do período.
- **O que fizemos:** criamos a coluna `volta_atipica`, verdadeira em três situações:
  - a primeira volta da corrida, que começa com o carro parado;
  - as voltas ligadas a uma parada, ou seja, a volta em que o piloto entra nos boxes e a seguinte, em que ele sai;
  - as voltas mais de 7% mais lentas que a mediana da corrida. A mediana é o valor do meio quando colocamos todos os tempos em ordem.
- **Por quê:** essas voltas aconteceram de verdade, mas não mostram o ritmo normal do piloto. Muitas são voltas atrás do carro de segurança. Marcar permite separar o ritmo normal para a P5. O limite de 107% vem de uma regra da própria Fórmula 1.
- **Resultado:** 38.998 voltas marcadas, 14,2% do total. São 17.865 ligadas a paradas e 32.478 acima do limite, e uma mesma volta pode ter os dois motivos. Nenhuma foi removida.
- **Como conferimos:** mostramos o percentual e a contagem por motivo.
- **Onde está:** seção 4.5.

## D9 — Coluna `periodo`
- **Problema relacionado:** nenhum. A coluna atende à pergunta P1.
- **O que foi afetado:** o ano da corrida.
- **O que fizemos:** criamos a coluna `periodo`, com o valor 2022-2025 para as corridas a partir de 2022 e 2014-2021 para as anteriores.
- **Por quê:** permite comparar antes e depois da mudança de regras de 2022.
- **Resultado:** 1 coluna nova.
- **Como conferimos:** o menor e o maior ano de cada período, e um `assert` garante que só existem esses dois valores.
- **Onde está:** seção 5.1.

## D10 — Coluna `terminou`
- **Problema relacionado:** Q7.
- **O que foi afetado:** a coluna `status_grupo`.
- **O que fizemos:** criamos a coluna `terminou`, verdadeira para os grupos Terminou e Retardatário.
- **Por quê:** separa quem completou a corrida de quem não completou, o que é usado em P1, P3 e P4.
- **Resultado:** 4.244 verdadeiros e 861 falsos.
- **Como conferimos:** uma tabela cruzando `status_grupo` com `terminou`.
- **Onde está:** seção 5.1.

## D11 — Registros repetidos e códigos órfãos
- **Problema relacionado:** Q2 e Q9.
- **O que foi afetado:** os identificadores de todas as tabelas.
- **O que fizemos:** **nenhuma ação foi necessária.** No período não há registros repetidos nem códigos órfãos, que são códigos sem correspondência na tabela de origem. As 91 repetições da base inteira são de 1950 a 1978, quando dois pilotos podiam dividir o mesmo carro, e já saem com o recorte. Mantivemos uma linha de classificação sem resultado: Lance Stroll no GP da Espanha de 2025, que participou da classificação mas não largou.
- **Por quê:** garantir que cada registro aparece uma vez só e que as tabelas se ligam corretamente, sem apagar registros verdadeiros.
- **Resultado:** nenhuma linha alterada.
- **Como conferimos:** uma tabela de conferência na seção 4.2 e `assert` de registros únicos e de ligação entre tabelas na seção 5.4.
- **Onde está:** seções 4.2 e 5.4.

## D12 — Junção das tabelas
- **Problema relacionado:** nenhum. A junção facilita os próximos RAs.
- **O que foi afetado:** a tabela de resultados recebeu informações das corridas, como ano, etapa, data e nome, dos circuitos, como nome e país, dos pilotos, das equipes e dos status.
- **O que fizemos:** juntamos as tabelas sem perder nenhum resultado. A opção `validate` garante que cada resultado encontra no máximo uma linha em cada tabela, sem duplicar nada.
- **Por quê:** ter uma base única e fácil de ler, com nomes em vez de só códigos.
- **Resultado:** a tabela de resultados ficou com 21 colunas e continua com 5.105 linhas.
- **Como conferimos:** depois de cada junção, um `assert` confere que o número de linhas não mudou.
- **Onde está:** seções 4.4, 5.1 e 5.2.

## D13 — Parada com duração zero
Esta decisão foi incluída depois do diagnóstico, porque o problema não era esperado.
- **Problema relacionado:** Q5 e Q1.
- **O que foi afetado:** uma parada do período, no GP da Áustria de 2025, com duração zero e sem duração em texto. Ela é a segunda parada do piloto de código 852, na corrida de código 1155.
- **O que fizemos:** mantivemos a linha e deixamos a duração vazia.
- **Por quê:** a parada aconteceu e conta no número de paradas da P3, mas não sabemos quanto tempo ela durou. Uma parada não pode durar zero segundos, então manter o zero seria registrar algo impossível.
- **Resultado:** 1 valor alterado.
- **Como conferimos:** um `assert` garante que existe exatamente 1 duração vazia e que todas as outras são maiores que zero.
- **Onde está:** seção 4.3.
