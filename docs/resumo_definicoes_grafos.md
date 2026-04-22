# Resumo de Definições e Fórmulas de Grafos

## 1. Definição básica de rede / grafo

Em termos gerais, uma **rede** ou **grafo** é um conjunto de elementos chamados **nós** (*nodes* ou *vertices*), junto com um conjunto de conexões entre pares de nós, chamadas **arestas** (*links* ou *edges*). As arestas representam a existência de uma relação entre os elementos representados pelos nós.

### Definição formal
Um grafo pode ser representado por:

\[
G = (V, E)
\]

onde:
- \(V\): conjunto de vértices (nós)
- \(E\): conjunto de arestas

com:

\[
E \subseteq V \times V
\]

Em grafos ponderados, pode-se escrever:

\[
G = (V, E, W)
\]

onde:
- \(W\): conjunto de pesos das arestas, normalmente com \(W \subseteq \mathbb{R}^{+}\)

---

## 2. Nós, arestas e pesos

- **Nó / vértice**: entidade da rede
- **Aresta / link**: conexão entre dois nós
- **Peso**: valor associado a uma aresta, representando intensidade, custo, frequência, distância etc.

---

## 3. Tipos básicos de grafos

### 3.1. Grafo não direcionado
A conexão não possui direção:

\[
(u,v) = (v,u)
\]

Exemplo: amizade mútua.

### 3.2. Grafo direcionado
A conexão possui direção:

\[
(u,v) \neq (v,u)
\]

Exemplo: seguir alguém em rede social.

### 3.3. Grafo não ponderado
As arestas apenas indicam existência ou ausência de conexão.

### 3.4. Grafo ponderado
As arestas possuem pesos associados.

### 3.5. Combinações principais
- não direcionado e não ponderado
- direcionado e não ponderado
- não direcionado e ponderado
- direcionado e ponderado

---

## 4. Complemento de um grafo

O **complemento** de um grafo \(G\) é obtido:
- removendo todas as arestas existentes em \(G\)
- conectando todos os pares de nós que não estavam conectados em \(G\)

---

## 5. Grafos estendidos

## 5.1. Grafo bipartido

Um grafo bipartido possui dois conjuntos disjuntos de nós, por exemplo \(U\) e \(V\), e as arestas só podem ligar nós de conjuntos diferentes.

Exemplos:
- estudantes ↔ disciplinas
- usuários ↔ livros
- professores ↔ departamentos

### Número máximo de arestas em um bipartido
Se os conjuntos têm tamanhos \(N_1\) e \(N_2\):

\[
L_{\max} = N_1 \times N_2
\]

### Projeção em grafos bipartidos
Uma projeção transforma o bipartido em um grafo de apenas um dos conjuntos, ligando dois nós quando eles compartilham vizinhos no outro conjunto.

---

## 5.2. Multigraph

Um **multigrafo** permite múltiplas arestas entre o mesmo par de nós.

---

## 5.3. Multilayer graph

Em um **grafo multicamada**, os mesmos nós podem se conectar em diferentes **camadas** ou **tipos de relação**.

### Representação
\[
G = (V, E, L)
\]

onde:
- \(V\): nós
- \(E\): arestas
- \(L\): conjunto de camadas

Uma aresta pode ser representada como:

\[
(u,v,l)
\]

onde \(l\) indica a camada.

Exemplos de camadas:
- amizade
- relação profissional
- LinkedIn
- Facebook

---

## 5.4. Dynamic graph

Um **grafo dinâmico** representa uma rede que muda ao longo do tempo.

### Representação
\[
G = (G_1, G_2, \dots, G_n)
\]

com:

\[
G_i = (V_i, E_i)
\]

Cada \(G_i\) representa a rede em um instante \(i\).

---

## 6. Subgrafos e subnetworks

Uma **subnetwork** ou **subgrafo** é uma parte da rede original obtida a partir de um subconjunto de nós e das arestas entre eles.

### Ego network
É um tipo especial de subgrafo formado por:
- um nó central (**ego**)
- seus vizinhos
- e, dependendo da convenção, as conexões entre esses vizinhos

Muito usado em análise de redes sociais.

---

## 7. Densidade e esparsidade

## 7.1. Número máximo de arestas

### Grafo não direcionado
\[
L_{\max} = \binom{N}{2} = \frac{N(N-1)}{2}
\]

### Grafo direcionado
\[
L_{\max}^{directed} = N(N-1)
\]

### Rede completa
Uma rede é **completa** quando todos os pares possíveis de nós estão conectados.

---

## 7.2. Densidade

A **densidade** de uma rede é a fração das arestas possíveis que realmente existem.

### Fórmula geral
\[
d = \frac{L}{L_{\max}}
\]

