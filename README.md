# Disciplina de Inteligência Artificial — UniCesumar

**Professor:** Munif Gebara

**Integrantes:**
- Carlos Eduardo Souza Favarão — 23034356-2
- Heloísa Tognolli Scarante — 23211463-2

---

## Contextualização

Os jogos competitivos online geram uma grande quantidade de dados sobre o desempenho dos jogadores. Em jogos como Valorant, informações como eliminações, mortes, assistências, dano causado, dano recebido, headshots e ranking competitivo permitem identificar padrões de comportamento e diferentes estilos de jogo. A análise desses dados por meio de técnicas de Inteligência Artificial oferece uma oportunidade de segmentar automaticamente os perfis de jogadores, gerando insights úteis para equipes, coaches e plataformas de esportes eletrônicos.

---

## Problematização e Objetivo

O objetivo do trabalho é identificar perfis distintos de jogadores com base em suas estatísticas de desempenho, utilizando técnicas de agrupamento e classificação. A hipótese central é que as métricas coletadas permitem separar automaticamente jogadores em grupos coesos — como perfis mais competitivos, casuais ou intermediários — e que esses grupos podem ser reproduzidos por um classificador supervisionado com alta acurácia.

---

## Dataset

Foi utilizado o **Valorant Dataset V3**, disponível na plataforma Kaggle (`notnguyen/valorant-dataset-v3`). O dataset contém estatísticas de desempenho de múltiplos jogadores coletadas ao longo de diversas partidas competitivas.

---

## Principais Atributos

| Atributo | Descrição |
|---|---|
| kills | Total de eliminações |
| deaths | Total de mortes |
| assists | Total de assistências |
| damage | Dano total causado |
| damage_received | Dano total recebido |
| headshots | Total de headshots |
| traded | Mortes trocadas com aliados |
| matches | Número de partidas jogadas |
| tier | Rank competitivo do jogador |

---

## Tratamento dos Dados

O pipeline de preparação dos dados seguiu as etapas abaixo:

- Remoção da coluna `player` (identificador sem valor preditivo para o modelo).
- Conversão das colunas numéricas em formato de texto com separadores de milhar (ex: `"1,234"` → `1234`).
- Eliminação de linhas com valores nulos após a conversão.
- Label Encoding da coluna `tier`, transformando os ranks categóricos em valores numéricos ordinais.
- Substituição de zeros por 1 nas colunas `kills`, `deaths` e `matches`, evitando divisão por zero.

Foram criados novos atributos derivados das estatísticas originais:

| Atributo | Fórmula | Descrição |
|---|---|---|
| kd_ratio | kills / deaths | Eficiência em eliminações |
| headshot_ratio | headshots / kills | Precisão nos abates |
| damage_per_match | damage / matches | Produção ofensiva média por partida |
| assist_per_match | assists / matches | Suporte médio por partida |
| traded_ratio | traded / matches | Taxa de mortes trocadas por partida |

---

## Normalização

Todos os atributos foram padronizados com `StandardScaler` (média 0, desvio padrão 1). Essa etapa é obrigatória para algoritmos baseados em distância como K-Means e KNN, pois garante que nenhuma variável com escala maior domine artificialmente o cálculo de proximidade.

---

## Divisão Treino/Teste

Para os modelos supervisionados, a base foi dividida em **80% treino** e **20% teste**, com `stratify=y` para garantir proporção igual de classes em ambos os conjuntos.

---

## Métodos de IA utilizados

### K-Means (Aprendizado Não Supervisionado)

O K-Means é um algoritmo de clusterização que agrupa os dados sem utilizar rótulos previamente definidos. O algoritmo posiciona K centroides no espaço de features e, iterativamente, atribui cada ponto ao centroide mais próximo e recalcula a posição dos centroides como a média dos pontos de cada grupo, repetindo até convergir.

Para determinar o número ideal de clusters, foram utilizados dois métodos:

- **Método do Cotovelo:** plota a inércia para K de 1 a 10. O ponto de inflexão indica onde aumentar K deixa de trazer ganho expressivo.
- **Silhouette Score:** mede a coesão interna e a separação entre clusters. Varia de -1 a 1; o K com maior score foi selecionado automaticamente.

O K-Means identificou **3 clusters**, interpretados como: **Veteranos** (alta KD ratio e dano), **Competitivos** (desempenho intermediário consistente) e **Casuais** (menor volume de partidas e métricas mais baixas).

### K-Nearest Neighbors (KNN) — Aprendizado Supervisionado

