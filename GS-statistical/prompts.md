# Registro de Prompts — Global Solution 2026.2

**Projeto:** Ponto Cego
**Equipe:** Frota 42
**Disciplina:** Data Science & Statistical Computing
**Integrantes:** Felipe Balbino Murad - RM562347 e Lucas Mesquita Massoni - RM561686
**Ferramenta de IA utilizada:** Adapta ONE

---

## Como este arquivo deve ser lido

O briefing da Global Solution permite o uso de IA como copiloto e exige que os
prompts sejam registrados. Cada entrada abaixo documenta: o que foi pedido, por
que foi pedido, o que a IA devolveu e o que a equipe fez com isso.

O código entregue não foi copiado sem leitura. Cada célula do notebook foi
executada, conferida contra os números esperados e explicada por algum
integrante. As decisões técnicas listadas na seção 3 foram tomadas pela equipe
com base nos resultados, não aceitas automaticamente.

---

## 1. Registro de prompts

### 01 — Definição do problema e das variáveis-alvo

**Prompt:** "Estamos fazendo a Global Solution da FIAP sobre lotação de ônibus
urbano. O cidadão não sabe se o próximo ônibus está lotado antes de decidir
embarcar. Precisamos definir duas variáveis-alvo: uma numérica para regressão
linear e uma binária para regressão logística, ambas ligadas ao mesmo problema."

**Objetivo:** fechar o escopo da disciplina antes de escrever qualquer código.

**Resultado:** alvo numérico = taxa de ocupação (passageiros a bordo dividido
pela capacidade); alvo binário = ônibus lotado (ocupação maior ou igual a 80%
da capacidade). Sugestão de horizonte de previsão e de hipóteses H1 e H2.

**Decisão da equipe:** adotamos 80% como limiar de lotado porque é o ponto em
que o passageiro deixa de ter assento garantido. Definimos duas hipóteses
testáveis: H1 (o padrão de ocupação é previsível sem sensor) e H2 (a leitura da
câmera do ponto anterior reduz o erro).

---

### 02 — Estrutura da base simulada

**Prompt:** "Como não temos acesso a dados reais de contagem de passageiros,
precisamos simular uma base plausível. Quais variáveis uma base de APC
(automatic passenger counting) deveria ter e como gerar os padrões de forma
que a base represente a realidade sem ficar artificialmente simples?"

**Objetivo:** construir uma base defensável, com fonte declarada.

**Resultado:** estrutura com viagem_id, linha, sentido, veículo, data_hora,
ponto_seq, embarques, desembarques, chuva_mm e temp_c. Perfil horário com dois
picos (7h e 18h), variação por dia da semana, efeito de chuva na demanda e
acúmulo de passageiros ao longo do itinerário.

**Decisão da equipe:** três linhas com características diferentes (uma mais
carregada, uma média e uma mais leve) para permitir comparação entre linhas.
Período de 45 dias, de 01/08 a 15/09, com o corte do split em 01/09 — o que
deixa 31 dias para treino e 14 dias para teste.

---

### 03 — Quantidade de viagens por dia

**Prompt:** "Quantas viagens por dia e quantos pontos por linha são suficientes
para treinar os modelos sem gerar uma base grande demais para o Colab?"

**Objetivo:** dimensionar a base.

**Resultado:** 10 viagens por dia por linha, em dois sentidos, com 28 a 42
pontos por linha, resultando em cerca de 93.600 registros e 2.700 viagens.

**Decisão da equipe:** adotamos o dimensionamento sugerido. O volume ficou
adequado: roda em segundos no Colab e é suficiente para os modelos.

---

### 04 — Features de trajeto e tempo

**Prompt:** "Quais features construir a partir de hora, dia da semana e posição
no trajeto? Como representar hora de forma que o modelo entenda que 23h e 0h
são próximas?"

**Objetivo:** preparar as variáveis de entrada.

**Resultado:** duas alternativas sugeridas — codificação cíclica (seno e cosseno)
para hora e dia da semana, e o termo quadrático da posição no trajeto.

