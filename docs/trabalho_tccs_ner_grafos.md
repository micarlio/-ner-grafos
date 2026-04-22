# Trabalho 01 — Tema: TCCs de Engenharia da Computação

> Versão em Markdown otimizada a partir do PDF da Unidade 01, mantendo apenas o que é relevante para o tema **TCCs de Engenharia da Computação**.

## 1. Objetivo do trabalho

Construir um projeto de **análise de textos com grafos**, usando **TCCs de Engenharia da Computação** como corpus.

A ideia central é:

1. baixar **N TCCs** em PDF;
2. extrair o texto dos documentos;
3. identificar **entidades nomeadas (NER)** no texto;
4. montar um **grafo de coocorrência** entre essas entidades;
5. aplicar os conceitos estudados na disciplina para **analisar criticamente** o grafo gerado;
6. comparar diferentes estratégias de coocorrência:
   - por **sentença**;
   - por **parágrafo**;
   - por **janela de k caracteres**.

---

## 2. O que já foi estudado na disciplina e deve ser aproveitado

Segundo o material da unidade, os conteúdos já trabalhados incluem:

### Fundamentos de Grafos I
- introdução à teoria de grafos;
- aplicações;
- representações.

### Fundamentos de Grafos II
- probabilidade em grafos;
- matriz de adjacência;
- propriedades estruturais da rede:
  - grau dos nós;
  - densidade;
  - diâmetro.

### Mundos Pequenos
- distribuição de grau;
- distância;
- algoritmos para exploração de grafos:
  - BFS;
  - DFS;
- componentes conectados;
- coeficiente de agrupamento (*clustering*).

### Estudos de caso
- Wikipedia;
- coautoria;
- 6 graus de separação;
- WikiGame.

### Ferramentas
- `networkx`;
- `Gephi`;
- `nxviz`.

---

## 3. Como NLP/LLM entram no trabalho

O material propõe usar **Processamento de Linguagem Natural (NLP)** para transformar texto em uma estrutura de dados no formato de **grafo**.

No seu tema, isso significa usar os textos dos TCCs para:

- extrair entidades relevantes;
- observar quais entidades aparecem juntas;
- representar essas relações em rede.

---

## 4. O que é NER

**NER (Named Entity Recognition)** é uma tarefa de PLN que identifica e classifica entidades no texto em categorias predefinidas.

Exemplos de categorias mostradas no material:

- **PER** — Pessoa;
- **ORG** — Organização;
- **LOC** — Localização;
- **MISC** — Outros termos relevantes, como equipamentos, sensores ou modelos.

### Exemplo conceitual
Na frase:

> “O professor Ivanovitch trabalha na UFRN em Natal e comprou uma NVIDIA Jetson em janeiro de 2026.”

Podemos ter:

- **Ivanovitch** → `PER`
- **UFRN** → `ORG`
- **Natal** → `LOC`
- **NVIDIA Jetson** → `MISC`

---

## 5. O que é o grafo de coocorrência

Depois de extrair as entidades, deve-se construir um **grafo de coocorrência**.

### Ideia
- cada **nó** representa uma entidade;
- uma **aresta** liga duas entidades quando elas aparecem juntas em um mesmo contexto;
- o **peso da aresta** pode representar a frequência dessa coocorrência.

### Possíveis contextos de coocorrência
- **sentença**;
- **parágrafo**;
- **janela de k caracteres**.

Essa comparação é parte explícita do trabalho.

---

## 6. Recorte do seu tema: TCCs de Engenharia da Computação

Para o seu caso, o corpus é formado por **trabalhos de conclusão de curso de Engenharia da Computação**.

O fluxo esperado é:

1. selecionar e baixar **N TCCs**;
2. converter os PDFs em texto;
3. limpar e organizar o texto;
4. aplicar NER;
5. gerar o grafo de coocorrência;
6. analisar o comportamento da rede com os conceitos de grafos estudados;
7. comparar estratégias de construção do grafo.

---

## 7. O que o professor está pedindo, adaptado ao seu tema

