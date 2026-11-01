# Plano de Trabalho — Deliverable 1: Exploratory Data Analysis (EDA)
**Projeto:** Amazing Creative Desk Coworking (ACDC) — Data Mining (NOVA IMS)  
**Data de Entrega:** 4 de Novembro de 2026  
**Entregáveis Obrigatórios:**
1. **`GroupXX_EDA_Code.ipynb`** (5 pontos) — Notebook Jupyter limpo, documentado, organizado por secções Markdown e executado do início ao fim sem erros.
2. **`GroupXX_EDA_Slides.pdf`** (15 pontos) — Apresentação executiva com **3 a 5 slides de conteúdo** (excluindo o slide de capa), focada em *insights*, interpretações de negócio e preparação para o *clustering* (metodologia CRISP-DM).

---

## Visão Geral do Estado Atual do Repositório
O notebook [`notebooks/EDA.ipynb`](notebooks/EDA.ipynb) já tem uma boa base inicial até à secção 4:
- **Feito:** Dicionário de dados (`dictionary_rows`), inspeção inicial de tipos, correção de tipos incongruentes (`uses_print`, `guest_passes_issued`, `lifetime_value`, parsing de datas em formato misto ISO e `DD/MM/YYYY`), remoção de duplicados exatos e *canonical mapping* de variáveis categóricas (`gender`, `country`, `industry`, `plan_tier`).
- **Por fazer / organizar:** Limpeza da zona de rascunho (*draft material* a partir da célula 20), auditoria profunda de qualidade e consistência lógica dos dados, análise estatística e visual (univariada, bivariada e multivariada sob as perspetivas *Value-based* e *Behavioral*), criação e justificação de novas variáveis (*Feature Engineering*), síntese das 4 perguntas-chave do enunciado e criação dos *Discussion Slides*.

---

## Passo a Passo Explícito de Execução

### PASSO 1: Auditoria de Qualidade e Consistência Lógica dos Dados (Data Quality & Integrity)
**Objetivo:** Avaliar os problemas de qualidade dos dados e explicar como podem distorcer algoritmos de *clustering* (baseados em distâncias/densidade) e como serão mitigados.

1. **Auditoria de Valores em Falta (*Missingness*):**
   - Quantificar e visualizar padrões de *missing values* (ex.: `meeting_room_hours`, `guest_passes_issued`, `lifetime_value`, `age`, `gender`, `industry`).
   - Testar se a ausência de dados é aleatória ou estrutural (ex.: cruzar `uses_meeting_rooms == False` com `meeting_room_hours` nulo; cruzar `uses_guest_passes == False` com `guest_passes_issued` nulo).
   - Definir a estratégia de tratamento para a fase de *clustering* (quando imputar com `0` por representar não-utilização vs. imputação condicional/mediana vs. preservar categoria `Unknown`).
2. **Validação de Domínio, Datas Sentinela e Valores Implausíveis:**
   - **Idades (`age`):** Identificar valores negativos, menores de 18 anos ou > 100 anos (ou anos de nascimento introduzidos por engano no campo de idade).
   - **Datas (`membership_start`, `membership_end`, `last_badge_swipe`):**
     - Investigar a data sentinela `1970-01-01` em `membership_end` (representa membros ativos sem data de fim ou erro de registo?).
     - Verificar consistência temporal: `membership_start <= last_badge_swipe` e `membership_start <= membership_end`.
     - Auditar `days_since_last_visit` face a `last_badge_swipe` para verificar se a data de extração/referência é consistente em todo o dataset.
   - **Regras de Negócio e Consistência Cruzada (*Cross-Column Checks*):**
     - `is_team_account` vs. `n_colleagues_on_account` (ex.: contas individuais com colegas > 0 ou contas de equipa com 0 colegas).
     - `plan_days_per_week` (valores fora de `[1, 7]` ou incompatíveis com o `plan_tier`, ex.: `full_time` / `dedicated_desk`).
     - Consistência entre flags booleanas `uses_*` e as respetivas variáveis quantitativas (`meeting_room_hours`, `event_space_hours`, `guest_passes_issued`).
     - Verificar se `actual_visits_last_period > entitled_visits_last_period` (sinal comportamental de *over-utilization* e potencial *upsell*, não um erro!).

