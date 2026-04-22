# Projeto NER

Pipeline em notebooks para:

1. explorar um corpus de dissertações de mestrado da UFRN;
2. extrair entidades nomeadas com GLiNER;
3. construir e analisar grafos de coocorrência.

## Estrutura

```text
projeto_ner/
├── data/
│   ├── extracted_text/        # texto extraído dos PDFs com PyMuPDF
│   └── entities/gliner/       # entidades extraídas pelo Notebook 02
├── docs/                      # anotações e documentação auxiliar
├── figures/                   # figuras geradas pelos notebooks
├── graphs/                    # grafos exportados em GraphML
├── metadados_teses/           # metadados CSV por dissertação
├── notebooks/
│   ├── 01_corpus_exploration.ipynb
│   ├── 02_ner_analysis.ipynb
│   └── 03_graph_analysis.ipynb
├── results/                   # tabelas-resumo geradas no pipeline
├── teses/                     # PDFs brutos (nao versionados)
├── environment.yml
└── requirements.txt
```

## Modelo de NER

- Biblioteca: `gliner`
- Modelo: `urchade/gliner_multi-v2.1`
- Carregamento no notebook: `GLiNER.from_pretrained("urchade/gliner_multi-v2.1")`

O pacote `gliner` fica instalado no ambiente Python usado pelo Jupyter.
Os pesos do modelo nao ficam dentro deste repositorio: eles sao baixados automaticamente
para o cache local do Hugging Face na primeira execucao.

Em macOS/Linux, o cache costuma ficar em:

```text
~/.cache/huggingface/hub/
```

## Ambiente

Criacao do ambiente:

```bash
conda env create -f environment.yml
conda activate ner_clean
python -m ipykernel install --user --name ner_clean --display-name "Python (ner_clean)"
```

## Execucao

Execute os notebooks nesta ordem:

1. `notebooks/01_corpus_exploration.ipynb`
2. `notebooks/02_ner_analysis.ipynb`
3. `notebooks/03_graph_analysis.ipynb`

## Dados brutos

Os PDFs em `teses/` nao sao versionados por padrao.
Para reproduzir o Notebook 01 do zero, coloque os arquivos nesse diretorio com o mesmo padrao de nomes esperado pelo notebook:

```text
mestrado_<id>_*.pdf
```

Os CSVs em `metadados_teses/` preservam os metadados e links de origem.

## O que vai para o GitHub

Este repositorio foi organizado para versionar:

- notebooks;
- metadados;
- textos e entidades ja processados em `data/`;
- resultados em `results/`;
- figuras em `figures/`;
- grafos em `graphs/`.

Por padrao, os PDFs brutos em `teses/` ficam fora do controle de versao.

## Observacao

O fluxo recomendado do Notebook 01 usa apenas `PyMuPDF` para extracao de texto.
Se ainda houver alguma celula experimental antiga com `docling` no fim do notebook,
ela nao faz parte do pipeline principal.
