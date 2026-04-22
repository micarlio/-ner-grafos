# Planejamento Detalhado dos Notebooks

Cada notebook é independente (importa o que precisa de `data/` e `graphs/`), mas a ordem de execução segue a pipeline: 01 → 02 → 03 → 04. Os arquivos intermediários salvos pelo `src/` são os insumos dos notebooks.

---

## Notebook 01 — `corpus_exploration.ipynb`

**Objetivo:** entender o corpus antes de qualquer processamento. Responde: quais TCCs foram coletados, de onde, com que distribuição de anos e instituições, e qual é a qualidade do texto extraído.

Este notebook não exige que o NER já tenha rodado. Ele só precisa de `metadata.csv` e dos arquivos em `data/extracted_text/`.

---

### Seção 1 — Setup e carregamento

**Célula 1 — Markdown**
```
# Exploração do Corpus
Análise descritiva dos 30 TCCs de Engenharia da Computação coletados.
Objetivo: entender a composição do corpus antes de aplicar NER e construir os grafos.
```

**Célula 2 — Código: imports**
```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import json
import os
from pathlib import Path

# Configuração de estilo dos gráficos
plt.rcParams['figure.figsize'] = (10, 5)
plt.rcParams['font.size'] = 12
```

**Célula 3 — Código: carregar metadata.csv**
```python
meta = pd.read_csv('../data/metadata.csv')
print(f"Total de TCCs: {len(meta)}")
meta.head()
```
*Output esperado: tabela com colunas título, autor, ano, instituição, páginas, URL.*

---

### Seção 2 — Estatísticas do corpus

**Célula 4 — Markdown**
```
## Distribuição por ano e instituição
```

**Célula 5 — Código: distribuição por ano**
```python
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Distribuição por ano
meta['ano'].value_counts().sort_index().plot(kind='bar', ax=axes[0], color='steelblue')
axes[0].set_title('TCCs por Ano')
axes[0].set_xlabel('Ano')
axes[0].set_ylabel('Quantidade')
axes[0].tick_params(axis='x', rotation=45)

# Distribuição por instituição
meta['instituicao'].value_counts().plot(kind='barh', ax=axes[1], color='coral')
axes[1].set_title('TCCs por Instituição')
axes[1].set_xlabel('Quantidade')

plt.tight_layout()
plt.savefig('../figures/02_corpus_stats_ano_instituicao.png', dpi=150, bbox_inches='tight')
plt.show()
```
*Output esperado: dois gráficos de barras. Salva figura.*

**Célula 6 — Código: estatísticas de páginas**
```python
print("Estatísticas de número de páginas:")
print(meta['paginas'].describe().round(1))

meta['paginas'].plot(kind='hist', bins=15, color='mediumseagreen', edgecolor='white')
plt.title('Distribuição do Número de Páginas por TCC')
plt.xlabel('Número de páginas')
plt.ylabel('Frequência')
plt.tight_layout()
plt.savefig('../figures/02b_corpus_stats_paginas.png', dpi=150, bbox_inches='tight')
plt.show()
```
*Output esperado: tabela describe + histograma.*

---

### Seção 3 — Análise do texto extraído

**Célula 7 — Markdown**
```
## Qualidade e volume do texto extraído
Verificamos quantas palavras e parágrafos foram extraídos por TCC após a limpeza.
```

**Célula 8 — Código: carregar textos e calcular volume**
```python
text_stats = []

for fname in sorted(Path('../data/extracted_text/').glob('*.json')):
    with open(fname) as f:
        data = json.load(f)
    
    tcc_id = fname.stem
    all_text = ' '.join([p for p in data['paragraphs'] if p.strip()])
    n_words = len(all_text.split())
    n_paragraphs = len([p for p in data['paragraphs'] if len(p.strip()) > 50])
    n_pages = len(data['pages'])
    
    text_stats.append({
        'tcc_id': tcc_id,
        'n_words': n_words,
        'n_paragraphs': n_paragraphs,
        'n_pages': n_pages
    })

df_text = pd.DataFrame(text_stats)
print(df_text.describe().round(0))
```
*Output esperado: tabela com contagens por TCC.*