### Entregas principais

- download de **N TCCs de Engenharia da Computação**;
- extração do texto dos PDFs;
- geração de **grafo de coocorrência de NER**;
- aplicação do conteúdo já estudado na disciplina;
- **análise crítica** dos resultados;
- **análise de desempenho** entre:
  - sentença;
  - parágrafo;
  - k-caracteres;
- produção de **figuras ilustrativas e representativas**;
- organização de um **repositório estruturado no GitHub**;
- **apresentação assíncrona** de aproximadamente **10 minutos**.

---

## 8. O que é importante analisar no grafo

Como o foco da disciplina é grafos, o trabalho não deve parar em “extrair entidades”.

Você deve usar o grafo gerado para analisar propriedades como:

- entidades mais frequentes e mais conectadas;
- grau dos nós;
- densidade da rede;
- diâmetro;
- componentes conectados;
- distância entre entidades;
- distribuição de grau;
- coeficiente de agrupamento;
- diferenças estruturais entre grafos gerados por:
  - sentença;
  - parágrafo;
  - k-caracteres.

---

## 9. Comparação esperada: sentença vs parágrafo vs k-caracteres

O enunciado destaca explicitamente a necessidade de comparar abordagens.

### Sentença
- contexto mais restrito;
- tende a gerar relações mais específicas;
- pode perder relações que aparecem diluídas ao longo do texto.

### Parágrafo
- contexto mais amplo;
- pode capturar melhor a continuidade temática;
- também pode introduzir mais conexões fracas ou genéricas.

### Janela de k caracteres
- contexto controlado artificialmente;
- permite testar granularidades diferentes;
- depende fortemente da escolha de `k`.

O esperado é que você **não apenas implemente**, mas também **discuta criticamente** como cada estratégia afeta o grafo final.

---

## 10. Figuras que fazem sentido no seu trabalho

Para o seu tema, as figuras mais úteis tendem a ser:

- fluxo da metodologia;
- exemplo de texto com entidades reconhecidas;
- exemplo de subgrafo de coocorrência;
- visualização do grafo completo ou de uma subrede relevante;
- comparação visual entre:
  - grafo por sentença;
  - grafo por parágrafo;
  - grafo por k-caracteres;
- gráficos de métricas da rede:
  - distribuição de grau;
  - componentes;
  - clustering;
  - densidade.

---

## 11. Estrutura mínima esperada do repositório

Uma organização coerente no GitHub pode seguir esta ideia:

```text
repo/
├── data/
│   ├── raw_pdfs/
│   ├── extracted_text/
│   └── processed/
├── notebooks/
├── src/
│   ├── download/
│   ├── extraction/
│   ├── ner/
│   ├── graph/
│   └── analysis/
├── figures/
├── results/
├── README.md
└── requirements.txt
```

---

## 12. Resumo prático do que você realmente precisa fazer

### Etapa 1 — Corpus
Baixar **N TCCs** de Engenharia da Computação.

### Etapa 2 — Texto
Extrair o texto dos PDFs.

### Etapa 3 — Entidades
Aplicar **NER** para detectar pessoas, instituições, locais e outros termos relevantes.

### Etapa 4 — Grafo
Construir grafos de coocorrência com diferentes critérios:
- sentença;
- parágrafo;
- k-caracteres.

### Etapa 5 — Análise
Aplicar métricas e conceitos de grafos estudados na disciplina.

### Etapa 6 — Comparação crítica
Comparar as três estratégias e discutir:
- qualidade das relações geradas;
- comportamento estrutural do grafo;
- vantagens e limitações de cada abordagem.

### Etapa 7 — Entrega
Produzir:
- repositório organizado;
- figuras;
- resultados;
- apresentação em vídeo.

---

## 13. Essência do trabalho em uma frase

O trabalho pede que você transforme um conjunto de **TCCs de Engenharia da Computação** em uma **rede de entidades nomeadas**, e use essa rede para aplicar, demonstrar e discutir os conceitos de **teoria de grafos** estudados na disciplina.
