# ============================================================
# PROJETO FINAL - ANÁLISE DE VENDAS DE SUPERMERCADO
# ============================================================


# ============================================================
# 1. IMPORTAÇÃO DAS BIBLIOTECAS
# ============================================================

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import plotly.express as px


# ============================================================
# 2. LEITURA DA BASE DE DADOS
# ============================================================

arquivo = "MODULO7_PROJETOFINAL_BASE_SUPERMERCADO - MODULO7_PROJETOFINAL_BASE_SUPERMERCADO (1).csv.csv"

df = pd.read_csv(arquivo)

print("Base carregada com sucesso!")

df.head()


# ============================================================
# 3. CONHECENDO A BASE DE DADOS
# ============================================================

print("Dimensões da base:", df.shape)

print("\nColunas:")
print(df.columns.tolist())

print("\nTipos de dados:")
print(df.dtypes)

print("\nValores nulos:")
print(df.isnull().sum())


# ============================================================
# 4. MÉDIA, MEDIANA E DESVIO PADRÃO POR CATEGORIA
# ============================================================

estatisticas_categoria = (
    df.groupby("Categoria")["Preco_Normal"]
    .agg(
        Media="mean",
        Mediana="median",
        Desvio_Padrao="std",
        Quantidade="count"
    )
    .reset_index()
)

estatisticas_categoria["Media"] = (
    estatisticas_categoria["Media"].round(2)
)

estatisticas_categoria["Mediana"] = (
    estatisticas_categoria["Mediana"].round(2)
)

estatisticas_categoria["Desvio_Padrao"] = (
    estatisticas_categoria["Desvio_Padrao"].round(2)
)

print("Estatísticas por categoria:")

display(
    estatisticas_categoria.sort_values(
        "Media",
        ascending=False
    )
)


# ============================================================
# 5. IDENTIFICAÇÃO DAS CATEGORIAS
#    ACIMA OU ABAIXO DA MEDIANA
# ============================================================

estatisticas_categoria["Posicao_Media"] = np.where(
    estatisticas_categoria["Media"] >
    estatisticas_categoria["Mediana"],
    "Média acima da mediana",
    np.where(
        estatisticas_categoria["Media"] <
        estatisticas_categoria["Mediana"],
        "Média abaixo da mediana",
        "Média igual à mediana"
    )
)

print("Comparação entre média e mediana:")

display(
    estatisticas_categoria[
        [
            "Categoria",
            "Media",
            "Mediana",
            "Posicao_Media"
        ]
    ].sort_values(
        "Media",
        ascending=False
    )
)


# ============================================================
# 6. CATEGORIA COM MAIOR DESVIO PADRÃO
# ============================================================

indice_maior_desvio = (
    estatisticas_categoria["Desvio_Padrao"].idxmax()
)

categoria_maior_desvio = (
    estatisticas_categoria
    .loc[indice_maior_desvio, "Categoria"]
)

maior_desvio = (
    estatisticas_categoria
    .loc[indice_maior_desvio, "Desvio_Padrao"]
)

print(
    "Categoria com maior desvio padrão:",
    categoria_maior_desvio
)

print(
    "Maior desvio padrão:",
    maior_desvio
)


# ============================================================
# 7. BOXPLOT DA CATEGORIA COM MAIOR DESVIO PADRÃO
# ============================================================

dados_boxplot = df[
    df["Categoria"] == categoria_maior_desvio
]["Preco_Normal"]

plt.figure(figsize=(10, 6))

sns.boxplot(
    y=dados_boxplot
)

plt.title(
    f"Distribuição do Preço Normal - "
    f"Categoria: {categoria_maior_desvio}"
)

plt.ylabel("Preço Normal")

plt.show()


# ============================================================
# 8. CÁLCULO DO IQR PARA IDENTIFICAÇÃO DOS OUTLIERS
# ============================================================

Q1 = dados_boxplot.quantile(0.25)

Q3 = dados_boxplot.quantile(0.75)

IQR = Q3 - Q1

limite_inferior = Q1 - 1.5 * IQR

limite_superior = Q3 + 1.5 * IQR

print("Q1:", Q1)

print("Q3:", Q3)

print("IQR:", IQR)

print("\nLimite inferior:", limite_inferior)

print("Limite superior:", limite_superior)


# ============================================================
# 9. IDENTIFICAÇÃO DOS OUTLIERS
# ============================================================

outliers = df[
    (df["Categoria"] == categoria_maior_desvio)
    &
    (
        (df["Preco_Normal"] < limite_inferior)
        |
        (df["Preco_Normal"] > limite_superior)
    )
].copy()

print(
    "Quantidade de outliers:",
    len(outliers)
)

print("\nOutliers encontrados:")

display(
    outliers[
        [
            "title",
            "Marca",
            "Preco_Normal",
            "Categoria"
        ]
    ].sort_values(
        "Preco_Normal",
        ascending=False
    )
)