**Célula 9 — Código: gráfico de palavras por TCC**
```python
df_text_sorted = df_text.sort_values('n_words')

fig, ax = plt.subplots(figsize=(12, 5))
ax.barh(df_text_sorted['tcc_id'], df_text_sorted['n_words'], color='cornflowerblue')
ax.set_xlabel('Número de palavras (após limpeza)')
ax.set_title('Volume de Texto Extraído por TCC')
ax.axvline(df_text_sorted['n_words'].mean(), color='red', linestyle='--', label='Média')
ax.legend()
plt.tight_layout()
plt.savefig('../figures/02c_corpus_volume_texto.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

### Seção 4 — Exemplo de texto extraído

**Célula 10 — Markdown**
```
## Exemplo de texto extraído e limpo
Verificação qualitativa: amostragem de parágrafos para validar a extração.
```

**Célula 11 — Código: mostrar amostra de parágrafos**
```python
# Escolhe um TCC aleatório e mostra 3 parágrafos
import random

sample_file = random.choice(list(Path('../data/extracted_text/').glob('*.json')))
with open(sample_file) as f:
    sample = json.load(f)

print(f"TCC: {sample_file.stem}\n")
for i, p in enumerate(sample['paragraphs'][:5]):
    if len(p.strip()) > 100:
        print(f"--- Parágrafo {i+1} ---")
        print(p[:400])
        print()
```
*Output esperado: texto legível, sem cabeçalhos/rodapés/referências. Se aparecer lixo, é sinal para ajustar a limpeza.*

**Célula 12 — Markdown (conclusão da seção)**
```
### Observações sobre a qualidade da extração
[Preencher manualmente após rodar: quantos TCCs tiveram problemas, quais foram os problemas observados]
```

---

## Notebook 02 — `ner_examples.ipynb`

**Objetivo:** mostrar e avaliar os resultados do NER. Responde: quais entidades foram extraídas, em que proporção por tipo, quais são as mais frequentes, e a extração está fazendo sentido qualitativamente?

Pré-requisito: `src/03_ner/run_ner.py` já rodou e gerou os JSONs em `data/entities/`.

---

### Seção 1 — Setup

**Célula 1 — Markdown**
```
# Análise dos Resultados de NER
Visualização e avaliação das entidades nomeadas extraídas dos TCCs de Engenharia da Computação.
```

**Célula 2 — Código: imports**
```python
import pandas as pd
import json
from pathlib import Path
import matplotlib.pyplot as plt
from collections import Counter
import spacy
from spacy import displacy

nlp = spacy.load('pt_core_news_lg')
plt.rcParams['figure.figsize'] = (11, 5)
```

**Célula 3 — Código: carregar todas as entidades**
```python
all_entities = []

for fname in sorted(Path('../data/entities/').glob('*.json')):
    with open(fname) as f:
        entities = json.load(f)
    for ent in entities:
        ent['tcc_id'] = fname.stem
        all_entities.append(ent)

df_ents = pd.DataFrame(all_entities)
print(f"Total de ocorrências de entidades: {len(df_ents)}")
print(f"Entidades únicas: {df_ents['text_norm'].nunique()}")
print(f"\nDistribuição por tipo:")
print(df_ents['label'].value_counts())
```
*Output esperado: contagens totais e por tipo.*

---

### Seção 2 — Distribuição por tipo de entidade

**Célula 4 — Código: gráfico de pizza e barras**
```python
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Pizza com proporção por tipo
label_counts = df_ents['label'].value_counts()
axes[0].pie(label_counts, labels=label_counts.index, autopct='%1.1f%%',
            colors=['#4C72B0','#DD8452','#55A868','#C44E52'])
axes[0].set_title('Proporção por Tipo de Entidade')

# Barras com contagem
label_counts.plot(kind='bar', ax=axes[1], color=['#4C72B0','#DD8452','#55A868','#C44E52'])
axes[1].set_title('Ocorrências por Tipo de Entidade')
axes[1].set_xlabel('Tipo')
axes[1].set_ylabel('Quantidade de ocorrências')
axes[1].tick_params(axis='x', rotation=0)