---

### PASSO 2: Estatística Descritiva e Exploração Visual (Univariada, Bivariada e Multivariada)
**Objetivo:** Destacar distribuições, tendências, assimetrias (*skewness*), *outliers* e identificar quais as variáveis mais relevantes para as perspetivas de segmentação da ACDC.

1. **Análise Demográfica e de Conta (Contexto / Perfil):**
   - Distribuição de `age`, `gender`, `country`, `industry`.
   - Proporção de contas individuais vs. empresa (`is_team_account`) e distribuição do tamanho das equipas (`n_colleagues_on_account`).
2. **Perspetiva de Valor (*Value-Based Perspective*):**
   - Análise de `lifetime_value`, `plan_tier`, `n_renewals` e `plan_days_per_week`.
   - Cruzar `lifetime_value` com `plan_tier`, `n_renewals` e `is_team_account` (boxplots/violin plots e estatísticas robustas: mediana e IQR devido à assimetria).
3. **Perspetiva Comportamental e de Utilização (*Behavioral & Usage Perspective*):**
   - Frequência e recência de utilização: `total_visits`, `avg_weekly_visits`, `days_since_last_visit`, `actual_visits_last_period` vs. `entitled_visits_last_period`.
   - Consumo de *add-ons* e espaços: `meeting_room_hours`, `event_space_hours`, `guest_passes_issued` e as 8 flags `uses_*` (`uses_meeting_rooms`, `uses_event_space`, `uses_guest_passes`, `uses_print`, `uses_mail_handling`, `uses_phone_booths`, `uses_day_passes`, `uses_storage`, `uses_bike_locker`).
   - Identificar padrões de co-ocorrência de *add-ons* (ex.: quem usa salas de reunião também usa *guest passes* e *phone booths*?).
