# 🧠 Projeto de Ciência de Dados ITA 2025.02  
### Aprimoramento de Vídeos para Processamento iPPG (GREEN, CHROM e POS)

---

## 🎯 Objetivo do Projeto  

Demonstrar que um modelo de Ciência de Dados pode **aprimorar automaticamente a qualidade de vídeos** para o processamento de sinais **iPPG**, tornando a extração de informações fisiológicas mais robusta — independentemente do método de estimativa utilizado (**GREEN**, **CHROM** ou **POS**).  

Em outras palavras, o projeto mostra que um **pipeline inteligente de pré-processamento**, baseado em aprendizado de máquina, pode:  
- 🕵️‍♂️ **Identificar** quando o sinal óptico está degradado (por variações de luz, ruído de compressão ou baixa qualidade da captura);  
- 🔧 **Corrigir** dinamicamente a iluminação e a estabilidade fotométrica do vídeo;  
- 💪 **Melhorar** a performance global dos métodos tradicionais de rPPG, reduzindo a dependência das condições ideais de gravação.  

Link do videos origem:
---

## 🧩 Visão em Etapas  

### **1️⃣ Extração dos Sinais Brutos**  
- Gerar séries temporais **R**, **G** e **B** da região da testa (ROI).  
- Medir métricas de iluminação e qualidade (`Y_mean`, `Y_std`, `snr_green_db`).

---

### **2️⃣ Análise Exploratória (Data Quality)**  
- Usar o `quality_summary.csv` para classificar vídeos por estabilidade de luz e compressão (RAW vs C23).  
- Identificar fatores que mais degradam o sinal iPPG.

---

### **3️⃣ Modelagem e Aprendizado**  
- Treinar um modelo de ML que, a partir de variáveis fotométricas e temporais (médias, variações e contrastes), **aprenda a prever e corrigir distorções de iluminação**, otimizando o sinal antes do processamento fisiológico.

---

### **4️⃣ Avaliação**  
- Aplicar os métodos **GREEN**, **CHROM** e **POS** antes e depois da correção.  
- Comparar métricas de desempenho: **SNR**, **estabilidade temporal**, **coerência de frequência cardíaca**.  
- Mostrar que o modelo melhora o resultado de forma consistente, independentemente do método rPPG.

---

### ✅ **Resultado Esperado**  
Comprovar que o aprendizado de máquina pode atuar como um **otimizador universal de qualidade de vídeo para iPPG**,  
reduzindo a sensibilidade a ruídos de compressão e iluminação e aumentando a robustez dos sinais fisiológicos extraídos de vídeos reais.

---

## ⚙️ Pipeline de Execução  

### **Etapa 1 — Explorar e compreender os dados (`quality_summary.csv`)**

**Objetivo:** entender a qualidade geral dos vídeos e separar os bons dos ruins antes de qualquer modelagem.  

**O que fazer:**  
- Carregar o `quality_summary.csv` e analisar as colunas:  
  `compression`, `fps`, `Y_mean_avg`, `Y_mean_std_over_time`, `snr_green_db`.  
- Avaliar:  
  - Iluminação média → quão claros ou escuros são os vídeos.  
  - Variação global de iluminação → estabilidade da luz.  
  - SNR → força do sinal fisiológico.  
  - Compressão → qualidade técnica da imagem.  

**Entregáveis:**  
- Gráficos exploratórios (histogramas e boxplots).  
- Relatório de qualidade com faixas ideais:  
  - `Y_mean_avg` entre 80–180  
  - `Y_mean_std_over_time ≤ 12`  
  - `snr_green_db ≥ 3 dB`  
- Lista de vídeos aprovados para seguir no pipeline.  

💡 *Equivalente à fase de “data understanding” no CRISP-DM.*

---

### **Etapa 2 — Selecionar os vídeos com melhor qualidade**

**Objetivo:** criar um subconjunto de vídeos confiáveis para aprendizado.  

**O que fazer:**  
- Filtrar no `quality_summary.csv` os vídeos aprovados.  
- Comparar **RAW** vs **C23**:  
  - RAW → SNR mais alto (condição ideal).  
  - C23 → compressão moderada, simula o mundo real (YouTube-like).  

**Entregáveis:**  
- Tabela de vídeos aprovados e motivo da aprovação.  
- Gráfico de SNR médio por compressão.  

💡 *Essa lista será usada para filtrar o `master_rgb.csv` e trabalhar apenas com dados limpos.*

---

### **Etapa 3 — Analisar os sinais temporais (`master_rgb.csv`)**

**Objetivo:** estudar o comportamento dos sinais RGB e da luminância ao longo do tempo (por vídeo).  

**O que fazer:**  
- Carregar o `master_rgb.csv`.  
- Selecionar apenas vídeos aprovados.  
- Para cada vídeo:  
  - Plotar série temporal de `G_mean` (canal verde).  
  - Observar ruído, oscilações de iluminação, picos regulares (batimentos).  
  - Verificar correlação entre `Y_mean` (luz) e `G_mean` (sinal).  
    > Se a correlação for alta, há interferência luminosa — bom caso para o modelo aprender a corrigir.  

