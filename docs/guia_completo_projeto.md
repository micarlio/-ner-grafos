# Guia Completo: Projeto NER + Grafos com TCCs de Engenharia da Computação

> Análise detalhada, metodológica e aplicada ao enunciado e ao conteúdo da disciplina.

---

## 1. Interpretação do trabalho

O professor não está pedindo apenas um pipeline de NLP. Ele está pedindo que você use NLP como **instrumento de coleta de dados** para então aplicar todo o ferramental de teoria de grafos estudado na disciplina. A ordem de importância é: grafo em primeiro lugar, NER como meio.

O objetivo final é transformar um corpus de TCCs em uma **rede de entidades nomeadas** e usá-la para demonstrar, de forma prática, que você sabe o que é densidade, grau, distribuição de grau, componentes conectados, clustering e diâmetro — e o que essas métricas significam neste contexto específico.

As entregas esperadas, interpretando o enunciado de forma concreta, são:

- Um corpus de N TCCs em PDF, coletados e organizados de fonte pública.
- Scripts para extração de texto dos PDFs.
- Scripts de NER que identificam e classificam entidades nos textos.
- Três grafos de coocorrência: um por sentença, um por parágrafo, um por janela de k caracteres.
- Análise do grafo com as métricas estudadas (grau, densidade, clustering, componentes, diâmetro, distribuição de grau).
- Comparação crítica entre as três estratégias de coocorrência.
- Figuras representativas do corpus, do processo e dos grafos.
- Repositório no GitHub organizado por etapas.
- Apresentação assíncrona de aproximadamente 10 minutos.

O ponto mais ignorado por quem faz esse tipo de trabalho é a **análise crítica**. O professor espera que você não apenas gere os grafos, mas discuta o que os números significam: por que certa entidade é um hub? Por que o grafo por sentença tem menos arestas? Por que o clustering do grafo por parágrafo é mais alto? Sem isso, o trabalho é apenas uma pipeline automática, não um trabalho acadêmico.

---

## 2. Definição de escopo

### Quantos TCCs baixar

A recomendação objetiva é **N = 30 TCCs**. Justificativa:

- Com menos de 15 TCCs, o grafo fica excessivamente esparso. Com poucos textos, poucas entidades se repetem entre documentos, o que gera uma rede fragmentada com componentes desconectados, baixa densidade e sem estrutura interessante para analisar.
- Com mais de 60 TCCs, o custo computacional aumenta, o tempo de extração e NER cresce linearmente, e os ganhos analíticos são marginais para os objetivos da disciplina.
- **30 TCCs** geram entre 500 e 2.000 entidades únicas (dependendo do limiar de frequência que você aplicar), o que é suficiente para um grafo com estrutura analisável, clustering mensurável e componentes identificáveis.

### Como justificar N = 30

No relatório/apresentação, justifique assim: "O valor de N = 30 foi escolhido para garantir um corpus com volume suficiente para observar padrões estruturais no grafo de coocorrência, mantendo viabilidade computacional e temporal para análise manual das entidades extraídas. Estudos de redes de coautoria, como os discutidos na disciplina, demonstram que grafos com essa ordem de magnitude já exibem propriedades como small world e distribuição de grau heterogênea."

### Balanceamento qualidade × tempo × custo

O principal gargalo não é o processamento do grafo (NetworkX é rápido), mas a **extração de texto e o NER**. Para 30 TCCs de ~70 páginas cada, a extração com PyMuPDF leva segundos por arquivo. O NER com spaCy no CPU leva entre 10 e 60 segundos por documento, dependendo do tamanho. No total, espere entre 20 e 40 minutos para processar o corpus completo pela primeira vez. Salve os resultados intermediários em JSON para não re-processar.

---

## 3. Estratégia metodológica completa

### Etapa 1 — Coleta do corpus

A melhor fonte para TCCs brasileiros abertos é o **BDTD** (Biblioteca Digital Brasileira de Teses e Dissertações): https://bdtd.ibict.br. Permite busca por área e download direto de PDFs públicos.

