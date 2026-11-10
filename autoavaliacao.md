# Autoavaliação do projeto

## Identificação

- **Projeto:** Análise exploratória de pedidos de um app de delivery
- **Módulo:** DS-PY-018 — Análise Exploratória de Dados
- **Autor(a):** Cleisson Godinho do Nascimento
- **Turma:** 1735 - ADA
- **Data:** 09/11/2026

## Visão geral

O projeto analisou 8.058 pedidos realizados em 2024 nas cidades de São Paulo, Rio de Janeiro e Belo Horizonte. O objetivo foi transformar uma base com problemas de qualidade em informações úteis para o planejamento da diretoria, investigando operação, comportamento dos clientes e alavancas de negócio, especialmente chuva e cupons.

A análise foi estruturada desde a inspeção inicial até a publicação dos resultados. Foram realizadas limpeza, tratamento de valores inválidos, análise de outliers, estatística descritiva, sumarizações em diferentes granularidades, visualizações, correlações, testes de hipótese e recomendações de negócio.

## 1. Preparação e qualidade dos dados

A base original foi preservada em `bruto`, enquanto as transformações foram realizadas em `df`. Essa separação permitiu comparar os dados antes e depois da limpeza e evitou alterações acidentais no arquivo original.

Na inspeção inicial, foram identificados os seguintes problemas:

- `data_hora` estava armazenada como texto;
- `valor_pedido` continha valores numéricos e valores no formato monetário brasileiro, como `R$ 44,02`;
- `cidade` apresentava 13 grafias para apenas três cidades;
- `forma_pagamento` apresentava oito grafias para quatro categorias;
- `plataforma`, `avaliacao` e `distancia_km` possuíam valores faltantes;
- `tempo_entrega_min` continha oito valores negativos e 25 códigos `999`;
- `pais` e `moeda` eram colunas constantes;
- não foram encontradas linhas totalmente duplicadas nem `pedido_id` repetido.

As decisões de limpeza foram tomadas com base no significado de cada variável. As colunas constantes foram removidas automaticamente. A data foi convertida para `datetime`, e foram criadas as colunas `data`, `mes`, `hora` e `dia_semana`. As categorias de cidade e forma de pagamento foram padronizadas, `valor_pedido` foi convertido para número sem gerar valores ausentes e `cupom` foi convertido para booleano.

Os 153 valores faltantes de `plataforma` foram classificados como `Não informado`, pois não era possível identificar a plataforma original. Os 1.342 faltantes de `avaliacao` foram mantidos, porque a avaliação é opcional e o preenchimento artificial criaria notas não fornecidas pelos clientes. Os 315 faltantes originais de `distancia_km` também foram mantidos, pois não havia informação suficiente para reconstruí-los com segurança durante a etapa de limpeza.

## 2. Outliers: erro versus extremo legítimo

Um dos principais aprendizados do projeto foi não tratar automaticamente todo outlier como erro. Os critérios estatísticos foram usados para localizar valores que mereciam investigação, mas a decisão final considerou o contexto operacional.

Em `tempo_entrega_min`, os oito valores negativos e os 25 códigos `999` foram considerados inválidos. Esses 33 registros foram transformados em `NaN`, sem remoção de linhas, e identificados pela flag `tempo_invalido`.

Em `distancia_km`, o IQR sinalizou 194 valores e o z-score sinalizou sete. A inspeção revelou uma quebra clara entre 24,39 km e 160,26 km. Assim, foram mantidas as distâncias de até 24,39 km como extremos possíveis e classificadas como inválidas apenas as sete distâncias acima de 100 km. Esses valores foram transformados em `NaN` e registrados pela flag `distancia_invalida`.

Em `valor_pedido`, os critérios automáticos produziram resultados diferentes conforme o contexto. O IQR aplicado à base inteira sinalizou 352 pedidos; o IQR dentro de cada cozinha, 214; e o z-score por cozinha, 32. Isso mostrou que o padrão de ticket depende do tipo de cozinha. A razão entre o valor do pedido e a mediana da cozinha revelou um salto entre 20,45 e 30,94, classificando apenas um pedido como `pedido_grande`. O pedido de R$ 2.299,10 foi mantido por apresentar distância, taxa e tempo de entrega plausíveis, podendo representar um pedido coletivo ou corporativo.

## 3. Estatística descritiva e escolha das métricas

A análise descritiva mostrou que `valor_pedido` era a variável mais assimétrica, com assimetria de 17,52 e coeficiente de variação de 84,22%. A média do ticket foi de R$ 71,55 e a mediana de R$ 61,61. Por causa da forte assimetria e dos pedidos muito altos, a mediana foi escolhida como medida principal do ticket típico.

A retirada do único pedido marcado como grande reduziu a média geral de R$ 71,55 para R$ 71,28, enquanto a mediana permaneceu em R$ 61,61. Isso confirmou que a mediana é mais resistente a valores extremos.