# ============================================================
# 10. ESTATÍSTICAS DOS OUTLIERS
# ============================================================

print(
    "Quantidade de outliers:",
    len(outliers)
)

print(
    "\nMenor valor considerado outlier:",
    outliers["Preco_Normal"].min()
)

print(
    "Maior valor considerado outlier:",
    outliers["Preco_Normal"].max()
)


# ============================================================
# 11. MÉDIA DE DESCONTO POR CATEGORIA
# ============================================================

media_desconto_categoria = (
    df.groupby("Categoria")["Desconto"]
    .mean()
    .sort_values(
        ascending=False
    )
)

print("Média de desconto por categoria:")

display(
    media_desconto_categoria.round(2)
)


# ============================================================
# 12. GRÁFICO DE BARRAS
#     MÉDIA DE DESCONTO POR CATEGORIA
# ============================================================

plt.figure(figsize=(12, 6))

media_desconto_categoria.plot(
    kind="bar"
)

plt.title(
    "Média de Desconto por Categoria"
)

plt.xlabel("Categoria")

plt.ylabel("Média de Desconto")

plt.xticks(
    rotation=45,
    ha="right"
)

plt.tight_layout()

plt.show()


# ============================================================
# 13. AGRUPAMENTO POR CATEGORIA E MARCA
# ============================================================

categoria_marca = (
    df.groupby(
        [
            "Categoria",
            "Marca"
        ]
    )
    .agg(
        Media_Desconto=(
            "Desconto",
            "mean"
        ),
        Quantidade_Produtos=(
            "Desconto",
            "count"
        )
    )
    .reset_index()
)

categoria_marca["Media_Desconto"] = (
    categoria_marca["Media_Desconto"]
    .round(2)
)

print(
    "Média de desconto por categoria e marca:"
)

display(
    categoria_marca.head(20)
)


# ============================================================
# 14. MAPA INTERATIVO - TREEMAP
#     CATEGORIA → MARCA
# ============================================================

fig = px.treemap(
    categoria_marca,
    path=[
        "Categoria",
        "Marca"
    ],
    values="Media_Desconto",
    color="Media_Desconto",
    hover_data={
        "Media_Desconto": ":.2f",
        "Quantidade_Produtos": True
    },
    title=(
        "Mapa Interativo - "
        "Média de Desconto por Categoria e Marca"
    )
)

fig.update_layout(
    margin=dict(
        t=50,
        l=10,
        r=10,
        b=10
    )
)

fig.show()


# ============================================================
# 15. RANKING DAS MARCAS COM MAIOR DESCONTO
# ============================================================

ranking_marcas = (
    categoria_marca
    .sort_values(
        "Media_Desconto",
        ascending=False
    )
)

print(
    "Top 20 combinações de categoria e marca "
    "com maior média de desconto:"
)

display(
    ranking_marcas.head(20)
)


# ============================================================
# 16. PRINCIPAIS RESULTADOS E CONCLUSÕES
# ============================================================

categoria_maior_media = (
    estatisticas_categoria
    .loc[
        estatisticas_categoria["Media"].idxmax(),
        "Categoria"
    ]
)

maior_media = (
    estatisticas_categoria
    .loc[
        estatisticas_categoria["Media"].idxmax(),
        "Media"
    ]
)

categoria_maior_mediana = (
    estatisticas_categoria
    .loc[
        estatisticas_categoria["Mediana"].idxmax(),
        "Categoria"
    ]
)

maior_mediana = (
    estatisticas_categoria
    .loc[
        estatisticas_categoria["Mediana"].idxmax(),
        "Mediana"
    ]
)

categoria_maior_desconto = (
    media_desconto_categoria.idxmax()
)

maior_desconto = (
    media_desconto_categoria.max()
)

print("=" * 60)

print("PRINCIPAIS RESULTADOS DA ANÁLISE")

print("=" * 60)

print(
    f"\n1. Categoria com maior preço médio:"
    f"\n   {categoria_maior_media}"
    f"\n   Média: {maior_media:.2f}"
)

print(
    f"\n2. Categoria com maior mediana:"
    f"\n   {categoria_maior_mediana}"
    f"\n   Mediana: {maior_mediana:.2f}"
)

print(
    f"\n3. Categoria com maior desvio padrão:"
    f"\n   {categoria_maior_desvio}"
    f"\n   Desvio padrão: {maior_desvio:.2f}"
)

print(
    f"\n4. Categoria com maior média de desconto:"
    f"\n   {categoria_maior_desconto}"
    f"\n   Média de desconto: {maior_desconto:.2f}"
)

print(
    f"\n5. Quantidade de outliers encontrados:"
    f"\n   {len(outliers)}"
)

print("\n" + "=" * 60)
print("ANÁLISE CONCLUÍDA!")
print("=" * 60)