Outras fontes confiáveis:
- Repositório da UFSC: https://repositorio.ufsc.br
- LUMe UFRGS: https://lume.ufrgs.br
- Repositório USP: https://www.teses.usp.br
- Repositório UFPE: https://repositorio.ufpe.br

Busque por termos como "Engenharia da Computação" + "trabalho de conclusão de curso" nos filtros. Baixe os PDFs manualmente ou com um script básico usando `requests`. Guarde os metadados (título, autor, ano, instituição) em um arquivo `metadata.csv` desde o início — isso vai enriquecer a análise depois.

### Etapa 2 — Extração do texto

Use **PyMuPDF** (`pip install pymupdf`). Extraia por página, depois agrupe em parágrafos por heurística de espaçamento. Detalhe na seção 4.

### Etapa 3 — Limpeza e pré-processamento

Remova: referências bibliográficas (tudo após a última seção "Referências" ou "Bibliography"), cabeçalhos e rodapés (linhas curtas repetidas no topo/base das páginas), números de página isolados, legendas de figuras/tabelas. Normalize: remova hifenização de quebra de linha (`compu-\ntação` → `computação`), normalize espaços.

### Etapa 4 — NER

Use spaCy com o modelo `pt_core_news_lg` para TCCs em português. Se os TCCs forem em inglês, use `en_core_web_lg`. Salve as entidades por documento em JSON.

### Etapa 5 — Construção do grafo

Implemente as três estratégias em módulos separados. Cada estratégia gera um `networkx.Graph` ponderado. Salve os grafos em formato `.graphml` para usar no Gephi também.

### Etapa 6 — Análise

Calcule todas as métricas da disciplina para cada grafo. Monte uma tabela comparativa. Visualize.

### Etapa 7 — Repositório, figuras e apresentação

Organize o repositório conforme a seção 10. Gere as figuras conforme a seção 9. Grave a apresentação com as análises, não apenas com a pipeline.

---

## 4. Extração de texto dos PDFs

### A melhor ferramenta: PyMuPDF

```python
import fitz  # pymupdf

def extract_text_by_page(pdf_path):
    doc = fitz.open(pdf_path)
    pages = []
    for page in doc:
        text = page.get_text("text")
        pages.append(text)
    return pages
```

PyMuPDF é superior às alternativas porque: é muito mais rápido que `pdfminer.six`, preserva melhor a ordem de leitura, e lida razoavelmente bem com PDFs de duas colunas (comum em artigos, menos em TCCs).

Para TCCs a extração por página já é suficiente como granularidade primária. A partir do texto por página, você reconstrói parágrafos por heurística.

### Problemas comuns em PDFs acadêmicos e como tratar

**Layout de duas colunas**: alguns TCCs adotam formato de artigo com duas colunas. PyMuPDF pode mesclar o texto da coluna esquerda com o da coluna direita. Solução: use `page.get_text("blocks")` para obter blocos posicionados e ordene pela posição vertical+horizontal.

**Hifenização de final de linha**: palavras quebradas com hífen (`aprendi-\nzagem`) precisam ser reunidas. Use a regex `re.sub(r'-\n', '', text)` antes de qualquer processamento.

**Cabeçalhos e rodapés repetitivos**: detecte linhas que se repetem em 80%+ das páginas e remova-as. Uma forma simples: colete as primeiras e últimas linhas de cada página e compare por frequência.

**Referências bibliográficas**: identifique a última seção com a palavra "Referências" ou "References" e descarte tudo a partir daí. Isso é importante porque referências geram muitos falsos positivos no NER (nomes de autores de outros trabalhos que nada têm a ver com o conteúdo do TCC).

**Fórmulas matemáticas e código-fonte**: viram lixo na extração. Não é necessário removê-los completamente, mas eles não vão afetar o NER de forma significativa (o modelo vai ignorar tokens sem sentido).

### Extração por página × bloco × parágrafo

Para o NER e para o grafo de coocorrência, você precisa das unidades contextuais (sentenças e parágrafos) bem definidas. A estratégia recomendada é:

1. Extraia por página (granularidade básica).
2. Dentro de cada página, detecte parágrafos por duplo `\n\n` após a limpeza de rodapés.
3. Passe o texto limpo por parágrafo para o spaCy, que vai segmentar em sentenças automaticamente.

Assim você tem as três unidades disponíveis: página (para limpeza), parágrafo (para co-ocorrência de parágrafo), sentença (para co-ocorrência de sentença), e pode construir a janela de k caracteres diretamente do texto concatenado.

### Garantindo texto útil para NER

Após a extração e limpeza, o texto ideal para o NER é um fluxo contínuo por parágrafo, sem cabeçalhos, sem referências, sem hifenização quebrada e com pontuação preservada. O spaCy depende da pontuação para segmentar sentenças corretamente.

---

## 5. NER no contexto do trabalho

### O que são as entidades relevantes em TCCs de Engenharia da Computação

Nos TCCs de Engenharia da Computação, as entidades mais informativas para construir um grafo de conhecimento são:

**Categoria PER (Pessoa)**: autores clássicos citados no texto (Alan Turing, Yann LeCun, Geoffrey Hinton), orientadores mencionados, pesquisadores cujas técnicas são usadas. Esses nós ligam o trabalho ao ecossistema intelectual da área.

**Categoria ORG (Organização)**: universidades, empresas de tecnologia (Google, NVIDIA, Meta, Microsoft), laboratórios de pesquisa (DeepMind, OpenAI), agências de fomento (CAPES, CNPq). Revelam o ecossistema institucional do campo.

**Categoria LOC (Localização)**: cidades de conferências, países onde ferramentas foram desenvolvidas, locais de estudos de caso. Geralmente menos interessante para análise.

**Categoria MISC (Miscelânea)**: esta é a categoria mais rica para TCCs de Engenharia da Computação. Inclui: algoritmos (YOLO, ResNet, BERT, Transformer), frameworks (TensorFlow, PyTorch, Arduino, ROS), linguagens de programação (Python, Java, C++), protocolos (MQTT, HTTP, Modbus), hardware (Raspberry Pi, ESP32, FPGA), conceitos técnicos quando tratados como termos específicos. Esta categoria vai gerar os nós mais interessantes do grafo.

### Modelos prontos vs. adaptação

Para este trabalho, **modelos prontos são suficientes e recomendados**. A justificativa é que adaptar (fine-tuning) um modelo de NER para entidades específicas de Engenharia da Computação exigiria um dataset anotado manualmente que você não tem. Além disso, os modelos do spaCy já capturam bem entidades como organizações e pessoas.

O problema real é que MISC em TCCs de Engenharia da Computação vai incluir muita coisa não capturada por modelos genéricos: "YOLO", "ResNet", "MQTT" dificilmente serão marcados como entidades por um modelo treinado em notícias ou textos jornalísticos.

**Solução prática**: use o NER do spaCy para PER, ORG e LOC, e **adicione uma lista de termos técnicos** relevantes para Engenharia da Computação como entidades MISC. Essa lista pode ser construída manualmente com 50-100 termos comuns (Python, TensorFlow, PyTorch, Arduino, Raspberry Pi, YOLO, ResNet, BERT, LSTM, CNN, SVM, etc.) e detectada por correspondência exata no texto. O spaCy tem um componente `EntityRuler` para isso.

### Como avaliar se as entidades extraídas fazem sentido

Amostrage manual: pegue 5 TCCs aleatórios, veja quais entidades foram extraídas e classifique como corretas/incorretas. Calcule uma precisão aproximada. Para o trabalho, uma avaliação qualitativa com exemplos é suficiente — o professor não espera um benchmark formal de NER, mas espera que você discuta limitações.

Sinais de problema: muitos tokens genéricos sendo marcados como entidades (artigos, preposições), nenhuma entidade técnica sendo capturada, entidades duplicadas com caixa diferente ("Python" vs "python"). Trate esses casos na etapa de normalização.

---

## 6. Grafo de coocorrência