A avaliação média foi de 4,35 em uma escala de 1 a 5, com mediana 4 e moda 5. A interpretação considerou somente os 6.716 pedidos avaliados, sem preencher as avaliações ausentes.

## 4. Granularidade e métricas operacionais

A análise utilizou diferentes níveis de granularidade porque cada pergunta exigia um nível específico de observação.

Na granularidade diária, foram identificados três dias anômalos: 16/07, com 41 pedidos; 19/07, com 40; e 14/08, com apenas um pedido, associado à instabilidade do aplicativo. Na granularidade semanal, a última semana não foi comparada diretamente com as demais porque possuía apenas dois dias. Na granularidade mensal, julho e dezembro se destacaram pelo maior volume.

A média diária foi usada para comparar os dias da semana, evitando favorecer segunda e terça-feira, que ocorreram 53 vezes em 2024, enquanto os demais dias ocorreram 52 vezes. Sexta-feira apresentou a maior média, com 26,52 pedidos por dia, seguida de sábado, com 26,25.

Para chuva, o volume foi analisado por `cidade × data`, pois a condição de chuva era igual para todos os pedidos de uma cidade em um mesmo dia. Essa escolha evitou dar peso excessivo aos dias que já possuíam mais pedidos.

Os percentis do tempo de entrega foram P50 = 35 minutos, P75 = 43, P90 = 50 e P95 = 55. Entre os pedidos com tempo válido, 1.476, ou 18,39%, ultrapassaram 45 minutos. Isso significa que quase um em cada cinco pedidos avaliáveis quebraria uma promessa de entrega em até 45 minutos.

## 5. Visualizações e storytelling

Os gráficos foram construídos com um padrão visual único, incluindo cores consistentes, títulos-conclusão, unidades nos eixos, destaque no dado principal e redução de elementos desnecessários.

As escolhas de visualização foram orientadas pela pergunta de negócio:

- histograma para mostrar a assimetria do ticket;
- barras para participação por cidade;
- boxplot para comparar a distribuição do ticket entre cozinhas;
- série temporal com média móvel para mostrar evolução, campanha e instabilidade;
- mapa de calor para destacar os picos de hora e dia da semana;
- curvas de densidade para comparar tempos com e sem chuva;
- linha com intervalo de confiança para relacionar tempo e avaliação;
- painéis de barras para conectar uso de cupom e clientes novos.

A figura principal reuniu as três mensagens mais relevantes para a diretoria: a chuva aumenta o tempo de entrega, julho elevou o uso de cupons e pedidos com cupom apresentam ticket mediano menor.

## 6. Correlações

As correlações de Pearson e Spearman foram comparadas porque as variáveis possuíam diferentes distribuições e escalas. Spearman foi especialmente útil para `avaliacao`, que é ordinal, e para relações monotônicas.

A distância apresentou correlação positiva com o tempo de entrega. Na amostra de 2.000 pedidos, as inclinações das retas com e sem chuva foram semelhantes, mas o intercepto com chuva foi aproximadamente 9,56 minutos maior. Isso indicou que a chuva desloca a relação para cima, aumentando o tempo esperado para uma mesma distância.

A correlação de Pearson entre distância e taxa de entrega foi de 0,578 na base completa, mas chegou a 1,000 entre os pedidos com frete pago. A relação estimada foi `taxa_entrega = 4,99 + 0,90 × distancia_km`. Esse resultado mostrou que os pedidos com taxa zero enfraqueciam a correlação geral e que seria possível estimar distâncias faltantes em pedidos com taxa positiva, embora essas estimativas precisassem ser identificadas como imputadas.

## 7. Testes de hipótese

Os testes foram realizados com nível de significância de 5% e acompanhados por tamanhos de efeito, evitando conclusões baseadas somente no p-valor.

### H1 — Chuva × tempo de entrega

O teste de Mann-Whitney mostrou que a mediana passou de 33 minutos sem chuva para 42 minutos com chuva. A diferença de 9 minutos foi estatisticamente significativa e operacionalmente relevante. H0 foi rejeitada.

### H2 — Cupom × ticket

O ticket mediano foi de R$ 57,22 com cupom e R$ 62,82 sem cupom. O Mann-Whitney confirmou a diferença de R$ 5,60. H0 foi rejeitada. O teste t sobre o valor bruto foi considerado frágil devido à forte assimetria do ticket e aos valores extremos.

### H3 — Tempo × avaliação

A correlação de Spearman foi −0,2444, indicando associação negativa de pequena magnitude. Pedidos entregues em até 35 minutos tiveram avaliação média de 4,488 e mediana 5; acima de 35 minutos, média de 4,203 e mediana 4. A diferença média foi de 0,285 ponto e a correlação bisserial de postos foi 0,2120. H0 foi rejeitada.

### H4 — Tipo de cozinha × ticket

O Kruskal-Wallis e a ANOVA indicaram diferenças entre as cozinhas. O epsilon ao quadrado foi 0,4058 e o eta ao quadrado, 0,1512. A cozinha Japonesa apresentou mediana de R$ 107,97 e Doces, R$ 38,42, uma diferença de R$ 69,55. H0 foi rejeitada.