Após a clusterização, os rótulos gerados pelo K-Means foram utilizados como variável alvo para o treinamento do KNN. O algoritmo classifica um novo ponto com base nos K vizinhos mais próximos no espaço de features: a classe mais frequente entre esses vizinhos é atribuída ao ponto. O KNN não constrói um modelo explícito — ele memoriza os dados de treino e realiza o cálculo de distâncias no momento da predição. O valor de K foi testado de 1 a 20 vizinhos, medindo a acurácia em cada caso. O K com maior acurácia foi selecionado automaticamente.

### Árvore de Decisão — Comparação

A Árvore de Decisão foi utilizada como modelo comparativo ao KNN. O algoritmo aprende regras hierárquicas dividindo os dados recursivamente com base na feature e no limiar que melhor separa as classes (critério Gini). O parâmetro `max_depth=5` foi definido para evitar overfitting. Uma vantagem importante da Árvore de Decisão é a interpretabilidade: ela gera um gráfico de importância das variáveis, revelando quais features mais influenciaram as decisões do modelo.

---

## Avaliação dos Modelos

### K-Means

- Método do Cotovelo — identificação visual do número ideal de clusters.
- Silhouette Score — medida quantitativa da qualidade dos agrupamentos.
- Visualização PCA — redução para 2 componentes principais para plotagem dos clusters em 2D.

### KNN e Árvore de Decisão

Ambos os modelos foram avaliados com as seguintes métricas:

| Métrica | O que mede |
|---|---|
| Acurácia | Proporção de classificações corretas sobre o total |
| Precisão | Dos classificados como X, quantos realmente são X |
| Recall (Revocação) | Dos que realmente são X, quantos foram classificados corretamente |
| F1-Score | Média harmônica entre Precisão e Recall |
| Matriz de Confusão | Distribuição de acertos e erros por classe |

<img width="1189" height="490" alt="image" src="https://github.com/user-attachments/assets/a3d9c9e6-2427-4b9e-823e-c4b761780eea" />
<img width="845" height="547" alt="image" src="https://github.com/user-attachments/assets/3a837f92-5b66-49fd-beef-3049ad39d299" />
<img width="863" height="471" alt="image" src="https://github.com/user-attachments/assets/ec6cc55a-a87f-42f2-960e-87c16ee2ae23" />
<img width="507" height="455" alt="image" src="https://github.com/user-attachments/assets/a9f4fbca-7d61-4f0d-a38b-f46c0ad93ef0" />
<img width="855" height="473" alt="image" src="https://github.com/user-attachments/assets/4c980ec6-3a04-422b-80f8-d05633551aa1" />
<img width="507" height="455" alt="image" src="https://github.com/user-attachments/assets/5ae1bdf2-8989-4537-b0b8-3026a45acacc" />
<img width="989" height="489" alt="image" src="https://github.com/user-attachments/assets/314a9b13-f196-4ce0-bddc-4a5bf87a36d1" />
<img width="989" height="590" alt="image" src="https://github.com/user-attachments/assets/043e18b3-d0b4-458e-aa98-6f3205d4d14c" />

---

## Comparação dos Resultados

| Critério | KNN | Árvore |
|---|---|---|
| Interpretabilidade | Baixa | Alta (regras visíveis) |
| Velocidade de predição | Lenta (calcula distâncias) | Rápida |
| Sensível à normalização | Sim (obrigatório) | Não |
| Importância de features | Não disponível | Disponível |
| Risco de overfitting | Controlado pelo K | Controlado pelo max_depth |
| Acurácia | 0.9759 | 0.9425 |

---

## Conclusão

O projeto demonstrou que é possível identificar perfis distintos de jogadores de Valorant de forma automática utilizando técnicas de Inteligência Artificial não supervisionada e supervisionada de maneira complementar. O algoritmo K-Means identificou três grupos coesos de jogadores com características estatísticas bem diferenciadas. A técnica de PCA permitiu visualizar graficamente a separação entre esses grupos no espaço bidimensional. O KNN e a Árvore de Decisão, treinados sobre os rótulos gerados pelo K-Means, demonstraram alta capacidade de reproduzir os padrões identificados na clusterização.

A Árvore de Decisão oferece vantagem interpretativa ao revelar quais features são mais relevantes para a classificação dos perfis. O KNN, por sua vez, é mais simples de implementar e robusto para datasets de tamanho moderado. Como trabalhos futuros, sugere-se explorar algoritmos como Random Forest ou Gradient Boosting para comparação adicional, bem como aumentar a base de dados e testar diferentes configurações de feature engineering.