4. **Análise de Correlação e Redundância:**
   - Matriz de correlação (Spearman e Pearson) para variáveis numéricas e associação (Cramér's V / Point-Biserial) para variáveis categóricas/binárias.
   - Sinalizar variáveis colineares que podem ter peso duplicado no *clustering* (ex.: `total_visits` vs. antiguidade + `avg_weekly_visits`).

---

### PASSO 3: Criação e Justificação de Novas Variáveis (*Feature Engineering*)
**Objetivo:** Desenvolver variáveis derivadas que capturem aspetos mais ricos do comportamento e valor do cliente, justificando o seu contributo para os modelos de *clustering*.

1. **Antiguidade e Retenção (*Tenure & Loyalty*):**
   - `tenure_days` / `tenure_weeks`: Duração da subscrição (calculada entre `membership_start` e `membership_end` ou data de referência para membros ativos).
   - `is_active_member`: Indicador binário derivado de `membership_end` (distinguindo quem já saiu de quem permanece ativo).
   - `renewal_rate` (ou renovações por ano de antiguidade): Normaliza `n_renewals` pelo tempo de casa.
2. **Intensidade e Eficiência de Utilização (*Usage Intensity & Plan Fit*):**
   - `last_period_utilization_ratio`: `actual_visits_last_period / entitled_visits_last_period` (identifica *under-users* em risco de *churn* vs. *over-users* candidatos a *upgrade* de plano).
   - `visit_consistency_ratio`: Rácio entre a frequência recente e a frequência histórica (`avg_weekly_visits`), detetando desaceleração de uso (*churn signal*).
3. **Adesão a Serviços Extra (*Add-on Breadth & Intensity*):**
   - `addon_count` (ou `addon_breadth_score`): Soma das flags binárias de serviços extra utilizados pelo membro (de 0 a 9), medindo a dependência do ecossistema ACDC.
   - `collab_intensity`: Uso combinado de recursos colaborativos (`meeting_room_hours`, `event_space_hours`, `guest_passes_issued`) por visita ou por semana.
4. **Métricas Económicas Normalizadas (*Normalized Value Metrics*):**
   - `value_per_visit`: `lifetime_value / total_visits` (custo/receita efetiva por deslocação ao espaço).
   - `monthly_revenue_rate`: `lifetime_value / tenure_months` (valor recorrente mensal independentemente da data de entrada).
   - `revenue_per_seat`: `lifetime_value / (1 + n_colleagues_on_account)` para comparar de forma justa contas individuais e contas de equipa.
5. **Validação Visual das Novas Features:**
   - Mostrar gráficos e tabelas que comprovem como estas variáveis separam melhor os grupos de clientes do que as variáveis brutas isoladas.

---

### PASSO 4: Sinais Preliminares de Segmentação e Resposta às 4 Perguntas do Enunciado
**Objetivo:** Sintetizar no final do notebook uma secção clara de conclusões de negócio ligadas ao CRISP-DM e às 4 perguntas obrigatórias:

1. **Descobertas mais inesperadas ou reveladoras (*Unexpected/Insightful Findings*):**
   - Que padrões surpreendentes surgiram (ex.: desalinhamento entre `plan_tier` e utilização real, contas de equipa vs. individuais, perfis de utilização de *add-ons*) e o que revelam sobre potenciais *clusters*?
2. **Limitações dos dados e riscos para o *clustering* (*Data Limitations & Mitigation*):**
   - Assimetria extrema (*skewness*), *outliers*, variáveis dependentes do tempo de exposição (*tenure bias*), mistura de escalas/tipos de dados, datas sentinela e *missing values*. Como mitigar (transformações log/Yeo-Johnson, *scaling* robusto, separação de perspetivas, imputação informada).
3. **Padrões de atividade e hipóteses de *clusters* naturais (*Anticipated Customer Clusters*):**
   - Definir as perspetivas de segmentação recomendadas:
     - **Perspetiva 1 — Value / Commercial:** (ex.: `monthly_revenue_rate`, `lifetime_value`, `plan_tier`, `n_renewals`, `is_team_account`).
     - **Perspetiva 2 — Behavioral / Space Usage:** (ex.: `avg_weekly_visits`, `last_period_utilization_ratio`, `days_since_last_visit`, `addon_count`, `meeting_room_hours`).
   - Descrever os arquétipos preliminares esperados (ex.: *Corporate Anchors / Team Heavy Users*, *Dedicated Solo Regulars*, *Flexible Collaborators / Event & Meeting Users*, *Dormant / At-Risk Under-users*, *Cost-Conscious Basic Desk Users*).
4. **Comunicação para *Stakeholders* Não-Técnicos:**
   - Resumo executivo do número esperado de segmentos (ex.: 4 a 6 segmentos), as variáveis diferenciadoras mais importantes e os desafios práticos antecipados.

---

### PASSO 5: Opcional — Componente de Bónus (+10% a +20%)
**Objetivo:** Garantir pontuação extra sem comprometer o núcleo obrigatório.
- **Opção Recomendada (Mais rápida e 100% alinhada com o Notebook): Opção 2 — *Multidimensional Outlier & Anomaly Detection* (+10%)**
  - Aplicar **Isolation Forest** e/ou **Local Outlier Factor (LOF)** / **Distância de Mahalanobis** sobre as variáveis numéricas limpas.
  - Identificar perfis de clientes anómalos (ex.: elevado LTV com zero visitas, ou consumo extremo de salas de reunião/eventos).
  - Discutir explicitamente se estas anomalias devem ser excluídas, ter peso reduzido ou constituir um micro-segmento VIP/especial antes do *K-Means / Hierarchical Clustering*.
- *(Opcional Secundário)* **Opção 3 — *Interactive EDA Dashboard* (+10%):** Criar uma app leve em `Streamlit` ou `Plotly` se sobrar tempo no dia 3 de novembro.

---

### PASSO 6: Limpeza Final do Notebook (`GroupXX_EDA_Code.ipynb`)
1. Remover todo o código redundante da secção de rascunho (*draft material* após a célula 20) integrando o que for útil nas secções certas.
2. Garantir numeração consistente de secções em Markdown (1. Business & Metadata Understanding, 2. Data Quality & Cleaning, 3. Univariate & Bivariate Exploration, 4. Feature Engineering, 5. Preliminary Segmentation Signals & Key Takeaways).
3. Executar **"Restart Kernel and Run All Cells"** no ambiente `.venv` para garantir zero erros e todos os gráficos visíveis.
4. Guardar uma cópia com o nome oficial de entrega: `GroupXX_EDA_Code.ipynb`.

---

### PASSO 7: Construção dos Discussion Slides (`GroupXX_EDA_Slides.pdf` — 15 Pontos!)
> **Nota Crítica:** Os slides valem **75% da nota desta entrega (15 em 20 pontos)** e têm um limite rigoroso de **mínimo 3, máximo 5 slides de conteúdo** (excluindo a capa). Não devem ser uma lista de descrições de colunas, mas sim **insights estratégicos e decisões metodológicas**.

**Estrutura Recomendada para os 5 Slides de Conteúdo:**
- **Slide 0 (Capa):** Título do Projeto, Grupo XX, Nomes e IDs dos estudantes, Data.
- **Slide 1 — Contexto de Negócio & Diagnóstico de Qualidade dos Dados (Questão 2):**
  - Resumo do dataset ACDC, principais problemas críticos detetados (datas sentinela, *missingness* estrutural vs. real, *outliers*, *exposure bias*) e como foram mitigados para garantir um *clustering* fiável.
- **Slide 2 — Principais Insights & Descobertas Inesperadas (Questão 1):**
  - 2 a 3 visualizações fortes que mostrem comportamentos reveladores (ex.: utilização vs. escalão do plano, diferença entre contas individuais e de equipa, concentração de receita vs. frequência de visitas).
- **Slide 3 — Feature Engineering & Perspetivas de Segmentação (Questão 3 - Parte 1):**
  - Apresentação das novas variáveis criadas (*Utilization Ratio*, *Monthly Revenue Rate*, *Add-on Breadth*, *Tenure*) e como estruturam as perspetivas **Value-Based** e **Behavioral**.
- **Slide 4 — Hipóteses de Segmentos Naturais / Arquétipos de Clientes (Questão 3 - Parte 2):**
  - Perfis preliminares de clientes observados nos dados (4–5 grupos naturais antecipados) suportados por evidência visual das features criadas.
- **Slide 5 — Roadmap para Clustering & Mensagem Executiva para a ACDC (Questão 4):**
  - Como comunicar os segmentos aos *stakeholders* da ACDC, número esperado de clusters, variáveis diferenciadoras chave, desafios técnicos antecipados para a Fase 2 e próximos passos.

---

## Cronograma Sugerido (1 a 4 de Novembro)

| Dia | Foco Principal | Entregável Interno |
| :--- | :--- | :--- |
| **Dia 1 (Hoje, 1 Nov)** | Passo 1 (Auditoria completa de qualidade e limpeza) + Passo 2 (Exploração visual univariada e bivariada) | Secções 1, 2 e 3 do `EDA.ipynb` fechadas |
| **Dia 2 (2 Nov)** | Passo 3 (Feature Engineering) + Passo 4 (Síntese das 4 perguntas) + Passo 5 (Outlier Detection Multidimensional) | Notebook `EDA.ipynb` 100% completo e testado |
| **Dia 3 (3 Nov)** | Passo 7 (Criação dos 5 Discussion Slides com os melhores gráficos do notebook) | `GroupXX_EDA_Slides.pdf` pronto |
| **Dia 4 (4 Nov)** | Revisão final, verificação das regras de submissão e entrega no Moodle | Submissão final (`GroupXX_EDA_Code.ipynb` + `GroupXX_EDA_Slides.pdf`) |