### H5 — Forma de pagamento × cidade

O qui-quadrado apresentou p-valor de 0,6951 e V de Cramér de 0,0155. Não houve evidência de associação relevante entre cidade e forma de pagamento. H0 não foi rejeitada. Como contraste, `cupom × cliente_novo` apresentou associação significativa, com V de Cramér de 0,2320.

### H6 — Julho × uso de cupom

O uso de cupom foi de 45,62% em julho e 18,44% nos demais meses, diferença de 27,18 pontos percentuais e V de Cramér de 0,2008. Após controlar `cliente_novo`, a diferença permaneceu entre clientes novos e não novos. H0 foi rejeitada.

## 8. Principais achados

1. A chuva elevou o tempo mediano de entrega em 9 minutos, de 33 para 42.
2. 18,39% dos pedidos com tempo válido ultrapassaram 45 minutos.
3. Pedidos acima de 35 minutos tiveram avaliação média 0,285 ponto menor.
4. O uso de cupom chegou a 45,62% em julho, contra 18,44% nos demais meses.
5. O ticket mediano com cupom foi R$ 5,60 menor que sem cupom.

## 9. Recomendações

1. Reforçar a disponibilidade de entregadores e ajustar a previsão de prazo nos dias chuvosos, acompanhando a mediana do tempo na chuva e o percentual acima de 45 minutos.
2. Criar alertas operacionais antes de o pedido ultrapassar 35 minutos, acompanhando a proporção entregue até esse limite e a avaliação dos pedidos atrasados.
3. Testar cupons com valor mínimo e foco em aquisição incremental, acompanhando taxa de uso, ticket mediano, participação de clientes novos e recorrência futura.

## 10. Limitações

- Os dados são observacionais e não permitem concluir causalidade.
- Não existem variáveis de trânsito, disponibilidade de entregadores, tempo de preparo ou quantidade de itens.
- A avaliação é opcional e possui 16,65% de faltantes, podendo haver viés de resposta.
- A base cobre somente 2024 e três cidades.
- Não existe `cliente_id`, impedindo análises de recorrência e retenção.
- Não há custo do cupom nem margem do pedido, portanto não foi possível avaliar rentabilidade promocional.

## 11. Reprodutibilidade e publicação

A base original foi mantida sem alterações em `dados/delivery_pedidos.csv`. A base tratada foi salva em `saidas/delivery_pedidos_limpo.csv`, sem colunas auxiliares de cálculo, mas mantendo as flags de qualidade. A figura principal foi salva em `saidas/figura_principal.png`. O README documenta perguntas, dados, metodologia, achados, hipóteses, recomendações, limitações e instruções de execução.

Antes da entrega, o projeto deve ser validado com **Kernel → Restart & Run All**, conferindo a recriação dos arquivos de saída e a ausência de erros não controlados.

## Pontos fortes

- A limpeza foi documentada e preservou todas as 8.058 linhas, usando flags para manter a rastreabilidade dos problemas.
- Outliers foram investigados no contexto do negócio, distinguindo erros de extremos legítimos.
- As análises respeitaram a granularidade adequada para cada pergunta, especialmente em chuva, tempo e séries temporais.
- Os testes de hipótese incluíram tamanhos de efeito e traduções para o negócio, não apenas p-valores.
- As visualizações seguiram um padrão consistente e priorizaram títulos-conclusão e leitura executiva.
- O projeto foi organizado para publicação, com base limpa, figura principal, README e requirements.

## Pontos a melhorar

- Automatizar mais verificações de integridade da base e dos arquivos gerados.
- Incluir intervalos de confiança e análises de sensibilidade em mais métricas executivas.
- Testar modelos que controlem simultaneamente chuva, distância, cidade, horário e cozinha.
- Aprimorar a avaliação financeira dos cupons com dados de custo, margem e retenção.
- Validar a reprodutibilidade em um ambiente limpo antes da publicação final.

## Principal aprendizado

O principal aprendizado foi compreender que uma análise exploratória de qualidade não consiste apenas em calcular métricas ou remover valores discrepantes. Foi necessário preservar o dado original, justificar cada transformação, escolher a granularidade correta, distinguir erro de extremo legítimo, selecionar testes coerentes com os tipos das variáveis e traduzir significância estatística em impacto para o negócio. O projeto também mostrou que uma análise só se torna útil para a empresa quando é reproduzível, visualmente clara e capaz de sustentar decisões práticas.

## Autoavaliação final

Considero que o projeto cumpriu os objetivos propostos ao integrar preparação de dados, análise exploratória, estatística, visualização e comunicação executiva. As decisões mais importantes foram sustentadas por evidências e mantiveram a rastreabilidade dos dados. O trabalho também reconheceu limitações e evitou conclusões causais indevidas. Como evolução, a próxima etapa seria transformar os achados em experimentos e indicadores operacionais contínuos, especialmente para chuva, atrasos e cupons.