**Entregáveis:**  
- Gráficos temporais por vídeo.  
- Insights sobre padrões de ruído de iluminação.

💡 *Aqui o dataset vira um “laboratório de observação”: entender como a luz influencia o sinal.*

---

### **Etapa 4 — Extrair janelas e features (base para o modelo de ML)**  

**Objetivo:** preparar uma base de aprendizado com indicadores numéricos que representem o comportamento dos sinais em pequenos intervalos.  

**O que fazer:**  
- Dividir cada vídeo em **janelas fixas** (ex.: 10 s).  
- Calcular features:  
  - **Fotométricas:** média, desvio padrão e coeficiente de variação de `R_mean`, `G_mean`, `B_mean`, `Y_mean`, `Y_std`.  
  - **Temporais:** variação de `Y_mean` dentro da janela.  
  - **Espectrais (opcional):** potência do sinal G na faixa 0.7–4 Hz.  
- Atribuir a cada janela:  
  - `video_id`, `compression`, `snr_green_db` (como rótulo de qualidade).  

**Entregável:**  
- Tabela de features com milhares de amostras (1 por janela) — base para o modelo de ML.  

💡 *Agora você tem um dataset clássico de Data Science: colunas = features, linhas = janelas, alvo = qualidade.*

---

### **Etapa 5 — Treinar o modelo de aprendizado de máquina**

**Objetivo:** criar um modelo que aprenda a **prever e corrigir** as condições de iluminação que prejudicam o sinal iPPG.  

**Como pensar:**  
- Problema: **regressão** (predizer SNR) ou **classificação** (“bom”, “médio”, “ruim”).  
- Target: `snr_green_db`.  
- Features: estatísticas de `Y_mean`, `Y_std`, variação temporal etc.  
- Algoritmos:  
  - Random Forest / Gradient Boosting (interpretação fácil);  
  - Rede Neural simples (para não linearidades).  

**O que o modelo aprende:**  
Reconhecer padrões de iluminação ruins e sugerir **correções automáticas** (ajuste de brilho, contraste, normalização, etc.).  

💡 *O modelo aprende “como o vídeo deveria parecer” para maximizar o SNR.*

---

### **Etapa 6 — Aplicar o modelo e comparar resultados**

**Objetivo:** provar que o modelo melhora a qualidade do sinal independentemente do método rPPG.  

**O que fazer:**  
- Aplicar o modelo para corrigir a iluminação.  
- Rodar **GREEN**, **CHROM** e **POS** antes e depois da correção.  
- Comparar:  
  - aumento do **SNR médio**;  
  - redução da variabilidade entre métodos;  
  - melhoria na consistência dos picos.  

**Entregáveis:**  
- Tabelas “antes vs depois” (por compressão e faixa de iluminação).  
- Gráfico de ganho médio de SNR (%).  
- Conclusão:  
  > “Nos vídeos C23, o modelo aumentou o SNR médio em X dB e reduziu o erro de BPM em Y%.”

---

## 📊 Metadados  

### **quality_summary.csv**

| Coluna | Significado |
|---------|-------------|
| `video_id` | Identificador do vídeo |
| `compression` | Qualidade técnica (`RAW` = sem perdas; `C23` = compressão moderada) |
| `n_frames` | Quantidade total de frames processados |
| `fps` | Taxa de quadros por segundo |
| `duration_s` | Duração total do vídeo |
| `Y_mean_avg` | Brilho médio global |
| `Y_mean_std_over_time` | Oscilação de brilho ao longo do tempo |
| `snr_green_db` | Força e pureza do sinal (maior = melhor) |

---

### **master_rgb.csv**

| Coluna | Significado |
|---------|-------------|
| `video_id` | Identificador do vídeo |
| `compression` | Tipo de compressão (`RAW` / `C23`) |
| `frame_idx` | Ordem temporal do frame |
| `t_sec` | Tempo em segundos |
| `R_mean`, `G_mean`, `B_mean` | Médias dos canais de cor |
| `Y_mean` | Brilho médio no frame |
| `Y_std` | Variação interna de brilho |
| `fps` | Frames por segundo |
| `label` | Classe do vídeo (sempre `Real`) |

---

## 🏁 Conclusão do Pipeline  

1. Mesmo em vídeos com compressão (**C23**) e iluminação variável, o modelo de Ciência de Dados consegue **detectar e corrigir automaticamente** essas falhas.  
2. Essa correção **melhora o desempenho** dos métodos tradicionais de iPPG (**GREEN**, **CHROM**, **POS**).  
3. O pipeline **padroniza a qualidade dos vídeos** antes da análise fisiológica, tornando o sistema mais **robusto, automatizado e escalável**.  

---

## ✉️ Autor  

---