**Decisão da equipe:** mantivemos o termo quadrático, que se mostrou essencial
para representar a desaceleração da ocupação no fim do percurso. A codificação
cíclica foi implementada na preparação dos dados, mas descartada do modelo
final: com os dois picos estreitos de 7h e 18h, a hora como variável categórica
reproduziu o perfil horário com muito mais fidelidade do que a harmônica
senoidal. A comparação está documentada na entrada 08.

---

### 05 — Leitura da câmera sem vazamento

**Prompt:** "Como usar a ocupação observada pela câmera no ponto anterior sem
criar vazamento de dados? Precisamos que o modelo use apenas informação
disponível no momento da previsão."

**Objetivo:** garantir rigor metodológico na variável central do projeto.

**Resultado:** uso de deslocamento (shift) de uma posição dentro de cada
viagem, garantindo que o modelo receba o valor do ponto k-1 ao prever o ponto
k. Inclusão de indicador binário para registrar quando a leitura existe.

**Decisão da equipe:** adotamos o deslocamento e mantivemos o indicador
binário. A primeira parada de cada viagem não tem leitura anterior, o que
representa 2,9% dos registros — tratamos isso com imputação e não com zero,
porque zero ensinaria ao modelo que ônibus sem câmera estão vazios.

---

### 06 — Divisão treino e teste

**Prompt:** "Como dividir a base entre treino e teste de forma temporal, sem
que o mesmo dado apareça nos dois lados?"

**Objetivo:** cumprir o critério de avaliação que exige split sem vazamento.

**Resultado:** divisão por data, com 70% do período para treino e 30% para
teste, além de três verificações automáticas de integridade.

**Decisão da equipe:** adotamos a divisão temporal (não aleatória), porque o
uso real é prever o futuro a partir do passado. A primeira versão da divisão
usava o horário de cada registro e falhou: uma viagem que começou antes da
meia-noite ficou dividida entre os dois conjuntos. A verificação automática
detectou o problema e a correção foi atribuir cada viagem inteira ao dia em
que começou. Esse caso está documentado na seção 4.

---

### 07 — Baseline de comparação

**Prompt:** "Como provar que o modelo é melhor que uma solução trivial? Que
baseline usar para que as métricas tenham significado?"

**Objetivo:** evitar a armadilha de reportar R² sem referência.

**Resultado:** baseline calculado como a média histórica de ocupação por linha
e horário — o que um sistema sem modelo nenhum faria.

**Decisão da equipe:** adotamos esse baseline porque ele usa as duas variáveis
mais informativas disponíveis (linha e hora). Isso elevou a régua: o modelo
teria que superar uma referência forte, não uma fraca.

---

### 08 — Regressão linear: primeira versão e diagnóstico

**Prompt:** "Montamos a regressão linear com features de trajeto, clima e
codificação cíclica de hora. O modelo perdeu para o baseline. O que está
errado e como corrigir?"

**Objetivo:** entender por que o modelo não agregava.

**Resultado:** duas causas identificadas. A primeira, o baseline usava a linha
e o modelo não recebia essa informação, além de representar a hora com uma
única harmônica — insuficiente para reproduzir dois picos estreitos. A segunda,
as variáveis posição relativa e paradas restantes carregavam a mesma
informação, gerando coeficientes opostos de mesmo tamanho.

**Decisão da equipe:** passamos linha e hora como variáveis categóricas,
removemos a variável redundante e adicionamos o termo quadrático. O modelo
passou a superar o baseline com folga.

---

### 09 — Teste da hipótese H2

**Prompt:** "Como comparar dois modelos — um sem a leitura da câmera e outro
com ela — para medir se o sensor realmente agrega? Precisamos também saber em
que parte do trajeto a câmera faz mais diferença."

**Objetivo:** testar H2 de forma isolada.