### Definição formal no seu caso

O grafo de coocorrência de NER é um grafo **não direcionado e ponderado**:

```
G = (V, E, W)
```

Onde:
- **V** = conjunto de entidades únicas extraídas de todos os TCCs (depois de normalização)
- **E** = pares de entidades que aparecem juntas em alguma unidade de contexto
- **W** = frequência total de coocorrência de cada par em todo o corpus

O grafo é não direcionado porque coocorrência não tem sentido direcional (se "Python" e "TensorFlow" aparecem juntos, nenhum implica o outro).

### O que significa coocorrência

Duas entidades coocorrem quando aparecem na **mesma unidade de contexto**. A escolha da unidade de contexto é o coração da comparação que o professor pediu.

### Sentença como unidade de contexto

```python
import spacy
from itertools import combinations
import networkx as nx

nlp = spacy.load("pt_core_news_lg")

def build_graph_by_sentence(documents_entities):
    """
    documents_entities: lista de documentos, cada um sendo
    lista de (entidade_texto, entidade_tipo, sent_index)
    """
    G = nx.Graph()
    # para cada sentença, pegar todas as entidades e adicionar arestas
    ...
```

A sentença é o contexto mais restrito. A vantagem é que as relações geradas tendem a ser semanticamente mais precisas: se "Python" e "TensorFlow" aparecem na mesma sentença, é muito provável que a relação seja real e direta. A desvantagem é que relações temáticas mais amplas — por exemplo, um parágrafo inteiro discutindo como BERT, Transformers e atenção se relacionam — podem gerar poucas arestas porque cada sentença menciona apenas parte do conjunto.

**Impacto esperado no grafo**: menos arestas, grafo mais esparso, menor densidade, potencialmente mais componentes desconectados, arestas de peso mais alto representando relações genuinamente fortes.

### Parágrafo como unidade de contexto

O parágrafo é um contexto mais amplo e tematicamente coeso. Um parágrafo sobre "aprendizado profundo" vai mencionar várias entidades juntas que talvez nunca apareçam na mesma sentença. O risco é introduzir **conexões espúrias**: duas entidades no mesmo parágrafo podem estar em posições bem distantes e sem relação real.

**Impacto esperado no grafo**: mais arestas, maior densidade, maior clustering (mais triângulos, porque entidades que coocorrem com uma terceira tendem a aparecer nos mesmos parágrafos), maior componente conectado, diâmetro menor.

### Janela de k caracteres

A janela de k caracteres define um contexto de tamanho fixo ao redor de cada ocorrência de entidade. Para cada entidade encontrada na posição `i` do texto, você considera todas as outras entidades dentro do intervalo `[i-k/2, i+k/2]` (em caracteres).

Valores sugeridos para k: 200, 500 e 1000 caracteres, para comparação interna. k=200 aprox. a uma sentença, k=1000 aprox. um parágrafo. A vantagem é o controle fino da granularidade. A desvantagem é que ignora fronteiras naturais do texto (uma janela pode cruzar parágrafos ou seções).

**Impacto esperado no grafo**: intermediário entre sentença e parágrafo para k moderado. A sensibilidade ao parâmetro k é em si uma análise interessante.

### Implementação esquemática

```python
def build_graph_by_window(text, entities, k=500):
    """
    text: string do documento inteiro
    entities: lista de (start_char, end_char, texto, tipo)
    k: tamanho da janela em caracteres
    """
    G = nx.Graph()
    for i, (start_i, end_i, ent_i, type_i) in enumerate(entities):
        center_i = (start_i + end_i) // 2
        for j, (start_j, end_j, ent_j, type_j) in enumerate(entities):
            if i >= j:
                continue
            center_j = (start_j + end_j) // 2
            if abs(center_i - center_j) <= k // 2:
                if G.has_edge(ent_i, ent_j):
                    G[ent_i][ent_j]['weight'] += 1
                else:
                    G.add_edge(ent_i, ent_j, weight=1)
    return G
```

---

## 7. Análise crítica e aplicação do conteúdo da disciplina