plt.tight_layout()
plt.savefig('../figures/03_entity_types_distribution.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

### Seção 3 — Entidades mais frequentes

**Célula 5 — Markdown**
```
## Top entidades por frequência
Análise das entidades mais recorrentes no corpus inteiro e por tipo.
```

**Célula 6 — Código: top-20 geral**
```python
top20 = df_ents['text_norm'].value_counts().head(20)

fig, ax = plt.subplots(figsize=(10, 7))
top20.sort_values().plot(kind='barh', ax=ax, color='steelblue')
ax.set_title('Top 20 Entidades Mais Frequentes no Corpus')
ax.set_xlabel('Frequência')
plt.tight_layout()
plt.savefig('../figures/04_top20_entities.png', dpi=150, bbox_inches='tight')
plt.show()
```
*Output esperado: as entidades mais recorrentes. Esperado ver Python, Machine Learning, instituições, etc.*

**Célula 7 — Código: top-10 por tipo**
```python
fig, axes = plt.subplots(2, 2, figsize=(14, 10))
tipos = ['MISC', 'ORG', 'PER', 'LOC']
cores = ['#4C72B0', '#DD8452', '#55A868', '#C44E52']

for ax, tipo, cor in zip(axes.flat, tipos, cores):
    top = df_ents[df_ents['label'] == tipo]['text_norm'].value_counts().head(10)
    top.sort_values().plot(kind='barh', ax=ax, color=cor)
    ax.set_title(f'Top 10 — {tipo}')
    ax.set_xlabel('Frequência')

plt.suptitle('Top 10 Entidades por Tipo', fontsize=14, y=1.01)
plt.tight_layout()
plt.savefig('../figures/04b_top10_por_tipo.png', dpi=150, bbox_inches='tight')
plt.show()
```
*Este gráfico é muito ilustrativo: MISC vai mostrar o ecossistema tecnológico dos TCCs.*

---

### Seção 4 — Exemplos visuais de NER anotado

**Célula 8 — Markdown**
```
## Visualização de entidades no texto original
Mostramos parágrafos com as entidades destacadas por categoria usando o displacy do spaCy.
```

**Célula 9 — Código: visualizar NER em parágrafo real**
```python
# Pegar um parágrafo rico em entidades para demonstração
# Escolher manualmente um bom exemplo após rodar o notebook
sample_text = """
O sistema proposto utiliza Python e TensorFlow para treinar uma rede neural convolucional (CNN) 
aplicada ao reconhecimento de padrões. Os experimentos foram conduzidos na UFPE com suporte 
do laboratório LabES. O modelo ResNet-50, proposto por He et al., foi utilizado como 
arquitetura base, com ajuste fino no conjunto de dados gerado no Recife.
"""

doc = nlp(sample_text)
displacy.render(doc, style='ent', jupyter=True)
```
*Output esperado: HTML com entidades coloridas por tipo. Esta figura vai para os slides.*

**Célula 10 — Código: salvar a visualização como HTML**
```python
html = displacy.render(doc, style='ent', page=True)
with open('../figures/05_ner_example.html', 'w', encoding='utf-8') as f:
    f.write(html)
print("Salvo em figures/05_ner_example.html")
```

---

### Seção 5 — Avaliação qualitativa e limitações

**Célula 11 — Código: exemplos de possíveis erros**
```python
# Mostrar entidades marcadas como PER que parecem ser tecnologias (falsos positivos)
# e vice-versa — análise manual de qualidade

suspeitos = df_ents[df_ents['label'] == 'PER']['text_norm'].value_counts().head(30)
print("Entidades marcadas como PER (verificar se fazem sentido):")
print(suspeitos.to_string())
```

**Célula 12 — Markdown (a ser preenchida após análise)**
```
## Limitações identificadas no NER
- [ex: "Java" não foi capturado como MISC em alguns TCCs]
- [ex: nomes compostos de frameworks foram separados em múltiplas entidades]
- [ex: siglas em maiúsculas nem sempre foram reconhecidas corretamente]

Decisões tomadas para mitigar esses problemas: [descrever ajustes feitos no EntityRuler]
```

---

### Seção 6 — Distribuição por TCC

**Célula 13 — Código: entidades por TCC**
```python
ents_per_tcc = df_ents.groupby('tcc_id')['text_norm'].count().sort_values()

ents_per_tcc.plot(kind='barh', figsize=(10, 8), color='mediumpurple')
plt.title('Número de Ocorrências de Entidades por TCC')
plt.xlabel('Ocorrências')
plt.tight_layout()
plt.savefig('../figures/03b_entities_per_tcc.png', dpi=150, bbox_inches='tight')
plt.show()

print(f"\nMédia de entidades por TCC: {ents_per_tcc.mean():.0f}")
print(f"Mínimo: {ents_per_tcc.min()} | Máximo: {ents_per_tcc.max()}")
```

---

## Notebook 03 — `graph_analysis.ipynb`

**Objetivo:** análise completa dos três grafos com todas as métricas da disciplina. Este é o notebook mais importante e mais extenso. Responde: qual é a estrutura de cada grafo? Quais são os hubs? O grafo tem propriedades de small world? Quais comunidades temáticas emergem?

Pré-requisito: `src/04_graph/build_graph_*.py` já rodou e gerou os `.graphml` em `graphs/`.

---

### Seção 1 — Setup e carregamento

**Célula 1 — Markdown**
```
# Análise dos Grafos de Coocorrência de NER
Aplicação dos conceitos de teoria de grafos estudados na disciplina para analisar
a rede de entidades nomeadas extraídas dos TCCs de Engenharia da Computação.
```

**Célula 2 — Código: imports**
```python
import networkx as nx
import matplotlib.pyplot as plt
import matplotlib.colors as mcolors
import numpy as np
import pandas as pd
from collections import Counter

plt.rcParams['figure.figsize'] = (12, 8)
plt.rcParams['font.size'] = 11
```

**Célula 3 — Código: carregar os três grafos**
```python
G_sent = nx.read_graphml('../graphs/graph_sentence.graphml')
G_para = nx.read_graphml('../graphs/graph_paragraph.graphml')
G_kchars = nx.read_graphml('../graphs/graph_kchars.graphml')

grafos = {
    'Sentença': G_sent,
    'Parágrafo': G_para,
    'k-chars (500)': G_kchars
}

for nome, G in grafos.items():
    print(f"{nome}: {G.number_of_nodes()} nós, {G.number_of_edges()} arestas")
```
*Output esperado: os três grafos com tamanhos crescentes (sentença < k-chars < parágrafo).*

---

### Seção 2 — Métricas estruturais básicas

**Célula 4 — Markdown**
```
## Métricas estruturais
Calculamos as principais propriedades estruturais de cada grafo, conforme
estudado na disciplina: densidade, grau médio, componentes conectados e diâmetro.
```

**Célula 5 — Código: calcular métricas**
```python
def compute_metrics(G, name):
    # Componente gigante
    gcc_nodes = max(nx.connected_components(G), key=len)
    GCC = G.subgraph(gcc_nodes).copy()
    
    metrics = {
        'Estratégia': name,
        'Nós (V)': G.number_of_nodes(),
        'Arestas (E)': G.number_of_edges(),
        'Densidade': round(nx.density(G), 4),
        'Grau médio': round(sum(dict(G.degree()).values()) / G.number_of_nodes(), 2),
        'Componentes': nx.number_connected_components(G),
        'Tamanho GCC': len(GCC),
        'Diâmetro GCC': nx.diameter(GCC),
        'Dist. média GCC': round(nx.average_shortest_path_length(GCC), 3),
        'Clustering médio': round(nx.average_clustering(G), 4),
    }
    return metrics

resultados = [compute_metrics(G, nome) for nome, G in grafos.items()]
df_metrics = pd.DataFrame(resultados).set_index('Estratégia')
df_metrics
```
*Output esperado: tabela comparativa. Esta tabela vai para o relatório e para os slides.*

**Célula 6 — Código: salvar métricas**
```python
df_metrics.to_csv('../results/metrics_summary.csv')
print("Métricas salvas em results/metrics_summary.csv")
```

---

### Seção 3 — Análise de grau

**Célula 7 — Markdown**
```
## Distribuição de grau
A distribuição de grau revela a heterogeneidade da rede: redes com cauda longa
(poucos hubs muito conectados) seguem padrões característicos de redes do mundo real.
```

**Célula 8 — Código: distribuição de grau (escala log-log)**
```python
fig, axes = plt.subplots(1, 3, figsize=(16, 5))

for ax, (nome, G) in zip(axes, grafos.items()):
    degrees = [d for _, d in G.degree()]
    degree_counts = Counter(degrees)
    ks = sorted(degree_counts.keys())
    ps = [degree_counts[k] / G.number_of_nodes() for k in ks]
    
    ax.scatter(ks, ps, alpha=0.7, s=30, color='steelblue')
    ax.set_xscale('log')
    ax.set_yscale('log')
    ax.set_title(f'Distribuição de Grau\n({nome})')
    ax.set_xlabel('Grau k')
    ax.set_ylabel('P(k)')
    ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('../figures/07_degree_distribution.png', dpi=150, bbox_inches='tight')
plt.show()
```

**Célula 9 — Código: top-15 nós por grau**
```python
for nome, G in grafos.items():
    top = sorted(G.degree(), key=lambda x: x[1], reverse=True)[:15]
    print(f"\n=== Top 15 por grau — {nome} ===")
    for node, deg in top:
        print(f"  {node:30s} grau={deg}")
```
*Output esperado: os hubs de cada grafo. Provavelmente Python, Machine Learning, TensorFlow aparecem no topo.*

---

### Seção 4 — Componentes conectados

**Célula 10 — Markdown**
```
## Componentes conectados
Analisamos como o grafo se fragmenta: quantos componentes existem, qual o tamanho
do componente gigante (GCC) e o que os componentes pequenos representam.
```

**Célula 11 — Código: distribuição de tamanho dos componentes**
```python
fig, axes = plt.subplots(1, 3, figsize=(16, 5))

for ax, (nome, G) in zip(axes, grafos.items()):
    comp_sizes = sorted([len(c) for c in nx.connected_components(G)], reverse=True)
    
    ax.bar(range(len(comp_sizes)), comp_sizes, color='coral', alpha=0.8)
    ax.set_title(f'Tamanho dos Componentes\n({nome})')
    ax.set_xlabel('Componente (ordenado por tamanho)')
    ax.set_ylabel('Número de nós')
    
    # Destaca o GCC
    ax.bar(0, comp_sizes[0], color='darkred', label=f'GCC: {comp_sizes[0]} nós')
    ax.legend(fontsize=9)

plt.tight_layout()
plt.savefig('../figures/10_components.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

### Seção 5 — Coeficiente de clustering

**Célula 12 — Markdown**
```
## Coeficiente de clustering
O clustering mede o fechamento triádico: quantos vizinhos de um nó também são
vizinhos entre si. Alto clustering indica clusters temáticos coesos.
```

**Célula 13 — Código: clustering por nó e top clusterers**
```python
for nome, G in grafos.items():
    clust = nx.clustering(G)
    # Filtra nós com grau > 2 para clustering ser significativo
    clust_filtered = {n: c for n, c in clust.items() if G.degree(n) > 2}
    top_clust = sorted(clust_filtered.items(), key=lambda x: x[1], reverse=True)[:10]
    
    print(f"\n=== Top 10 clustering — {nome} (grau > 2) ===")
    for node, c in top_clust:
        print(f"  {node:30s} C={c:.3f}, grau={G.degree(node)}")
```

**Célula 14 — Código: histograma de clustering**
```python
fig, axes = plt.subplots(1, 3, figsize=(16, 5))

for ax, (nome, G) in zip(axes, grafos.items()):
    clust_vals = list(nx.clustering(G).values())
    ax.hist(clust_vals, bins=20, color='mediumseagreen', edgecolor='white')
    ax.axvline(np.mean(clust_vals), color='red', linestyle='--',
               label=f'Média: {np.mean(clust_vals):.3f}')
    ax.set_title(f'Distribuição de Clustering\n({nome})')
    ax.set_xlabel('Coeficiente de clustering C(i)')
    ax.set_ylabel('Número de nós')
    ax.legend()

plt.tight_layout()
plt.savefig('../figures/07b_clustering_distribution.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

### Seção 6 — Visualização do grafo

**Célula 15 — Markdown**
```
## Visualização dos grafos
Visualizamos o componente gigante de cada grafo. O tamanho de cada nó é proporcional
ao seu grau. Usamos layout spring (Fruchterman-Reingold) para melhor legibilidade.
```

**Célula 16 — Código: visualização dos três grafos lado a lado**
```python
fig, axes = plt.subplots(1, 3, figsize=(20, 7))

for ax, (nome, G) in zip(axes, grafos.items()):
    # Pegar apenas o GCC
    gcc_nodes = max(nx.connected_components(G), key=len)
    GCC = G.subgraph(gcc_nodes).copy()
    
    # Limitar a nós com grau mínimo para visualização não ficar congestionada
    min_degree = 3
    nodes_to_show = [n for n, d in GCC.degree() if d >= min_degree]
    subG = GCC.subgraph(nodes_to_show)
    
    pos = nx.spring_layout(subG, seed=42, k=1.5)
    degrees = dict(subG.degree())
    node_sizes = [degrees[n] * 30 for n in subG.nodes()]
    
    nx.draw_networkx_nodes(subG, pos, ax=ax, node_size=node_sizes,
                           node_color='steelblue', alpha=0.8)
    nx.draw_networkx_edges(subG, pos, ax=ax, alpha=0.2, width=0.5)
    
    # Labels apenas para os top-10 nós
    top_nodes = sorted(degrees, key=degrees.get, reverse=True)[:10]
    labels = {n: n for n in top_nodes}
    nx.draw_networkx_labels(subG, pos, labels, ax=ax, font_size=7)
    
    ax.set_title(f'{nome}\n(GCC, grau ≥ {min_degree}, {subG.number_of_nodes()} nós)')
    ax.axis('off')

plt.suptitle('Grafos de Coocorrência de NER — TCCs de Engenharia da Computação',
             fontsize=14, y=1.02)
plt.tight_layout()
plt.savefig('../figures/06_graphs_comparison.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

### Seção 7 — Ego-network de um hub

**Célula 17 — Código: ego-network**
```python
# Escolher o nó de maior grau no grafo por sentença como exemplo
top_node = max(G_sent.degree(), key=lambda x: x[1])[0]
ego = nx.ego_graph(G_sent, top_node, radius=1)

pos = nx.spring_layout(ego, seed=42)
node_sizes = [500 if n == top_node else 200 for n in ego.nodes()]
node_colors = ['red' if n == top_node else 'steelblue' for n in ego.nodes()]

plt.figure(figsize=(10, 8))
nx.draw_networkx(ego, pos,
                 node_size=node_sizes,
                 node_color=node_colors,
                 font_size=8,
                 alpha=0.9,
                 width=0.8)
plt.title(f'Ego-network de "{top_node}" (grafo por sentença)\n'
          f'Vizinhos diretos: {ego.number_of_nodes()-1}')
plt.axis('off')
plt.tight_layout()
plt.savefig('../figures/08_ego_network.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

### Seção 8 — Teste de Small World

**Célula 18 — Markdown**
```
## Verificação de propriedades Small World
Comparamos o grafo gerado com um grafo aleatório equivalente (mesmo V e E).
Em redes small world: clustering_real >> clustering_aleatório, mas distância_real ≈ distância_aleatória.
```

**Célula 19 — Código: comparação com grafo aleatório**
```python
for nome, G in grafos.items():
    gcc_nodes = max(nx.connected_components(G), key=len)
    GCC = G.subgraph(gcc_nodes).copy()
    
    n = GCC.number_of_nodes()
    m = GCC.number_of_edges()
    p = (2 * m) / (n * (n - 1))  # probabilidade equivalente
    
    # Grafo aleatório Erdős–Rényi equivalente
    G_random = nx.erdos_renyi_graph(n, p, seed=42)
    
    # Pegar GCC do aleatório para evitar erro no shortest path
    gcc_rand = max(nx.connected_components(G_random), key=len)
    G_rand_gcc = G_random.subgraph(gcc_rand)
    
    C_real = nx.average_clustering(GCC)
    C_rand = nx.average_clustering(G_rand_gcc)
    L_real = nx.average_shortest_path_length(GCC)
    L_rand = nx.average_shortest_path_length(G_rand_gcc)
    
    print(f"\n=== {nome} ===")
    print(f"  Clustering real:      {C_real:.4f}")
    print(f"  Clustering aleatório: {C_rand:.4f}")
    print(f"  Ratio C_real/C_rand:  {C_real/C_rand:.2f}x")
    print(f"  Distância média real: {L_real:.3f}")
    print(f"  Distância média rand: {L_rand:.3f}")
    print(f"  Ratio L_real/L_rand:  {L_real/L_rand:.2f}x")
    print(f"  → Small World?  C >> C_rand e L ≈ L_rand")
```
*Output esperado: se C_real/C_rand >> 1 e L_real/L_rand ≈ 1, o grafo tem propriedades de small world. Esta análise é o ponto mais sofisticado do trabalho.*

---

## Notebook 04 — `comparison.ipynb`

**Objetivo:** comparação direta e crítica das três estratégias. Enquanto o notebook 03 analisa cada grafo em profundidade, o 04 coloca os três lado a lado e extrai conclusões. Este notebook é o que mais vai aparecer na apresentação.

Pré-requisito: notebook 03 rodado e `results/metrics_summary.csv` gerado.

---

### Seção 1 — Setup

**Célula 1 — Markdown**
```
# Comparação entre Estratégias de Coocorrência
Sentença × Parágrafo × Janela de k caracteres

Este notebook compara as três estratégias de construção do grafo de coocorrência
em termos de estrutura da rede, qualidade das relações geradas e interpretabilidade.
```

**Célula 2 — Código: imports e carregamento**
```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import networkx as nx
import numpy as np

df = pd.read_csv('../results/metrics_summary.csv', index_col='Estratégia')

G_sent   = nx.read_graphml('../graphs/graph_sentence.graphml')
G_para   = nx.read_graphml('../graphs/graph_paragraph.graphml')
G_kchars = nx.read_graphml('../graphs/graph_kchars.graphml')

grafos = {'Sentença': G_sent, 'Parágrafo': G_para, 'k-chars (500)': G_kchars}
```

---

### Seção 2 — Tabela e heatmap de métricas

**Célula 3 — Markdown**
```
## Visão geral comparativa
```

**Célula 4 — Código: exibir tabela formatada**
```python
# Tabela estilizada para o notebook
df.style.background_gradient(cmap='Blues', axis=0)
```

**Célula 5 — Código: heatmap normalizado**
```python
# Normaliza cada coluna 0–1 para comparação visual
df_norm = (df - df.min()) / (df.max() - df.min())

fig, ax = plt.subplots(figsize=(12, 4))
im = ax.imshow(df_norm.values, cmap='YlOrRd', aspect='auto')

ax.set_xticks(range(len(df_norm.columns)))
ax.set_xticklabels(df_norm.columns, rotation=40, ha='right', fontsize=9)
ax.set_yticks(range(len(df_norm.index)))
ax.set_yticklabels(df_norm.index, fontsize=11)

# Adicionar valores originais nas células
for i in range(len(df.index)):
    for j in range(len(df.columns)):
        ax.text(j, i, str(df.iloc[i, j]), ha='center', va='center', fontsize=8)

plt.colorbar(im, ax=ax, label='Valor normalizado (0=mín, 1=máx)')
ax.set_title('Comparação de Métricas — Estratégias de Coocorrência')
plt.tight_layout()
plt.savefig('../figures/09_metrics_comparison.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

### Seção 3 — Comparação das arestas mais pesadas

**Célula 6 — Markdown**
```
## Qualidade das relações: arestas de maior peso
Comparamos os pares de entidades com maior coocorrência em cada estratégia.
Isso permite avaliar qualitativamente se as relações geradas fazem sentido semântico.
```

**Célula 7 — Código: top-15 arestas por peso**
```python
for nome, G in grafos.items():
    edges_sorted = sorted(G.edges(data=True), key=lambda x: x[2].get('weight', 1), reverse=True)
    print(f"\n=== Top 15 pares de maior coocorrência — {nome} ===")
    for u, v, data in edges_sorted[:15]:
        w = data.get('weight', 1)
        print(f"  {u:25s} ↔ {v:25s}  peso={w}")
```
*Esta é a análise qualitativa mais importante. Os pares do grafo por sentença devem ser mais "tight" semanticamente do que os do parágrafo.*

---

### Seção 4 — Comparação de grau dos mesmos nós

**Célula 8 — Markdown**
```
## Como o grau dos nós varia entre estratégias?
Um nó importante na estratégia por sentença continua importante por parágrafo?
Analisamos a correlação de grau para os nós que aparecem nos três grafos.
```

**Célula 9 — Código: correlação de grau entre estratégias**
```python
# Nós em comum nos três grafos
common_nodes = set(G_sent.nodes()) & set(G_para.nodes()) & set(G_kchars.nodes())
print(f"Nós presentes nos três grafos: {len(common_nodes)}")

degree_sent  = {n: G_sent.degree(n)   for n in common_nodes}
degree_para  = {n: G_para.degree(n)   for n in common_nodes}
degree_kchar = {n: G_kchars.degree(n) for n in common_nodes}

df_deg = pd.DataFrame({
    'Sentença': degree_sent,
    'Parágrafo': degree_para,
    'k-chars': degree_kchar
})

fig, axes = plt.subplots(1, 2, figsize=(13, 5))

axes[0].scatter(df_deg['Sentença'], df_deg['Parágrafo'], alpha=0.5, s=30)
axes[0].set_xlabel('Grau (Sentença)')
axes[0].set_ylabel('Grau (Parágrafo)')
axes[0].set_title('Correlação de Grau: Sentença × Parágrafo')
corr1 = df_deg['Sentença'].corr(df_deg['Parágrafo'])
axes[0].text(0.05, 0.92, f'r = {corr1:.3f}', transform=axes[0].transAxes)

axes[1].scatter(df_deg['Sentença'], df_deg['k-chars'], alpha=0.5, s=30, color='coral')
axes[1].set_xlabel('Grau (Sentença)')
axes[1].set_ylabel('Grau (k-chars)')
axes[1].set_title('Correlação de Grau: Sentença × k-chars')
corr2 = df_deg['Sentença'].corr(df_deg['k-chars'])
axes[1].text(0.05, 0.92, f'r = {corr2:.3f}', transform=axes[1].transAxes)

plt.tight_layout()
plt.savefig('../figures/09b_degree_correlation.png', dpi=150, bbox_inches='tight')
plt.show()
```
*Se r > 0.8: os hubs são os mesmos independente da estratégia. Se r baixo: a estratégia muda quem é central.*

---

### Seção 5 — Análise de nós exclusivos

**Célula 10 — Código: nós presentes apenas em uma estratégia**
```python
only_sent  = set(G_sent.nodes()) - set(G_para.nodes()) - set(G_kchars.nodes())
only_para  = set(G_para.nodes()) - set(G_sent.nodes()) - set(G_kchars.nodes())
only_kchar = set(G_kchars.nodes()) - set(G_sent.nodes()) - set(G_para.nodes())

print(f"Nós exclusivos da estratégia por Sentença:  {len(only_sent)}")
print(f"  Exemplos: {list(only_sent)[:10]}")
print(f"\nNós exclusivos da estratégia por Parágrafo: {len(only_para)}")
print(f"  Exemplos: {list(only_para)[:10]}")
print(f"\nNós exclusivos da estratégia k-chars:        {len(only_kchar)}")
print(f"  Exemplos: {list(only_kchar)[:10]}")
```
*Nós exclusivos de uma estratégia tendem a ser entidades raras que só aparecem em contextos muito específicos.*

---

### Seção 6 — Discussão crítica e conclusões

**Célula 11 — Markdown (principal célula de análise do trabalho)**
```
## Discussão crítica comparativa

### Densidade e conectividade
[Preencher com base nos resultados: ex: "O grafo por parágrafo apresentou densidade X vezes maior 
que o grafo por sentença. Isso é esperado: ao ampliar a janela de contexto, mais pares de entidades
passam a coocorrer, mas parte dessas conexões pode ser espúria..."]

### Qualidade das relações
[Analisar os top-15 pares de cada estratégia. Discutir: quais relações fazem mais sentido?
Por que a estratégia por sentença tende a gerar pares mais semanticamente próximos?]

### Hubs e centralidade
[Os nós de maior grau são os mesmos nas três estratégias? A correlação calculada indica que...]

### Propriedades de small world
[Comparar com os resultados do notebook 03. O ratio C_real/C_rand foi de X na sentença
e Y no parágrafo. Isso indica que...]

### Qual estratégia usar e para quê?
- **Sentença**: melhor para identificar relações diretas e específicas entre tecnologias.
  Ideal para criar um grafo de "o que é usado junto com o quê" em Engenharia da Computação.
- **Parágrafo**: melhor para mapear o ecossistema temático mais amplo. 
  Ideal para visualizar quais áreas (IoT, ML, redes) são discutidas em conjunto nos TCCs.
- **k-chars (500)**: intermediário, sensível ao parâmetro k. Útil para explorar diferentes
  granularidades de co-ocorrência de forma controlada.

### Limitações
[Mencionar: modelo NER genérico, ausência de desambiguação, problemas de extração de PDF,
ausência de análise de comunidades formais com Louvain, corpus restrito a 30 TCCs...]
```

**Célula 12 — Código: salvar tabela final em LaTeX (opcional, útil para relatório)**
```python
# Para quem quiser usar a tabela no LaTeX do relatório
print(df.to_latex(float_format="%.4f"))
```

---

## Resumo: o que cada notebook entrega

| Notebook | Entrega principal | Figuras geradas |
|---|---|---|
| 01_corpus | Entendimento do corpus, validação da extração | corpus_stats, volume_texto |
| 02_ner | Validação do NER, distribuição de entidades | tipos, top20, exemplo anotado |
| 03_graph | Análise profunda de cada grafo | grafos, grau, clustering, ego-net, small world |
| 04_comparison | Conclusão crítica comparando as três estratégias | heatmap, correlação de grau, top pares |

A ordem de execução importa: o 03 gera `metrics_summary.csv` que o 04 usa. Os notebooks 01 e 02 são independentes entre si mas devem rodar antes do 03, pois validam os insumos.