onde:
- \(L\): número real de arestas
- \(L_{\max}\): número máximo possível de arestas

### Para grafos não direcionados
\[
d = \frac{2L}{N(N-1)}
\]

### Para grafos direcionados
\[
d = \frac{L}{N(N-1)}
\]

### Interpretação
- \(d = 1\): rede completa
- \(d < 1\): rede incompleta
- \(d \ll 1\): rede muito esparsa

---

## 7.3. Esparsidade e densidade assintótica

- A rede é **esparsa** quando:

\[
L \sim N
\]

ou cresce mais devagar.

- A rede é **densa** quando:

\[
L \sim N^2
\]

ou cresce quadraticamente com o tamanho da rede.

---

## 8. Grau e força dos nós

## 8.1. Grau em grafo não direcionado

O **grau** de um nó, \(k_i\), é o número de arestas incidentes nele.

---

## 8.2. Grau em grafo direcionado

- **in-degree**: número de arestas que entram no nó
- **out-degree**: número de arestas que saem do nó

---

## 8.3. Strength em grafos ponderados

Quando há pesos, usa-se a **força** do nó:

- \(s_i\): soma dos pesos das arestas do nó
- \(s_{in}\): soma dos pesos que entram
- \(s_{out}\): soma dos pesos que saem

---

## 8.4. Grau médio da rede

A média dos graus é:

\[
\langle k \rangle = \frac{\sum_i k_i}{N}
\]

Para redes não direcionadas:

\[
\langle k \rangle = \frac{2L}{N}
\]

porque cada aresta contribui para o grau de dois nós.

### Relação entre grau médio e densidade
\[
\langle k \rangle = d(N-1)
\]

e portanto:

\[
d = \frac{\langle k \rangle}{N-1}
\]

onde o grau máximo possível é:

\[
k_{\max} = N-1
\]

---

## 8.5. Distribuição de grau

A **distribuição de grau** mostra quantos nós possuem cada valor de grau.

Em termos probabilísticos, pode-se escrever \(p(k)\) como a fração de nós com grau \(k\).

Ela ajuda a entender a heterogeneidade da rede.

---

## 9. Walks, paths e distâncias

## 9.1. Walk
Uma **walk** é uma sequência de nós conectados por arestas. Dependendo do contexto, pode repetir nós e arestas.

## 9.2. Path
Um **path** é uma sequência de nós conectados sem repetição de nós.

### Comprimento
O comprimento de uma walk ou de um path é o número de arestas percorridas.

---

## 9.3. Caminho mínimo e distância

A **distância** entre dois nós é o menor número de arestas que precisam ser percorridas para conectá-los por um caminho.

Esse caminho é chamado de **shortest path**.

A distância entre \(i\) e \(j\) é normalmente denotada por:

\[
l_{ij}
\]

Em grafos ponderados, o caminho mínimo é o de **menor soma de pesos**, não necessariamente o de menor número de arestas.

---

## 9.4. Distância média

O **average shortest path length** é a média das distâncias mínimas entre todos os pares de nós.

\[
\langle l \rangle = \frac{\sum_{ij} l_{ij}}{\binom{N}{2}}
\]

ou equivalentemente:

\[
\langle l \rangle = \frac{2\sum_{ij} l_{ij}}{N(N-1)}
\]

---

## 9.5. Diâmetro da rede

O **diâmetro** é a maior distância mínima entre quaisquer dois nós da rede:

\[
l_{\max} = \max_{ij} l_{ij}
\]

É o “pior caso” de distância dentro da rede.

---

## 10. Matrizes e caminhos

## 10.1. Matriz de adjacência

A matriz de adjacência \(A\) representa o grafo como uma matriz:
- linhas e colunas correspondem aos nós
- \(A_{ij}=1\) se existe aresta entre \(i\) e \(j\)
- \(A_{ij}=0\) caso contrário

## 10.2. Potências da matriz

A entrada \((i,j)\) de \(A^2\) indica o número de **walks de comprimento 2** entre \(i\) e \(j\).

De forma geral, \(A^k\) fornece o número de walks de comprimento \(k\) entre pares de nós.

---

## 11. Conectividade

## 11.1. Connected component

Se dois nós não podem ser conectados por uma walk, eles pertencem a **componentes conexos** diferentes.

Um componente conexo é um subgrafo em que qualquer nó pode ser alcançado a partir de qualquer outro.

---

## 11.2. Giant Connected Component (GCC)

Em redes grandes, o maior componente conexo costuma ser chamado de **Giant Connected Component (GCC)**.

É o grande bloco principal da rede.

---

## 11.3. Conectividade em grafos direcionados