**Resultado:** comparação com as mesmas features e o mesmo conjunto de teste,
variando apenas a presença da variável de ocupação observada. Análise adicional
do erro por faixa do trajeto (0-25%, 25-50%, 50-75%, 75-100%).

**Decisão da equipe:** adotamos a análise por faixa porque ela mostra não só se
a câmera agrega, mas onde agrega — informação que virou a recomendação central
do projeto.

---

### 10 — Ajuste do horizonte de previsão

**Prompt:** "O modelo com câmera chegou a R² de 0,97 prevendo o ponto seguinte.
Isso é realista? Que horizonte de previsão representa o uso real do produto?"

**Objetivo:** evitar um resultado tecnicamente correto mas sem valor prático.

**Resultado:** diagnóstico de que prever a parada seguinte é uma tarefa quase
aritmética, porque entre duas paradas consecutivas entram e saem pouquíssimas
pessoas. Não havia vazamento, apenas um horizonte curto demais. Sugestão de
horizonte de 5 paradas, equivalente a cerca de um terço do itinerário.

**Decisão da equipe:** adotamos 5 paradas. O critério foi o comportamento do
usuário: ele consulta o aplicativo quando o ônibus está distante, não quando
já está chegando. Com esse horizonte, o R² caiu para uma faixa crível e o ganho
da câmera passou a ser mensurável de forma honesta.

---

### 11 — Regressão logística e escolha do threshold

**Prompt:** "Como escolher o ponto de corte da regressão logística? O que é
pior para o cidadão: ser avisado de lotação que não existe ou não ser avisado
quando o ônibus está realmente lotado?"

**Objetivo:** definir o threshold com critério de produto, não estatístico.

**Resultado:** análise do trade-off entre precisão e recall em faixas de 0,20 a
0,80, e discussão da assimetria de custo entre os dois tipos de erro.

**Decisão da equipe:** adotamos 0,50. O raciocínio foi: falso negativo faz o
cidadão embarcar lotado e chegar atrasado; falso positivo faz ele esperar o
próximo veículo. Como o primeiro custa muito mais, priorizamos recall. O ponto
de corte foi escolhido pela equipe, não pelo valor que maximiza uma métrica.

---

### 12 — Visualizações

**Prompt:** "Quais quatro gráficos contam a história do projeto de forma
completa para um avaliador que nunca viu o trabalho?"

**Objetivo:** cumprir o requisito de visualizações e sustentar o pitch.

**Resultado:** perfil horário, curva de acumulação ao longo do trajeto, real
versus previsto comparando os dois modelos, e barras de erro por faixa do
trajeto com os percentuais de redução.

**Decisão da equipe:** mantivemos os quatro. O último é o que melhor comunica o
achado central: a câmera não ajuda no início do percurso e corta o erro pela
metade no meio.

---

### 13 — Recomendações e limitações

**Prompt:** "Como escrever recomendações que sejam fundamentadas nas previsões
e que declarem limitações e impactos, em vez de afirmações genéricas?"

**Objetivo:** atender o critério de maior peso da rubrica.

**Resultado:** estrutura em que cada recomendação declara o número que a
sustenta, a ação proposta, o impacto esperado e a limitação que a acompanha.

**Decisão da equipe:** reescrevemos todas as recomendações nesse formato. A
seção de limitações inclui pontos que não apareceriam sozinhos: a prevalência
de lotação sobe de 15% para 17% porque a amostra exclui as últimas paradas de
cada viagem, e a câmera é tratada como leitura perfeita, o que transforma o
ganho medido em limite superior e não em promessa.

---

### 14 — Indicador de impacto

**Prompt:** "Como definir um indicador de impacto que meça benefício ao cidadão
e não desempenho interno do modelo?"

**Objetivo:** atender ao requisito de indicador quantificável da Ficha da
CitizenTech.

**Resultado:** proposta de medir a proporção de viagens críticas em que o
sistema emitiu alerta com pelo menos 5 paradas de antecedência.