Este é o diferencial entre um trabalho mediano e um bom trabalho. Aqui estão formas concretas de aplicar cada conceito estudado:

### Distribuição de grau

Plote a distribuição de grau do grafo (histograma e escala log-log). Em redes de coautoria e de citações acadêmicas — contextos análogos ao seu — é comum observar distribuições com cauda longa (few entities are connected to many others). Discuta: a distribuição do seu grafo lembra uma lei de potência? Se sim, o que isso significa? Quais são os hubs (entidades de alto grau) e por que fazem sentido no contexto de TCCs de Engenharia da Computação?

### Componentes conectados e Giant Component

Verifique quantos componentes conectados existem em cada grafo e qual o tamanho do maior deles (GCC). Um grafo por sentença provavelmente terá mais componentes isolados do que um por parágrafo. Discuta: entidades que ficam em componentes pequenos são específicas demais? Ou são erros do NER?

### Densidade

Calcule a densidade para os três grafos e compare. Esperado: d_sentença < d_janela < d_parágrafo. Discuta o que significa uma densidade maior: mais conexões — mas conexões de qualidade inferior?

### Coeficiente de clustering

Calcule o clustering médio e o clustering por nó. Nós com alto clustering são entidades que formam "tribos" coesas — por exemplo, Python, NumPy, Pandas e Scikit-learn provavelmente vão ter alto clustering entre si. Discuta esses clusters como comunidades temáticas dentro do campo.

### Diâmetro e distância média

Calcule o diâmetro do GCC e o average shortest path length. Compare com o que se espera de um small world (distâncias curtas mesmo em redes grandes). Discuta: o seu grafo é um mundo pequeno? O que isso significaria para o ecossistema de TCCs de Engenharia da Computação?

### Análogo a redes conhecidas

Compare explicitamente o seu grafo com as redes discutidas na disciplina: redes de coautoria (onde nós são pesquisadores e arestas são papers em comum), redes de citação (onde o contexto de coocorrência é o artigo). Diga em que seu grafo é similar e em que é diferente. Isso demonstra compreensão do conteúdo da disciplina, não apenas execução técnica.

### Análise de limitações

Discuta: o NER genérico perdeu entidades técnicas? A hifenização causou duplicações ("aprendiza-\ndo" extraído como entidade separada)? A falta de desambiguação (Java o país vs Java a linguagem) pode ter gerado arestas espúrias? Essas limitações são o que transforma o trabalho em texto acadêmico.

---

## 8. Análise de desempenho: sentença vs parágrafo vs k-caracteres

### Tabela de métricas para comparação

Para cada um dos três grafos, calcule e apresente em tabela:

| Métrica | Sentença | Parágrafo | k-chars (k=500) |
|---|---|---|---|
| Número de nós (V) | ... | ... | ... |
| Número de arestas (E) | ... | ... | ... |
| Densidade (d) | ... | ... | ... |
| Grau médio (⟨k⟩) | ... | ... | ... |
| Tamanho do GCC | ... | ... | ... |
| Número de componentes | ... | ... | ... |
| Diâmetro do GCC | ... | ... | ... |
| Distância média (⟨l⟩) | ... | ... | ... |
| Clustering médio (C) | ... | ... | ... |
| Tempo de construção (s) | ... | ... | ... |

### Análise qualitativa das relações geradas

Além das métricas estruturais, compare a **qualidade das arestas**. Pegue os 10 pares de entidades com maior peso em cada grafo e avalie manualmente se a relação é plausível. Você provavelmente vai encontrar que o grafo por sentença tem relações mais "focadas" (Python ↔ TensorFlow: ambos na mesma sentença quase sempre significa que estão sendo usados juntos), enquanto o grafo por parágrafo tem relações mais "temáticas" mas potencialmente mais ruidosas.

### Custo computacional

Para a comparação de desempenho, meça o tempo de construção de cada grafo com `time.time()`. A janela de k caracteres tem complexidade O(E·n) onde E é o número de entidades e n o número de ocorrências, mas para o tamanho do seu corpus isso não vai ser um problema prático.

