import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import dash
from dash import dcc, html
from dash.dependencies import Input, Output

# Load sample data
data = pd.read_csv('drug-overdose-death-rates new.csv')

# Initialize the Dash app
app = dash.Dash(__name__)

app.layout = html.Div([
    html.H1("Drug Overdose Death Rates Dashboard"),
    
     dcc.Dropdown(
        id='year-dropdown',
        options=[{'label': year, 'value': year} for year in data['Year'].unique()],
        value=data['Year'].max(),
        clearable=False,
        style={'width': '50%'}
    ),
    
    dcc.Graph(id='line-plot'),
    dcc.Graph(id='bar-chart'),
    dcc.Graph(id='stacked-bar-chart'),
    dcc.Graph(id='heatmap'),
    dcc.Graph(id='box-plot'),
    
    html.Div([
        html.Label('Select X Axis for Scatter Plot'),
        dcc.Dropdown(
            id='scatter-x',
            options=[{'label': col, 'value': col} for col in data.columns[1:]],
            value='Heroin overdose death rates (CDC WONDER)',
            clearable=False,
            style={'width': '50%'}
        ),
    ]),
    
    html.Div([
        html.Label('Select Y Axis for Scatter Plot'),
        dcc.Dropdown(
            id='scatter-y',
            options=[{'label': col, 'value': col} for col in data.columns[1:]],
            value='Synthetic opioids death rates (CDC WONDER)',
            clearable=False,
            style={'width': '50%'}
        ),
    ]),
    
    dcc.Graph(id='scatter-plot'),
])

@app.callback(
    [Output('line-plot', 'figure'),
     Output('bar-chart', 'figure'),
     Output('stacked-bar-chart', 'figure'),
     Output('heatmap', 'figure'),
     Output('box-plot', 'figure'),
     Output('scatter-plot', 'figure')],
    [Input('year-dropdown', 'value'),
     Input('scatter-x', 'value'),
     Input('scatter-y', 'value')]
)
def update_graphs(selected_year, scatter_x, scatter_y):
    filtered_data = data[data['Year'] == selected_year]
    
    
    # Line Plot
    line_fig = px.line(data, x='Year', y=['Any opioid death rates (CDC WONDER)',
                                          'Cocaine overdose death rates (CDC WONDER)',
                                          'Heroin overdose death rates (CDC WONDER)',
                                          'Synthetic opioids death rates (CDC WONDER)',
                                          'Prescription Opioids death rates (US CDC WONDER)'],
                       labels={'value': 'Death Rates', 'variable': 'Drug Type'},
                       title='Trends in Drug Overdose Death Rates')

    # Bar Chart
    bar_data = filtered_data.melt(id_vars=['Year'], value_vars=[
        'Any opioid death rates (CDC WONDER)',
        'Cocaine overdose death rates (CDC WONDER)',
        'Heroin overdose death rates (CDC WONDER)',
        'Synthetic opioids death rates (CDC WONDER)',
        'Prescription Opioids death rates (US CDC WONDER)'],
        var_name='Drug Type', value_name='Death Rates')
    
    bar_fig = px.bar(bar_data, x='Drug Type', y='Death Rates', 
                     title=f'Drug Overdose Death Rates in {selected_year}')

    # Stacked Bar Chart
    stacked_bar_fig = px.bar(data, x='Year', y=['Any opioid death rates (CDC WONDER)',
                                                'Cocaine overdose death rates (CDC WONDER)',
                                                'Heroin overdose death rates (CDC WONDER)',
                                                'Synthetic opioids death rates (CDC WONDER)',
                                                'Prescription Opioids death rates (US CDC WONDER)'],
                             title='Drug Overdose Death Rates Over Years',
                             labels={'value': 'Death Rates', 'variable': 'Drug Type'},
                             barmode='stack')

    # Heatmap
    heatmap_data = data.pivot_table(index='Year', values=[
        'Any opioid death rates (CDC WONDER)',
        'Cocaine overdose death rates (CDC WONDER)',
        'Heroin overdose death rates (CDC WONDER)',
        'Synthetic opioids death rates (CDC WONDER)',
        'Prescription Opioids death rates (US CDC WONDER)'])
    
    heatmap_fig = go.Figure(data=go.Heatmap(
        z=heatmap_data.values,
        x=heatmap_data.columns,
        y=heatmap_data.index,
        colorscale='YlGnBu'))
    heatmap_fig.update_layout(title='Heatmap of Drug Overdose Death Rates Over Years',
                              xaxis_title='Drug Type',
                              yaxis_title='Year')
    # Box Plot
    melted_data = filtered_data.melt(id_vars=['Year'], value_vars=[
        'Any opioid death rates (CDC WONDER)',
        'Cocaine overdose death rates (CDC WONDER)',
        'Heroin overdose death rates (CDC WONDER)',
        'Synthetic opioids death rates (CDC WONDER)',
        'Prescription Opioids death rates (US CDC WONDER)'],
        var_name='Drug Type', value_name='Death Rates')
    
    box_fig = px.box(melted_data, x='Drug Type', y='Death Rates', 
                     title='Distribution of Drug Overdose Death Rates')
# Scatter Plot
    scatter_fig = px.scatter(data, x=scatter_x, y=scatter_y,
                             title=f'Scatter Plot: {scatter_y} vs {scatter_x}')

    return line_fig, bar_fig, stacked_bar_fig, heatmap_fig, box_fig, scatter_fig


# Run the Dash app
if __name__ == '__main__':
    app.run_server(debug=True)