### Weakly connected
Um grafo direcionado é **fracamente conectado** se, ao ignorar a direção das arestas, ele se torna conectado.

### Strongly connected
Um grafo direcionado é **fortemente conectado** se, respeitando as direções, qualquer nó alcança qualquer outro.

### SCC e WCC
- **SCC**: *Strongly Connected Components*
- **WCC**: *Weakly Connected Components*

---

## 12. Small Worlds

Uma rede do tipo **small world** é aquela em que, mesmo sendo grande, os nós costumam estar a pequenas distâncias uns dos outros.

Exemplos discutidos:
- redes sociais
- redes de coautoria
- redes de atores
- Wikipédia

Relacionados a ideias como:
- “seis graus de separação”
- número de Erdős
- número de Bacon

---

## 13. BFS e DFS

## 13.1. Breadth-First Search (BFS)

A **busca em largura** explora a rede por camadas:
- visita primeiro os vizinhos imediatos
- depois os vizinhos desses vizinhos
- e assim por diante

### Estrutura usada
Fila (**FIFO**)

### Propriedade importante
Em grafos não ponderados, encontra o **menor caminho em número de passos**.

### Complexidade
\[
O(V + E)
\]

---

## 13.2. Depth-First Search (DFS)

A **busca em profundidade** segue um ramo até o máximo antes de voltar.

### Estrutura usada
Pilha (**LIFO**) ou recursão

### Propriedade
Não garante o caminho mínimo.

### Complexidade
\[
O(V + E)
\]

---

## 14. Friend of a Friend, tríades e triângulos

## 14.1. Triad
Uma tríade aberta de três nós.

## 14.2. Triangle
Uma tríade fechada, em que os três nós estão conectados entre si.

Esse é o princípio de “friend of a friend”.

---

## 15. Coeficiente de clustering

O **clustering coefficient** mede o fechamento triádico ao redor de um nó.

Para um nó \(i\):

\[
C(i) = \frac{\tau(i)}{\tau_{\max}(i)}
\]

onde:
- \(\tau(i)\): número de triângulos envolvendo o nó \(i\)
- \(\tau_{\max}(i)\): número máximo possível de triângulos envolvendo \(i\)

Como:

\[
\tau_{\max}(i)=\binom{k_i}{2}
\]

também se pode escrever:

\[
C(i)=\frac{\tau(i)}{\binom{k_i}{2}}=\frac{2\tau(i)}{k_i(k_i-1)}
\]

### Interpretação
- \(C(i)=1\): todos os vizinhos do nó se conhecem
- \(C(i)=0\): nenhum vizinho do nó se conecta aos demais

### Clustering médio
A média dos coeficientes locais pode ser escrita como:

\[
C = \frac{\sum_{i:k_i>1} C(i)}{N_{k>1}}
\]

onde \(N_{k>1}\) é o número de nós com grau maior que 1.

---

## 16. Conceitos aplicados com NetworkX

Ao longo das aulas, foram mostradas funções importantes do NetworkX para análise de grafos, incluindo:

### Caminhos e distâncias
- `nx.has_path(G, u, v)`
- `nx.shortest_path(G, u, v)`
- `nx.shortest_path_length(G, u, v)`
- `nx.average_shortest_path_length(G)`

### Conectividade
- `nx.is_connected(G)`
- `nx.connected_components(G)`
- `nx.number_connected_components(G)`
- `nx.node_connected_component(G, n)`

### Grafos direcionados
- `nx.is_strongly_connected(G)`
- `nx.is_weakly_connected(G)`
- `nx.strongly_connected_components(G)`
- `nx.weakly_connected_components(G)`

### Triângulos e clustering
- `nx.triangles(G)`
- `nx.clustering(G)`
- `nx.average_clustering(G)`

---

## 17. Exemplos clássicos discutidos

- **Pontes de Königsberg**: origem histórica da teoria de grafos
- **Coauthorship networks**: redes de coautoria
- **Número de Erdős**: distância até Paul Erdős na rede de coautoria
- **Oracle of Bacon**: distância entre atores na rede de filmes
- **WikiGame**: navegação em rede de páginas da Wikipédia

---

## 18. Resumo final

Os conceitos mais importantes estudados foram:

- definição de grafo
- nós, arestas e pesos
- grafos direcionados e não direcionados
- grafos ponderados e não ponderados
- complemento de grafo
- bipartite graph
- multigraph
- multilayer graph
- dynamic graph
- subnetwork e ego network
- densidade, esparsidade e rede completa
- grau, força, grau médio e distribuição de grau
- walks, paths e shortest path
- distância média e diâmetro
- connected components, SCC, WCC e GCC
- BFS e DFS
- triads, triangles e clustering coefficient
