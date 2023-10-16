### Hi there 👋

!pip install dash
!pip install dash-core-components
!pip install dash-html-components
!pip install plotly
!pip install scikit-learn

import pandas as pd
from openpyxl import load_workbook
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output
from sklearn.ensemble import RandomForestRegressor
import plotly.express as px

from google.colab import files
uploaded = files.upload()

# Obtén el nombre del archivo subido
import io

filename = 'Data Oficial Pronosticos Sell Out (1).xlsx'

if filename in uploaded:
    # Lee todas las hojas del archivo Excel
    all_sheets = pd.read_excel(io.BytesIO(uploaded[filename]), sheet_name=None)

    # Inicializa un DataFrame vacío para almacenar todos los datos
    all_data = pd.DataFrame()

    # Itera sobre cada hoja y concatena los datos en all_data
    for sheet_name, data in all_sheets.items():
        all_data = pd.concat([all_data, data])
else:
    print(f"No se encontró el archivo {filename}. Asegúrate de haber subido el archivo correct
#DETALLE POR CATEGORIA
import pandas as pd
import dash
from dash import dcc, html
from dash.dependencies import Input, Output
from statsmodels.tsa.statespace.sarimax import SARIMAX
import plotly.express as px

# 2. Preprocesamiento y Agrupamiento
all_data['Fechas'] = pd.to_datetime(all_data['Fechas'])
all_data['Costo B2B'] = pd.to_numeric(all_data['Costo B2B'], errors='coerce')
grouped_data = all_data.groupby(['Fechas', 'Cadena', 'Sub Cadena', 'Categoría', 'Código Interno'])['Costo B2B'].sum().reset_index()


app = dash.Dash(__name__)

# Asumimos que 'grouped_data' ya está definido y preprocesado
app.layout = html.Div([
    dcc.Dropdown(id='subcadena-dropdown', options=[{'label': i, 'value': i} for i in grouped_data['Sub Cadena'].unique()], value=grouped_data['Sub Cadena'].iloc[0]),
    dcc.Dropdown(id='categoria-dropdown'),
    dcc.Dropdown(id='sku-dropdown'),
    dcc.Graph(id='forecast-graph')
])

@app.callback(
    Output('categoria-dropdown', 'options'),
    Input('subcadena-dropdown', 'value')
)
def update_categoria_dropdown(selected_subcadena):
    filtered_data = grouped_data[grouped_data['Sub Cadena'] == selected_subcadena]
    categories = [{'label': i, 'value': i} for i in filtered_data['Categoría'].unique()]
    return categories

@app.callback(
    Output('sku-dropdown', 'options'),
    [Input('subcadena-dropdown', 'value'),
     Input('categoria-dropdown', 'value')]
)
def update_sku_dropdown(selected_subcadena, selected_categoria):
    filtered_data = grouped_data[
        (grouped_data['Sub Cadena'] == selected_subcadena) &
        (grouped_data['Categoría'] == selected_categoria)
    ]
    skus = [{'label': i, 'value': i} for i in filtered_data['Código Interno'].unique()]
    return skus

@app.callback(
    Output('forecast-graph', 'figure'),
    [Input('subcadena-dropdown', 'value'),
     Input('categoria-dropdown', 'value'),
     Input('sku-dropdown', 'value')]
)
def update_graph(selected_subcadena, selected_categoria, selected_sku):
    filtered_data = grouped_data[
        (grouped_data['Sub Cadena'] == selected_subcadena) &
        (grouped_data['Categoría'] == selected_categoria)
    ]

    if selected_sku:
        filtered_data = filtered_data[filtered_data['Código Interno'] == selected_sku]

    if filtered_data.empty:
        raise dash.exceptions.PreventUpdate("No data available for the selected combination")

    filtered_data.set_index('Fechas', inplace=True)
    weekly_data = filtered_data['Costo B2B'].resample('W').sum().reset_index()
    weekly_data.columns = ['ds', 'y']

    order = (1, 1, 1)
    seasonal_order = (1, 1, 1, 52)
    model = SARIMAX(weekly_data['y'], order=order, seasonal_order=seasonal_order, enforce_stationarity=False, enforce_invertibility=False)
    results = model.fit(disp=False)

    future_periods = 22
    forecast = results.get_forecast(steps=future_periods).predicted_mean
    future_dates = pd.date_range(weekly_data['ds'].max(), periods=future_periods, freq='W')

    fig = px.line(weekly_data, x='ds', y='y', title='Historical Data and Forecast')
    fig.add_scatter(x=future_dates, y=forecast, mode='lines', name='Forecast')

    return fig

if __name__ == '__main__':
    app.run_server(debug=True)


    all_data


    