### Que tipo de conclusão é interessante tirar

A conclusão mais valiosa não é "qual estratégia é melhor", mas "qual estratégia é mais adequada para qual tipo de pergunta":

- **Se quero identificar relações fortes e diretas entre tecnologias**: sentença.
- **Se quero mapear o ecossistema temático geral de um campo**: parágrafo.
- **Se quero controlar a granularidade e testar hipóteses sobre a distância semântica**: janela de k caracteres.

Isso demonstra pensamento crítico, não apenas execução.

---

## 9. Figuras e visualizações

### Figura 1 — Fluxo metodológico

Um diagrama de blocos mostrando as etapas do pipeline: Corpus → PDF → Texto → NER → Grafo → Análise. Use matplotlib ou draw.io. Esta figura vai aparecer na apresentação e no README.

### Figura 2 — Estatísticas do corpus

Histograma com o número de páginas de cada TCC, e outro com o número de entidades extraídas por TCC. Isso contextualiza o corpus para o leitor.

### Figura 3 — Distribuição de tipos de entidade

Gráfico de barras: quantidade de entidades por tipo (PER, ORG, LOC, MISC) no corpus inteiro. Esperado: MISC e ORG dominando em TCCs de Engenharia da Computação.

### Figura 4 — Top-20 entidades por frequência

Gráfico de barras horizontal com as 20 entidades mais frequentes no corpus. Esta é uma das figuras mais informativas do trabalho — vai mostrar quais são os termos centrais da área.

### Figura 5 — Exemplo de NER anotado

Mostre um parágrafo de um TCC com as entidades destacadas por cor/categoria. Pode ser gerado com `displacy.render` do spaCy e exportado como HTML ou imagem.

### Figura 6 — Visualização dos três grafos lado a lado

Três painéis: grafo por sentença, por parágrafo, por k-chars. Use NetworkX com layout `spring_layout` ou `kamada_kawai_layout`. Mostre apenas o GCC se o grafo completo for muito denso. Escale o tamanho dos nós pelo grau. Esta é a figura central do trabalho.

### Figura 7 — Distribuição de grau (escala log-log)

Scatter plot do grau k no eixo X vs a fração de nós P(k) no eixo Y, em escala log-log, para os três grafos. Inclua uma linha de referência de lei de potência se a distribuição se aproximar.

### Figura 8 — Ego-network de uma entidade relevante

Escolha uma entidade de alto grau (por exemplo, "Python" ou "Machine Learning") e visualize sua ego-network: o nó central, seus vizinhos diretos e as arestas entre eles. Isso torna o grafo interpretável para quem não é da área.

### Figura 9 — Tabela comparativa de métricas

Uma heatmap ou tabela visual com as métricas da seção 8 para as três estratégias. Facilita a comparação no relatório e na apresentação.

### Figura 10 — Componentes conectados

Mostre a distribuição de tamanho dos componentes conectados (quantos componentes têm 1 nó, 2 nós, etc.) para os três grafos. Isso ilustra a diferença de conectividade entre as estratégias.

---

## 10. Estrutura do repositório GitHub

