# ProjetoFinalBIMaster
Repositório dedicado a exposição do projeto final do curso BI Master

# Detecção Automatizada de Vazamento de Óleo em Geradores de Turbina a Gás Utilizando Aprendizado de Máquina

#### Aluno: Davi Oliveira Francisco
#### Orientadora: Leonardo Mendoza

---

Trabalho apresentado ao curso [BI MASTER](https://ica.puc-rio.ai/bi-master) como pré-requisito para conclusão de curso e obtenção de crédito na disciplina "Projetos de Sistemas Inteligentes de Apoio à Decisão".

- [Link para o código](https://github.com/dofrancisco/ProjetoFinalBIMaster).

---

### Resumo

No contexto de FPSOs (Floating Production Storage and Offloading), as GTGs (Gas Turbine Generators) são essenciais para garantir o fornecimento contínuo de energia elétrica, viabilizando as operações de extração, processamento e armazenamento de óleo em ambiente offshore. Devido à criticidade desses equipamentos, é imprescindível manter alta disponibilidade e confiabilidade, reduzindo ao máximo o tempo de inatividade não planejado. Um dos principais desafios de manutenção preditiva nessas turbinas é o monitoramento do aumento do consumo de óleo lubrificante, que pode indicar falhas incipientes, como vazamentos ou desgaste de componentes internos. O presente trabalho propõe o desenvolvimento de features e modelos de aprendizado de máquina para detecção automática de padrões anômalos no consumo de óleo das GTGs, possibilitando intervenções proativas e contribuindo para a integridade operacional, redução de custos e mitigação de riscos associados a falhas graves nesses sistemas críticos.

### Abstract

In the context of FPSOs (Floating Production Storage and Offloading), Gas Turbine Generators (GTGs) are critical assets for ensuring continuous power supply, enabling offshore oil extraction, processing, and storage operations. Due to the high operational demands and the criticality of these systems, maintaining their reliability and availability is essential to minimize unplanned downtime. One of the main challenges in predictive maintenance of GTGs is monitoring the increase in lubricating oil consumption, which may indicate incipient failures such as leaks or internal component wear. This work proposes the development of features and machine learning models for the automatic detection of anomalous patterns in GTG oil consumption, enabling proactive interventions and contributing to operational integrity, cost reduction, and risk mitigation associated with severe failures in these critical systems.

### 1. Introdução

O setor de óleo e gás offshore é caracterizado por operações complexas e de alta criticidade, onde a confiabilidade dos equipamentos é fundamental para garantir a segurança, a eficiência e a continuidade dos processos produtivos. Entre os sistemas essenciais presentes em plataformas do tipo FPSO (Floating Production Storage and Offloading), destacam-se os Geradores de Turbina a Gás (GTGs), responsáveis pelo fornecimento de energia elétrica para todas as operações a bordo. A ocorrência de falhas nesses equipamentos pode resultar em paradas não planejadas, prejuízos financeiros e riscos ambientais significativos.

Diante desse cenário, a manutenção preditiva baseada em dados tem se mostrado uma estratégia promissora para antecipar falhas e otimizar a gestão dos ativos. Em especial, o monitoramento do consumo de óleo lubrificante nas GTGs surge como um indicador relevante para a detecção precoce de vazamentos e desgastes internos, possibilitando intervenções corretivas antes que ocorram danos mais graves. No entanto, a identificação automática de padrões anômalos nesse contexto ainda representa um desafio, exigindo o desenvolvimento de soluções inteligentes capazes de lidar com a complexidade e a variabilidade dos dados operacionais.

Neste trabalho, propõe-se o desenvolvimento de um modelo de aprendizado de máquina voltado para a detecção de vazamentos de óleo em GTGs, **utilizando dados simulados de operação e consumo de óleo de máquinas fictícias**. O objetivo é criar um sistema capaz de identificar, de forma autônoma, desvios no comportamento esperado do consumo, fornecendo alertas para a equipe de manutenção. Para isso, serão exploradas técnicas de engenharia de features, análise estatística e algoritmos de detecção de anomalias, buscando maximizar a sensibilidade do modelo sem comprometer sua robustez frente a variações normais do processo.

### 2. Modelagem

Com o objetivo de detectar vazamentos nesses equipamentos é necessário criar features para que possamos identificar aumento ou qualquer alteração no consumo de óleo. Nesse contexto, foram desenvolvidas algumas funções específicas para calcular features que podem ajudar na detecção dessas alterações.

Algumas funções chave da etapa de *feature engineering* são:

- remove_transition_periods
- create_features_refined
- detect_changepoint
- regressao_intervalos_level

**Detalhando a geração de features para detecção de vazamentos**
  
O principal indicador criado foi o **daily_consumption**, que representa o consumo diário estimado a partir das variações do nível de óleo, já filtrando ruídos e reabastecimentos. Essa feature de daily_consumption é fundamental para o modelo e ajuda na interpretabilidade das features geradas.
  
Para segmentar o comportamento do consumo, utilizamos dois métodos de agrupamento:
  
1. **Auto Group Values**: O primeiro método identifica saltos abruptos no nível de óleo (tipicamente associados a reabastecimentos) e divide a série temporal em grupos automáticos entre esses eventos. Para cada grupo, é calculada a média do consumo diário, resultando na feature **auto_group_values**. Essa abordagem captura tendências de consumo em períodos mais longos, entre reabastecimentos.
  
2. **Changepoint**: O segundo método aplica detecção de pontos de mudança (changepoints) dentro de cada grupo automático, utilizando a biblioteca ruptures. Isso permite identificar alterações mais sutis ou repentinas no padrão de consumo que podem ocorrer mesmo entre dois reabastecimentos, onde o risco de vazamento pode aumentar rapidamente.
  
Por fim, foi criada a feature **composed_risk**, que combina a média do consumo dos grupos automáticos (**auto_group_values**) com as variações detectadas pelos changepoints. O objetivo é obter um indicador de risco mais robusto: enquanto o auto_group_values reflete variações de longo prazo, o changepoint enfatiza mudanças agudas que podem sinalizar o início ou agravamento de um vazamento entre reabastecimentos. O composed_risk será utilizado como principal métrica para detecção de vazamentos ao longo do tempo.
  
**Features de Regressão Linear por Intervalos**
  
Além das features acima, também foi implementada a geração de features baseadas em regressão linear por intervalos, por meio da função `regressao_intervalos_level`. Essa função segmenta a série temporal de nível de óleo em intervalos delimitados por eventos de reabastecimento (ou saltos abruptos) e, para cada intervalo, ajusta uma regressão linear. O coeficiente angular (slope) dessa regressão representa a inclinação média do consumo naquele período, enquanto o erro da regressão indica o quanto o comportamento real se desvia de uma tendência linear simples.
  
Essas features de regressão têm o objetivo de replicar, de forma quantitativa, a análise visual **frequentemente realizada** por engenheiros, que comparam as inclinações dos trechos entre reabastecimentos para identificar padrões anômalos de consumo. Assim, tanto o valor da regressão quanto o erro associado podem ser utilizados (ou ao menos testados) em modelos de detecção, enriquecendo o conjunto de variáveis e permitindo capturar mudanças de comportamento que não seriam evidentes apenas pelas médias dos grupos ou pelos changepoints.

### 3. Resultados

Nesta seção, são apresentados os principais resultados obtidos a partir da aplicação do pipeline de processamento e modelagem desenvolvido para detecção de anomalias no consumo de óleo lubrificante das GTGs em FPSOs.

Após a preparação dos dados e a engenharia de features, foi possível calcular o consumo diário de óleo para cada turbina, identificando períodos de operação normal e potenciais desvios. A análise exploratória revelou que, em condições normais, o consumo apresenta baixa variabilidade, com médias diárias próximas de 1-2%, exceto em momentos de reabastecimento ou paradas programadas.

A análise de resultados mostra que a conbinação correta de features e modelo pode fornecer insights valiosos para a rotina de um engenheiro de confiabilidade. Os modelos 1 (IsolationForest) e 3 (OCSVM) desenvolvidos demonstraram clara capacidade de geração de valor no contexto de manutenção preditiva e unidos ao sistema de alarmes construído neste trabalho podem ajudar o engenheiro a se dedicar a outras atividades confiando o trabalho de detecção de vazamentos em turbinas para o sistema desenvolvido. 

Dessa forma, o treinamento em um conjunto único de dados garante a escalabilidade ao passo que um único modelo pode ser aplicado a vários ativos. Além disso, a segmentação dos dados por grupos automáticos (separando os períodos de operação) contribuiu para a redução de falsos positivos, tornando a detecção mais sensível a desvios relevantes dentro de cada contexto operacional.

### 4. Conclusões

Os experimentos realizados demonstraram que a abordagem baseada em engenharia de features e detecção automática de anomalias treinado em um único dataset é eficaz para monitorar o consumo de óleo lubrificante em GTGs de FPSOs. O modelo desenvolvido foi capaz de identificar padrões anômalos de consumo com antecedência, possibilitando intervenções proativas e contribuindo para a redução do tempo de inatividade não planejado.

Como trabalhos futuros seria interessante aplicar ao modelo em questão mais features para estudar o impacto na capacidade de detecção e outros modelos mais modernos e complexos (como AutoEncoders e redes MLP, por exemplo). Outro ponto extremamente interessante a ser explorados seria a implementação de diagnóstico usando LLMs e análises de outros atributos do equipamento como carga na turbina e outros fatores que podem afetar o consumo de óleo nesses equipamentos que podem resultar em falso positivos em produção.

Em resumo, os resultados obtidos reforçam o potencial do uso de técnicas de machine learning para aprimorar a manutenção preditiva em sistemas críticos da indústria offshore, promovendo maior segurança, eficiência e sustentabilidade nas operações.

---

Matrícula: 231101072

Pontifícia Universidade Católica do Rio de Janeiro

Curso de Pós Graduação *Business Intelligence Master*