**Decisão da equipe:** adotamos esse indicador porque ele mede o que muda na
vida do passageiro — chegar ao ponto a tempo de decidir — em vez de acurácia
interna. A meta inicial é cobrir 80% das viagens críticas, que hoje são 34,9%
das 2.700 viagens da base.
---

## 2. O que a IA fez e o que a equipe fez

**A IA apoiou em:** estruturação da base simulada, escrita do código de
geração, implementação dos modelos, cálculo das métricas, sugestão de
visualizações e revisão do texto das recomendações.

**A equipe fez:** definiu o problema e o cidadão beneficiado; escolheu o limiar
de lotação; decidiu o horizonte de previsão e o threshold da logística;
executou e conferiu cada célula contra os números esperados; identificou e
corrigiu o vazamento na divisão treino/teste; e reescreveu as recomendações
com os números reais dos resultados.

**Nenhum integrante submeteu código que não saiba explicar.**

---

## 3. Decisões técnicas e suas justificativas

**Por que 80% de ocupação define "lotado"?**
Porque é a partir desse ponto que o passageiro deixa de ter assento garantido.
O limiar é uma decisão de produto, não um valor estatístico ótimo.

**Por que horizonte de 5 paradas?**
Porque representa cerca de um terço do itinerário, que é a distância em que o
cidadão ainda tem decisão a tomar. Com horizonte de 1 parada o modelo acerta
quase tudo, mas o cidadão já está vendo o ônibus; com horizonte muito longo o
erro cresce e a informação perde utilidade.

**Por que a divisão entre treino e teste é temporal e não aleatória?**
Porque o uso real é prever o futuro a partir do passado. Divisão aleatória
colocaria dados do futuro no treino e inflaria as métricas.

**Por que existe um baseline?**
Sem referência, um R² de 0,69 não significa nada. O baseline mostra o que um
sistema sem modelo conseguiria, e a diferença entre os dois é o que justifica o
trabalho.

**Por que priorizar recall em vez de acurácia?**
Porque os erros têm custos diferentes para o cidadão. Deixar de avisar sobre um
ônibus lotado é pior do que avisar incorretamente sobre um ônibus vazio.

**Por que R² e AUC dão valores tão diferentes (0,86 e 0,98)?**
Porque medem coisas distintas. O R² avalia a magnitude do erro na ocupação; o
AUC avalia apenas a capacidade de ordenar casos acima e abaixo do limiar.
Classificar é mais fácil que estimar.

---

## 4. Erros encontrados e corrigidos

**Vazamento na divisão treino/teste.**
A primeira versão dividia a base pelo horário de cada registro. Uma viagem que
começou antes da meia-noite e terminou depois ficou com parte dos registros no
treino e parte no teste — mesma viagem nos dois conjuntos. A verificação
automática detectou o problema. Correção: cada viagem pertence ao dia em que
começou, e a divisão é feita por esse dia.

**Modelo pior que o baseline na primeira versão.**
A regressão linear inicial perdeu para a média histórica. Causa: o modelo não
recebia a linha e representava a hora com uma única harmônica, insuficiente
para dois picos estreitos. Correção: linha e hora como variáveis categóricas e
termo quadrático para a posição no trajeto.

**Resultado bom demais para ser verdade.**
O modelo com câmera chegou a R² de 0,97 prevendo a parada seguinte. A
verificação confirmou que não havia vazamento: o problema era o horizonte curto
demais. Correção: horizonte de 5 paradas, que representa o uso real.

**Base artificialmente regular.**
A primeira simulação usava as mesmas curvas de embarque e desembarque em todas
as viagens, o que permitia reconstruir o percurso inteiro a partir de um único
ponto observado. Correção: cada viagem passou a ter seu próprio perfil por
parada, reproduzindo a irregularidade de campo.

---

## 5. Declaração

Declaramos que o uso de IA neste projeto foi feito como apoio técnico e
pedagógico, com registro integral dos prompts utilizados, conforme as regras da
Global Solution 2026.2. Todo o código entregue foi executado, verificado e é de
responsabilidade da equipe.