```
ner-tccs-engcomp/
│
├── data/
│   ├── raw_pdfs/              # PDFs originais (gitignore se forem grandes)
│   ├── metadata.csv           # título, autor, ano, instituição, URL de cada TCC
│   ├── extracted_text/        # textos extraídos por TCC (JSON: {page: ..., paragraphs: ...})
│   ├── entities/              # entidades extraídas por TCC (JSON: [{text, label, sent_idx, para_idx, char_start, char_end}])
│   └── processed/
│       ├── all_entities.csv   # todas as entidades com frequência
│       └── entity_pairs_*.csv # pares de entidades por estratégia
│
├── graphs/
│   ├── graph_sentence.graphml
│   ├── graph_paragraph.graphml
│   └── graph_kchars.graphml
│
├── src/
│   ├── 01_download/
│   │   └── download_tccs.py   # script de download dos PDFs
│   ├── 02_extraction/
│   │   ├── extract_text.py    # extração com PyMuPDF
│   │   └── clean_text.py      # limpeza: remoção de headers, referências, hifenização
│   ├── 03_ner/
│   │   ├── run_ner.py         # aplica spaCy + EntityRuler
│   │   └── tech_entities.py   # lista de termos técnicos de Engenharia da Computação
│   ├── 04_graph/
│   │   ├── build_graph_sentence.py
│   │   ├── build_graph_paragraph.py
│   │   └── build_graph_kchars.py
│   └── 05_analysis/
│       ├── compute_metrics.py  # densidade, grau, clustering, diâmetro, componentes
│       └── compare_strategies.py  # tabela comparativa das três estratégias
│
├── notebooks/
│   ├── 01_corpus_exploration.ipynb      # estatísticas do corpus
│   ├── 02_ner_examples.ipynb            # visualização de exemplos de NER
│   ├── 03_graph_analysis.ipynb          # análise completa dos grafos
│   └── 04_comparison.ipynb              # comparação entre estratégias
│
├── figures/
│   ├── 01_methodology_flow.png
│   ├── 02_corpus_stats.png
│   ├── 03_entity_types.png
│   ├── 04_top20_entities.png
│   ├── 05_ner_example.png
│   ├── 06_graphs_comparison.png
│   ├── 07_degree_distribution.png
│   ├── 08_ego_network.png
│   ├── 09_metrics_comparison.png
│   └── 10_components.png
│
├── results/
│   └── metrics_summary.csv             # tabela final de métricas por estratégia
│
├── README.md                           # documentação do projeto
├── requirements.txt                    # dependências (pymupdf, spacy, networkx, etc.)
└── .gitignore                          # ignorar raw_pdfs se forem grandes
```

### README mínimo

O README deve conter: objetivo do projeto, dataset utilizado (quais universidades, critério de seleção, período), como rodar cada etapa (comandos em ordem), descrição das principais figuras. Um README bem feito é o cartão de visita do repositório.

---

## 11. Plano de execução com prioridades

### Fase 1 — Base (faça isso primeiro, tudo depende disso)

1. **Baixar 30 TCCs e criar metadata.csv.** Valide: todos os PDFs abrem corretamente? Anote título, autor, ano, instituição.
2. **Extrair texto de 3-5 TCCs** como piloto. Verifique manualmente: o texto está legível? As referências foram removidas? Os parágrafos fazem sentido?
3. **Rodar NER nos 3-5 TCCs piloto.** Verifique manualmente: as entidades fazem sentido? Quantas entidades técnicas foram capturadas? Quais estão faltando? Ajuste a lista de termos técnicos do EntityRuler.

*Não avance para a fase 2 sem validar o piloto. Erros no NER propagam para o grafo e invalidam toda a análise.*

### Fase 2 — Pipeline completa

4. **Processar os 30 TCCs com extração + NER.** Salve tudo em JSON.
5. **Construir os três grafos.** Salve em `.graphml`.
6. **Calcular métricas básicas** para os três grafos. Monte a tabela comparativa.

*Valide: os grafos têm tamanho razoável? O grafo por parágrafo tem mais arestas que o por sentença? Se não, há bug na implementação.*

### Fase 3 — Análise e visualização

7. **Gerar todas as figuras.** Comece pelas mais simples (estatísticas do corpus) e vá para as mais complexas (grafos, distribuição de grau).
8. **Escrever a análise crítica.** Para cada métrica, escreva 2-3 frases interpretando o resultado no contexto dos TCCs de Engenharia da Computação.
9. **Explorar o Gephi.** Abra o `.graphml` no Gephi para gerar visualizações mais bonitas dos grafos com detecção de comunidades (algoritmo de Louvain).

### Fase 4 — Entrega

10. **Organizar o repositório** com a estrutura da seção 10.
11. **Escrever o README.**
12. **Gravar a apresentação de 10 minutos.** Sugestão de roteiro: 1min introdução, 2min corpus e NER, 2min grafos e metodologia, 3min análise e comparação, 2min conclusões e limitações.

