# interactive-stock-index-dashboard-
# Here is my attempt to re-creating an interactive dashboard that visualises stock data from the world major stock indexs from the plotly and dash library. 

import dash 
from dash import dcc
from dash import html 
import dash_html_components as html 
import pandas_datareader.data as web
import datetime
import pandas as pd
import plotly.express as px 
import dash_bootstrap_components as dbc
from dash.dependencies import Input, Output


#three important components in plotly: 1) components, 2) plotly graphs,
#3) the callback: connects the dash components + plot graphs 

start = datetime.datetime(2010, 1, 1)
end = datetime.datetime(2021, 10, 1)

df = web.DataReader(['^UKX', 'DAX', '^DJI', '^SPX', '^NDX', '^_US', '^NKX', '^SHC', '^TWSE'],
                   'stooq', start=start, end=end)
df = df.stack().reset_index()
print(df)

df.to_csv("index-final.csv", index=False) # save index stock data into csv file 

app = dash.Dash(__name__, external_stylesheets=[dbc.themes.LUX],
                meta_tags=[{'name': 'viewport',
                            'content': 'width=device-width, initial-scale=1.0'}]
               )

app.layout = dbc.Container([
    
    dbc.Row(
        dbc.Col(html.H1("Index Data Dashboard",
                       className='text-center text-secondary'),
               width=12) #adding parameters
        
    ), 
    
    dbc.Row([
        dbc.Col([
            dcc.Dropdown(id='my-dpdn', multi=False, value='^UKX',
                         options=[{'label': x, 'value': x}
                                  for x in sorted(df['Symbols'].unique())],
                        ),
                                            
            dcc.Graph(id='line-fig')
        ],  width={'size':5, 'offset':2, 'order':1},
            xs=12, sm=12, md=12, lg=5, xl=5
        ),
        
        dbc.Col([
            dcc.Dropdown(id='my-dpn2', multi=True, value=['DAX', '^DJI'],
                         options=[{'label': x, 'value': x}                         
                                  for x in sorted(df['Symbols'].unique())],
                        ),
            
            dcc.Graph(id='line-fig2')
        ],  width={'size':5, 'offset':2, 'order':1},
            xs=12, sm=12, md=12, lg=5, xl=5
        ),
        
    ], no_gutters=True, justify='start'),
    
    
    dbc.Row([
        dbc.Col([
            html.P("Select Index:",
                  style={"textDecoration": "underline"}),
            dcc.Checklist(id='my-checklist', value=['^SPX', '^UKX', '^NDX'],
                          options=[{'label': x, 'value': x}                         
                                  for x in sorted(df['Symbols'].unique())],
                         
                         labelClassName='mr-3'),
            dcc.Graph(id='my-hist')    
        ],  width={'size':5, 'offset':0},
            xs=12, sm=12, md=12, lg=5, xl=5
        ),
        
        
    ]),
        
    
], fluid=True)


# Callback section: connecting the components

# Line chart - Single
@app.callback(
    Output('line-fig', 'figure'),
    Input('my-dpdn', 'value')
)

def update_graph(stock_slctd):
    dff = df[df['Symbols']==stock_slctd]
    figln = px.line(dff, x='Date', y='Close')
    return figln


# Line chart - multiple
@app.callback(
    Output('line-fig2', 'figure'),
    Input('my-dpn2', 'value')
)

def update_graph(stock_slctd):
    dff = df[df['Symbols'].isin(stock_slctd)]
    figln2 = px.line(dff, x='Date', y='Close', color='Symbols')
    return figln2

# Histogram 
@app.callback(
    Output('my-hist', 'figure'),
    Input('my-checklist', 'value')

)

def update_graph(stock_slctd):
    dff = df[df['Symbols'].isin(stock_slctd)]
    dff = dff[dff['Date']=='2020-10-01']
    fighist = px.histogram(dff, x='Symbols', y='Close')
    return fighist


if __name__ == '__main__':
    app.run_server(debug=True, use_reloader=False)
