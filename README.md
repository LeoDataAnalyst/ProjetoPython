# Análise de Despesas em Viagens Públicas - Projeto Python
Descrição do Projeto
Este projeto realiza uma análise de dados de viagens públicas obtidas do Portal da Transparência, com o objetivo de extrair informações valiosas sobre as despesas associadas a diferentes cargos públicos em 2023. O projeto inclui uma análise descritiva para gerar resumos estatísticos e uma análise exploratória focada na identificação e detalhamento de outliers.

## Objetivos
- Consolidar as despesas de viagens públicas por cargo.
- Criar visualizações que permitam uma compreensão rápida dos dados.
- Explorar e detalhar outliers em despesas para identificar padrões e anomalias.

## Requisitos
- Python 3.10+
- Bibliotecas: (pandas, matplotlib, openpyxl (para exportar em Excel))

## Estrutura do Projeto
## 1. Análise Descritiva

### Código para carregamento e preparação dos dados:
~~~
import pandas as pd

# Caminhos para arquivos de entrada e saída
caminho = r"C:\Users\leona\Downloads\2023_20241027_Viagens\2023_Viagem.csv"
caminho_saida_tabela = r"C:\Users\leona\Desktop\ProjetoPython\tabela.xlsx"
caminho_saida_grafico = r"C:\Users\leona\Desktop\ProjetoPython\grafico.png"

# Carregando os dados
df__viagens = pd.read_csv(caminho, encoding='windows-1252', sep=';', decimal=',')

# Limpeza e preparação dos dados
df__viagens['Cargo'] = df__viagens['Cargo'].fillna('Não Identificado')  # Preencher valores nulos
df__viagens['Período - Data de início'] = pd.to_datetime(df__viagens['Período - Data de início'], format='%d/%m/%Y')
df__viagens['Período - Data de fim'] = pd.to_datetime(df__viagens['Período - Data de fim'], format='%d/%m/%Y')


# Cálculos e novas colunas
df__viagens['Despesas'] = df__viagens[['Valor diárias', 'Valor passagens', 'Valor outros gastos']].sum(axis=1)
df__viagens['Mês da viagem'] = df__viagens['Período - Data de início'].dt.month_name()
df__viagens['Duração da viagem (dias)'] = (df__viagens['Período - Data de fim'] - df__viagens['Período - Data de início']).dt.days
~~~
### Código para criação da tabela consolidada:
~~~
# Criando tabela consolidada
df_viagens_consolidas = (
    df__viagens
    .groupby('Cargo')
    .agg(
        despesa_media=('Despesas', 'mean'),
        duracao_media=('Duração da viagem (dias)', 'mean'),
        despesa_total=('Despesas', 'sum'),
        destino_mais_frequente=('Destinos', pd.Series.mode),
        n_viagens=('Nome', 'count')
    )
    .reset_index()
)

# Aplicando um filtro para cargos relevantes (representando mais de 1% do total de viagens)
df_cargos = df__viagens['Cargo'].value_counts(normalize=True).reset_index()
df_cargos.columns = ['Cargo', 'proportion']
cargos_relevantes = df_cargos.loc[df_cargos['proportion'] > 0.01, 'Cargo']
filtro = df_viagens_consolidas['Cargo'].isin(cargos_relevantes)

# Tabela final consolidada e filtrada
df_final = df_viagens_consolidas[filtro]
df_final = df_final.sort_values(by='despesa_media', ascending=False)

# Salvando a tabela final
df_final.to_excel(caminho_saida_tabela, index=False)
~~~
### Código para criação do gráfico de barras horizontal:
~~~
import matplotlib.pyplot as plt

# Criando a figura
fig, ax = plt.subplots(figsize=(16, 6))

# Plotando o gráfico
ax.barh(df_final['Cargo'], df_final['despesa_media'], color='#49deac')
ax.invert_yaxis()

# Ajustando o gráfico
ax.set_facecolor('#3b3b47')
fig.suptitle('Despesa média de viagens por cargo público (2023)')
plt.figtext(0.79, 0.89, 'Fonte: Portal da Transparência', fontsize=8)
plt.grid(color='gray', linestyle='--', linewidth=0.5)
plt.yticks(fontsize=8)
plt.xlabel('Despesa média')

# Salvando o gráfico
plt.savefig(caminho_saida_grafico, bbox_inches='tight')
~~~
![grafico1](https://github.com/user-attachments/assets/f3510729-e148-4c3d-804a-9f4086140830)
## 2. Análise Exploratória
### Código para análise de outliers e criação do gráfico de dispersão:
~~~
# Criando gráfico de dispersão
fig, ax = plt.subplots(figsize=(16, 6))
ax.scatter(df__viagens['Duração da viagem (dias)'], df__viagens['Despesas'], alpha=0.2)
plt.savefig(caminho_saida_grafico2, bbox_inches='tight')

# Filtrando despesas acima de 175 mil para análise detalhada
filtro2 = df__viagens['Despesas'] > 175_000

# Fazendo a união dos dados de passagens e viagens
df_tabela_exploratoria = df_passagens.merge(df__viagens[filtro2])
df_tabela_exploratoria = df_tabela_exploratoria.sort_values(by='Despesas', ascending=False)

# Salvando a tabela detalhada
df_tabela_exploratoria.to_excel(caminho_saida_tabela2, index=False)
~~~
![grafico2](https://github.com/user-attachments/assets/afacc445-0852-4554-b1c4-4cb3787784f5)
## Resultados
- Tabela consolidada: Criada e salva em Excel com estatísticas detalhadas por cargo.
- Gráficos: Gráficos de barras e de dispersão para visualização das despesas e identificação de outliers.
- Análise de outliers: Uma tabela detalhada das viagens com despesas superiores a R$175.000 foi gerada para investigação aprofundada.
## Aprendizados e Insights
- Análise descritiva: Identificação de cargos com maiores despesas médias, contribuindo para decisões de auditoria e otimização.
- Análise exploratória: Detecção de viagens com despesas muito altas, destacando a importância de revisar esses registros.