### Decisões críticas (não deixe para depois)

- **Escolha do modelo spaCy**: `pt_core_news_lg` para português, `en_core_web_lg` para inglês. Se os TCCs misturam idiomas, escolha o idioma predominante.
- **Lista de termos técnicos**: defina antes de processar o corpus inteiro. Adicione ao menos 50 termos comuns de Engenharia da Computação.
- **Normalização de entidades**: decida como tratar "Python 3", "Python 3.8", "Python". Recomendação: normalize para o termo base ("Python").
- **Limiar de frequência**: entidades que aparecem apenas 1 vez no corpus inteiro provavelmente são ruído. Considere remover entidades com frequência < 3 antes de construir o grafo.

### Erros comuns a evitar

- Não remover referências bibliográficas → grafo polui-se com nomes de autores externos sem relação com o conteúdo.
- Não normalizar maiúsculas/minúsculas → "Python" e "python" viram dois nós separados.
- Construir o grafo sem limiar de frequência → grafo com milhares de nós isolados, impossível de analisar.
- Calcular diâmetro no grafo desconexo → use sempre `G.subgraph(max_component)` antes.
- Não salvar resultados intermediários → re-processar 30 TCCs do zero toda vez que você mudar um parâmetro.

---

## 12. Recomendação final: a melhor versão deste projeto

A melhor versão deste projeto tem três características que o diferenciam de um trabalho mediano:

**Primeiro: o NER não é apenas automático.** Além do spaCy padrão, você usa um `EntityRuler` com uma lista curada de 50-100 entidades técnicas de Engenharia da Computação. Isso garante que termos como "YOLO", "PyTorch", "ESP32" e "MQTT" apareçam no grafo, tornando os resultados muito mais interessantes e aderentes ao tema.

**Segundo: a análise vai além das métricas.** Você não apenas calcula grau, clustering e densidade — você interpreta o que os hubs significam, por que certas entidades formam clusters, qual subgrafo representa o ecossistema de IoT vs. o de Machine Learning dentro dos TCCs. Você menciona explicitamente os conceitos da disciplina (small world, GCC, distribuição de grau heterogênea) e conecta os resultados ao que foi estudado.

**Terceiro: a comparação entre estratégias tem uma narrativa.** Você não apenas coloca três números na tabela. Você argumenta por que o grafo por parágrafo é mais denso, por que o clustering é mais alto, e por que isso não significa necessariamente que é "melhor" — apenas que captura um tipo diferente de relação semântica. Você discute qual estratégia você usaria se o objetivo fosse recomendar ferramentas a um estudante de Engenharia da Computação, e por quê.

### Stack tecnológica recomendada

```
pymupdf==1.23+         # extração de PDF
spacy==3.7+            # NER
pt_core_news_lg        # modelo spaCy para português
networkx==3.2+         # construção e análise de grafos
matplotlib==3.8+       # visualizações
gephi (desktop)        # visualização interativa dos grafos
pandas                 # manipulação de dados tabulares
jupyter                # notebooks de análise
```

### Estratégia de coocorrência recomendada como principal

Use a **sentença** como estratégia principal para sua análise mais detalhada. Justificativa: as relações são mais precisas, mais interpretáveis, e mais fáceis de defender. Use parágrafo e k-chars (k=500) como comparação. Isso inverte o que a maioria dos alunos faz (que é usar parágrafo por ser mais fácil de implementar) e demonstra que você entendeu que precisão > quantidade de arestas.

### Uma análise que o professor vai lembrar

Ao final do trabalho, verifique se o seu grafo por sentença tem propriedades de small world: distância média curta (< 6 saltos), clustering maior que um grafo aleatório equivalente. Se tiver, você pode afirmar que o ecossistema tecnológico presente nos TCCs de Engenharia da Computação exibe a propriedade de "seis graus de separação" — qualquer duas tecnologias estão conectadas por poucos intermediários. Isso é uma conclusão intelectualmente interessante e diretamente ligada ao conteúdo da disciplina.
