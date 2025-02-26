![houserocketreadme](https://user-images.githubusercontent.com/67663958/236849842-c184192e-57ac-4701-9ed0-4ff15856d243.png)

# RESUMO
A House Rocket é uma plataforma digital que tem como modelo de negócio, a compra e a venda de imóveis usando tecnologia. A plataforma necessita de uma melhor forma para buscar boas oportunidades de compra de casas em Seattle.

# INTRODUÇÃO
 A aplicação possibilita a utilização de filtro para facilitar a busca de casas por meio do preço, número de banheiros, quartos e se é beira-mar para que essas casas sejam demonstradas em um mapa que divide as áreas por média de preço e mostra as informações de cada casa junto a uma tabela.

# SOLUÇÃO
Retirando os dados do kaggle (https://www.kaggle.com/shivachandel/kc-house-data), e importando no Jupyter Notebook para fazer a análise exploratória de dados, é possível criar insights para um melhor desenvolvimento da solução e descobrir dados que tendem a ser desnecessários para a aplicação. Aplicando a limpeza e transformação dos dados, que irão aprimorar o que necessitamos para resolver o problema, a partir do pycharm, e selecionando os melhores atributos, levando em conta o que é pedido, é possível construir um relatório utilizando mapa e filtros que sejam colocados em produção no Streamlit, assim, facilitando a tomada de decisão. Os dados retirados para a construção do mapa (https://bit.ly/3nwBXha).

**Código da solução:** https://bit.ly/44mtXQf

**Notebook kaggle:** https://www.kaggle.com/code/hedvaldocosta/house-rocket

**Dashboard:** https://app.powerbi.com/view?r=eyJrIjoiZTAzOGI1ZjEtOWZlZS00ZDdlLTgzOTctMWQ2MDQ4NzMzZTBkIiwidCI6ImU0NmMwZTRiLThhY2YtNDMyZC05NjE3LWM4ZWJmNDdlMjY3NSJ9

**Aplicativo:** https://houserocket-dsqfawea9t3qlsjiocv4zc.streamlit.app/

![2023-05-03-10-50-36](https://user-images.githubusercontent.com/67663958/235936276-3fa83d82-2a12-47a9-bf39-42f0509ffbb6.gif)


# FERRAMENTAS
Python 3.9.13

PyCharm Community Edition 2022.1.3

folium 0.12.1

geopandas 0.10.2

numpy 1.20.3

pandas 1.2.4

plotly 5.4.0

streamlit 1.5.1

streamlit-folium 0.5.0

# SOBRE O CÓDIGO
```python
# pandas foi utilizado para carregar o arquivo .csv
import pandas as pd
# numpy foi utilizado para transformação dos dados
import numpy as np
# Construir a aplicação web
import streamlit as st
# Criação do mapa
import folium
# Carregamento do arquivo .geojson
import geopandas
# Demonstração do mapa na aplicação web
from streamlit_folium import folium_static
# Acrescentação de pontos e informações no mapa
from folium.plugins import MarkerCluster
```

```python
# Classe criada para acoplar as funções de ETL (extract, transform, load)
class HouseRocket:
    def __init__(self, file, geojsonfile):
        # Carregando os dados .csv e .geojson
        self.file = pd.read_csv(file)
        self.geojsonfile = geopandas.read_file(geojsonfile)
    # Etapa de transformação dos dados .csv
    def data_processing(self):
        # Tratamento dos dados para arredondar os valores nas colunas
        # banheiros e andares. Não seria viável utilizar valores float
        self.file['bathrooms'] = np.int64(round(self.file['bathrooms'] - 0.3))
        self.file['floors'] = np.int64(round(self.file['floors'] - 0.3))
        # Mudando os dados da coluna "waterfront" para uma melhor demonstração
        # no filtro "beira-mar" criado na aplicação
        self.file['waterfront'] = self.file['waterfront'].apply(lambda x: 'SIM' if x == 1 else 'NÃO')
        # Pegando apenas o ano de cada casa e aplicando em uma nova coluna
        self.file['age'] = np.int64(
            np.int64(pd.to_datetime(self.file['date']).dt.strftime('%Y')) - self.file['yr_built'])
        # Retirando colunas desnecessárias para a construção da aplicação
        self.file.drop(
            columns=['sqft_living', 'sqft_above', 'sqft_basement', 'yr_renovated', 'sqft_living15', 'sqft_lot15',
                     'date'],
            axis=1, inplace=True)
        # Remoção de outliers
        dados_remove = []
        for c in range(0, len(self.file['bedrooms'])):
            if (self.file['bedrooms'][c] in [0, 7, 8, 9, 10, 11, 33]) | (self.file['bathrooms'][c] in [0, 7, 8]):
                dados_remove.append(c)

        self.file.drop(index=dados_remove, inplace=True)
        return self.file
    # Execução do mapa e aplicação das marcações
    def reading_map(self, datamap):
        # Pegando os dados de latitude e longitude para criar o mapa
        mapa = folium.Map(location=[datamap['lat'].mean(), datamap['long'].mean()])
        # Criação dos marcadores no mapa
        marker_cluster = MarkerCluster().add_to(mapa)
        for name, row in datamap.iterrows():
            folium.Marker(location=[row['lat'], row['long']], popup=f'''
                ID:{row["id"]}
                PREÇO:{row["price"]}
                BEIRA-MAR:{row["waterfront"]}
                BANHEIROS:{row["bathrooms"]}
                QUARTOS:{row["bedrooms"]}
                ''').add_to(marker_cluster)
        # Execução do mapa
        df = datamap[['price', 'zipcode']].groupby('zipcode').mean().reset_index()
                data_map = self.geojsonfile[self.geojsonfile['ZIP'].isin(datamap['zipcode'].tolist())]
                folium.Choropleth(geo_data=data_map, data=df, columns=['zipcode', 'price'], key_on='feature.properties.ZIP',
                                  fill_color='YlOrRd',
                                  fill_opacity=0.9,
                                  line_opacity=0.4).add_to(mapa)
                return folium_static(mapa)
```

```python
# Aplicando os dados para a função "HouseRocket"
file_raw = 'https://raw.githubusercontent.com/HedvaldoCosta/HouseRocket/main/Arquivos/kc_house_data.csv'
geofile = 'https://raw.githubusercontent.com/HedvaldoCosta/HouseRocket/main/Arquivos/Zip_Codes.geojson'
houserocket = HouseRocket(file=file_raw, geojsonfile=geofile)
dataframe = houserocket.data_processing()
```
```python
# Aplicação de filtros, títulos e informações na barra lateral da aplicação
st.sidebar.title('Localização e atributos')
show_dataframe = st.sidebar.checkbox('Mostrar tabela', True)
show_map = st.sidebar.checkbox('Mostrar mapa')
filter_id = st.sidebar.multiselect('ID das casas', dataframe['id'].unique().tolist())
filter_price = st.sidebar.slider('Preço das casas', int(dataframe['price'].min()), int(dataframe['price'].max()),
                                 int(dataframe['price'].mean()))
filter_bedrooms = st.sidebar.selectbox('Número de quartos', dataframe['bedrooms'].sort_values().unique().tolist())
filter_bathrooms = st.sidebar.selectbox('Número de banheiros',
                                        dataframe['bathrooms'].sort_values().unique().tolist())
filter_waterfront = st.sidebar.selectbox('Beira-mar', dataframe['waterfront'].unique().tolist())
```

```python
# Execução do filtro sobre IDs das casas
if filter_id:
    dataframe_st = dataframe.loc[dataframe['id'].isin(filter_id)]
else:
    dataframe_st = dataframe.loc[(dataframe['price'] <= filter_price) &
                                 (dataframe['bedrooms'].isin([filter_bedrooms])) &
                                 (dataframe['bathrooms'].isin([filter_bathrooms])) &
                                 (dataframe['waterfront'].isin([filter_waterfront]))]

filter_map = dataframe_st[['id', 'zipcode', 'lat', 'long', 'price', 'bedrooms', 'bathrooms', 'waterfront']]
# Mostrando apenas a tabela
if (show_map == False) & (show_dataframe == True):
    st.dataframe(dataframe_st)
# Mostrando mapa e tabela (Se o total de casas for 0, demonstrará nada)
elif (show_map == True) & (show_dataframe == True) & (len(dataframe_st) != 0):
    st.dataframe(dataframe_st)
    houserocket.reading_map(datamap=filter_map)
# Mostrando apenas o mapa (Se o total de casas for 0, demonstrará nada)
elif (show_map == True) & (show_dataframe == False) & (len(dataframe_st) != 0):
    houserocket.reading_map(datamap=filter_map)
```

# SUMMARY
House Rocket is a digital platform whose business model is the purchase and sale of properties using technology. The platform needs a better way to search for good homebuying opportunities in Seattle.
# INTRODUCTION
The application makes it possible to use a filter to facilitate the search for houses by price, number of bathrooms, bedrooms and whether it is by the sea so that these houses are shown on a map that divides the areas by average price and shows the information of each house next to a table.
# SOLUTION
Taking data from kaggle (https://www.kaggle.com/shivachandel/kc-house-data), and importing it into Jupyter Notebook to perform exploratory data analysis, it is possible to create insights for better solution development and discover data that tends to be unnecessary for the application. Applying data cleaning and transformation, which will improve what we need to solve the problem, from pycharm, and selecting the best attributes, taking into account what is requested, it is possible to build a report using the map and filters that are placed in production on Streamlit, thus facilitating decision making. The data taken to build the map (https://bit.ly/3nwBXha).

**Solution code:** https://bit.ly/44mtXQf

![2023-05-03-10-50-36](https://user-images.githubusercontent.com/67663958/235936276-3fa83d82-2a12-47a9-bf39-42f0509ffbb6.gif)


# TOOLS
Python 3.9.13

PyCharm Community Edition 2022.1.3

folium 0.12.1

geopandas 0.10.2

numpy 1.20.3

pandas 1.2.4

plotly 5.4.0

streamlit 1.5.1

streamlit-folium 0.5.0

# ABOUT THE CODE
```python
# pandas was used to load the .csv file
import pandas as pd
# numpy was used for data transformation
import numpy as np
# Build the web application
import streamlit as st
# Map creation
import folium
# Loading the .geojson file
import geopandas
# Demo of the map in the web application
from streamlit_folium import folium_static
# Adding points and information on the map
from folium.plugins import MarkerCluster
```

```python
 #Class created to couple ETL functions (extract, transform, load)
class HouseRocket:
    def __init__(self, file, geojsonfile):
        # Loading .csv and .geojson data
        self.file = pd.read_csv(file)
        self.geojsonfile = geopandas.read_file(geojsonfile)
    # .csv data transformation step
    def data_processing(self):
        # Data handling to round the values in the bathroom and floor columns.
        # It would not be feasible to use float values
        self.file['bathrooms'] = np.int64(round(self.file['bathrooms'] - 0.3))
        self.file['floors'] = np.int64(round(self.file['floors'] - 0.3))
        # Changing the "waterfront" column data for a better demonstration in
        # the "seaside" filter created in the application
        self.file['waterfront'] = self.file['waterfront'].apply(lambda x: 'SIM' if x == 1 else 'NÃO')
        # Taking only the year of each house and applying it to a new column
        self.file['age'] = np.int64(
            np.int64(pd.to_datetime(self.file['date']).dt.strftime('%Y')) - self.file['yr_built'])
        # Removing unnecessary columns for building the application
        self.file.drop(
            columns=['sqft_living', 'sqft_above', 'sqft_basement', 'yr_renovated', 'sqft_living15', 'sqft_lot15',
                     'date'],
            axis=1, inplace=True)
        # removal of outliers
        dados_remove = []
        for c in range(0, len(self.file['bedrooms'])):
            if (self.file['bedrooms'][c] in [0, 7, 8, 9, 10, 11, 33]) | (self.file['bathrooms'][c] in [0, 7, 8]):
                dados_remove.append(c)

        self.file.drop(index=dados_remove, inplace=True)
        return self.file
    # Execution of the map and application of markings
    def reading_map(self, datamap):
        # Getting the latitude and longitude data to create the map
        mapa = folium.Map(location=[datamap['lat'].mean(), datamap['long'].mean()])
        # Creating markers on the map
        marker_cluster = MarkerCluster().add_to(mapa)
        for name, row in datamap.iterrows():
            folium.Marker(location=[row['lat'], row['long']], popup=f'''
                ID:{row["id"]}
                PREÇO:{row["price"]}
                BEIRA-MAR:{row["waterfront"]}
                BANHEIROS:{row["bathrooms"]}
                QUARTOS:{row["bedrooms"]}
                ''').add_to(marker_cluster)
        # Map execution
        df = datamap[['price', 'zipcode']].groupby('zipcode').mean().reset_index()
                data_map = self.geojsonfile[self.geojsonfile['ZIP'].isin(datamap['zipcode'].tolist())]
                folium.Choropleth(geo_data=data_map, data=df, columns=['zipcode', 'price'], key_on='feature.properties.ZIP',
                                  fill_color='YlOrRd',
                                  fill_opacity=0.9,
                                  line_opacity=0.4).add_to(mapa)
                return folium_static(mapa)
```

```python
# Applying data to the "HouseRocket" function
file_raw = 'https://raw.githubusercontent.com/HedvaldoCosta/HouseRocket/main/Arquivos/kc_house_data.csv'
geofile = 'https://raw.githubusercontent.com/HedvaldoCosta/HouseRocket/main/Arquivos/Zip_Codes.geojson'
houserocket = HouseRocket(file=file_raw, geojsonfile=geofile)
dataframe = houserocket.data_processing()
```

```python
# Application of filters, titles and information in the sidebar of the application
st.sidebar.title('Localização e atributos')
show_dataframe = st.sidebar.checkbox('Mostrar tabela', True)
show_map = st.sidebar.checkbox('Mostrar mapa')
filter_id = st.sidebar.multiselect('ID das casas', dataframe['id'].unique().tolist())
filter_price = st.sidebar.slider('Preço das casas', int(dataframe['price'].min()), int(dataframe['price'].max()),
                                 int(dataframe['price'].mean()))
filter_bedrooms = st.sidebar.selectbox('Número de quartos', dataframe['bedrooms'].sort_values().unique().tolist())
filter_bathrooms = st.sidebar.selectbox('Número de banheiros',
                                        dataframe['bathrooms'].sort_values().unique().tolist())
filter_waterfront = st.sidebar.selectbox('Beira-mar', dataframe['waterfront'].unique().tolist())
```

```python
# Running the filter on house IDs
if filter_id:
    dataframe_st = dataframe.loc[dataframe['id'].isin(filter_id)]
else:
    dataframe_st = dataframe.loc[(dataframe['price'] <= filter_price) &
                                 (dataframe['bedrooms'].isin([filter_bedrooms])) &
                                 (dataframe['bathrooms'].isin([filter_bathrooms])) &
                                 (dataframe['waterfront'].isin([filter_waterfront]))]

filter_map = dataframe_st[['id', 'zipcode', 'lat', 'long', 'price', 'bedrooms', 'bathrooms', 'waterfront']]
# Showing only the table
if (show_map == False) & (show_dataframe == True):
    st.dataframe(dataframe_st)
# Showing map and table (If total houses is 0, it will show nothing)
elif (show_map == True) & (show_dataframe == True) & (len(dataframe_st) != 0):
    st.dataframe(dataframe_st)
    houserocket.reading_map(datamap=filter_map)
# Showing only the map (If total houses is 0, it will show nothing)
elif (show_map == True) & (show_dataframe == False) & (len(dataframe_st) != 0):
    houserocket.reading_map(datamap=filter_map)
```
