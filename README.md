# A method of plotting public transport graphs from networkx in Tableau: an example.
Showcasing a small and simple method of plotting networkx public transport networks in tableau.

### step 0: imports
```python
import pandas as pd
import networkx as nx
```

### step 1: creating a networkx graph of the london underground network.

```python
def get_network(): 
    
    lines = pd.read_csv(r'../data/london.lines.csv', index_col='line')
    stations = pd.read_csv(r'../data/london.stations.csv')
    connections = pd.read_csv(r'../data/london.connections.csv')

    graph = nx.Graph()

    for station_id, station in stations.iterrows():
        graph.add_node(station['name'], lon=round(station['longitude'],4), lat=round(station['latitude'],4), s_id=station['id'])

    for connection_id, connection in connections.iterrows(): # adding edges to the network using the station names in the connections csv, and adding time estimates. 
        station1_name = stations.loc[stations['id'] == connection['station1'],'name'].item()
        station2_name = stations.loc[stations['id'] == connection['station2'],'name'].item()
        graph.add_edge(station1_name, station2_name, time = connection['time'], line = lines.loc[connection['line'], 'name'])
        
    return graph
```

### step 2: creating the spatial reference xlsx file.

```python
graph = get_network()

networkdf = pd.DataFrame()
lons, lats = map(nx.get_node_attributes, [graph, graph], ['lon', 'lat'])
lines, times = map(nx.get_edge_attributes, [graph, graph], ['line', 'time'])

for edge in list(graph.edges()):
    networkdf = networkdf.append({"s-e":'start', 
    'edge':edge, 
    'station':edge[0], 
    'lon': lons.get(edge[0]),
    'lat': lats.get(edge[0]),
    'line': lines.get(edge),
    'time' : times.get(edge)
    }, ignore_index=True)
    networkdf = networkdf.append({"s-e":'end', 
    'edge':edge, 
    'station':edge[1], 
    'lon': lons.get(edge[1]),
    'lat': lats.get(edge[1]),
    'line': lines.get(edge),
    'time' : times.get(edge)
    }, ignore_index=True)   
    
networkdf.to_excel('networkgraph.xlsx')    
```
The key here is to define a 'start' and 'end' of each edge. This will be used in Tableau to draw the lines representing the links. Make sure the edge name here is identical to the edge name in your metrics sheet so joining is made easy. The reference datafile in this repo is named 'networkgraph2.xlsx'.

### step 3: load your reference xlsx file, as well as your metrics xlsx file in tableau and join them: 


### step 4: plot away using these settings: 

