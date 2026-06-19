---
layout: post
title:  "Chapter 11: Graphs and NetworkX"
date:   2026-05-06
categories: book
TOC_order: 11
copyright: "Copyright © 2024-2026, P. L. Chiu. All Rights Reserved."
---

Graphs are abstract mathematical objects with vertices and edges (also called nodes and links). The term networks is also used to refer to graphs that represent data structures and may have additional attributes attached to the vertices and edges. There are many applications such as social networks, supply chain management, hierarchically organized information, etc.

NetworkX [1] is a popular Python library for graphs and networks, and in this chapter we will use it to demonstrate how to create graphs (randomly generated and constructed from datasets), save them to a file (in JSON format), and visualize them. We will then work through an example of social graphs in the novel *Huckleberry Finn* using data from the Stanford GraphBase [2].

## 11.1 Creating and Visualizing Graphs

We begin by creating a random graph. A standard method to do this is to take n nodes and for each pair of nodes put an edge between them with probability p. This is called an *Erdős–Rényi model*, and NetworkX provides a function `erdos_renyi_graph(n, p)` to create this type of graph. (An optional parameter `seed` can be set to a fixed number to generate the same graph.)

Once we have a graph G, we can visualize it. Graph visualization is a non-trivial problem. One difficulty is that there is no straightforward way to lay out a complicated graph object on a 2D image. A standard method employs a spring layout algorithm that simulates a system in which the nodes push one another apart by some repelling force, and the edges act like springs that pull the nodes together, and an equilibrium state provides the positions of the nodes. NetworkX has the function `spring_layout(G)` to compute the positions of the nodes of G. It uses the well-known Fruchterman-Reingold force-directed algorithm. The simulation terminates when it gets close to an equilibrium, and it relies on a random process to converge to one of the possibly many equilibriums. An optional `seed` parameter can be used to reproduce the layout.

With the computed node positions, we call the NetworkX function `draw(G, pos)` to render the visualization as a Pyplot image. See Figure 11.1-left. The code for creating and visualizing a random graph takes only a few lines:

```python
import networkx as nx
import matplotlib.pyplot as plt

G = nx.erdos_renyi_graph(n=10, p=0.25, seed=7)
pos = nx.spring_layout(G, seed=11)
nx.draw(G, pos, with_labels=True, node_color="lightgray", edge_color="gray")
plt.show()  
```

![Figure 11.1](/bgdv-book/assets/images/book/plot_1x2_rand-graph_n-10_weight-graph_n-4_grayscale_v01-a.png)  
*Figure 11.1. Left: A random graph (Erdős–Rényi model with n=10, p=0.25, seed=7). Right: A graph with edge weights which affects the spring layout.*

Next, we show how to construct a graph with **edge weights** by specifying the node labels, and the edges along with their weights:

```python
G = nx.Graph()
G.add_nodes_from(["A", "B", "C"])
G.add_edges_from([("A", "B", {"weight": 100}), 
                  ("A", "C", {"weight": 10}), 
                  ("A", "D", {"weight": 1})])
```

Visualizing this graph G with a spring layout (like we just did with the random graph above), we get the node positions shown in Figure 11.1-right. The spring layout algorithm assigns stronger attractive forces to the edges with higher weights. To show the edge labels, we add the following code above the line `plt.show()`:

```python
edge_labels = nx.get_edge_attributes(G, "weight")
nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels)
```

## 11.2 Saving Graphs to JSON Files

To save a graph's data to a file, we can employ the JSON format. Like the CSV format that we have been using, it is a human-readable text format, but it is based on key-value pairs and lists to specify objects for handling more complicated data structures.

NetworkX has functions for converting graphs to JSON format. The code snippets for saving and loading a graph G to and from data in a JSON file are:

```python
import json
import networkx as nx
from networkx.readwrite import json_graph

data = json_graph.node_link_data(G)
with open("nx_graph.json", "w") as f:
    json.dump(data, f)
```

```python
data = json.load(open("nx_graph.json"))
G = json_graph.node_link_graph(data)
```

For example, with the weighted graph G in Figure 11.1-right, the contents of the JSON file looks like:

```text
{"directed": false, "multigraph": false, "graph": {}, 
"nodes": [{"id": "A"}, {"id": "B"}, {"id": "C"}, {"id": "D"}], 
"links": [{"weight": 100, "source": "A", "target": "B"}, 
{"weight": 10, "source": "A", "target": "C"}, 
{"weight": 1, "source": "A", "target": "D"}]}
```

The human readability of the JSON file can be enhanced by formatting its text content into separate lines with indentation using the optional parameter `indent` in the function `json.dump()`; see Exercise 11.1.

## 11.3 Example: Social Graphs in the Novel *Huckleberry Finn*

In this section, we work through an example visualizing some of the social graphs in the novel *Huckleberry Finn* by Mark Twain. The data that we use here is available online from the Stanford GraphBase [2].

For each chapter in the novel, a subset of characters which are described as interacting in a scene form a group. In the data file (huck.dat), each line corresponds to a chapter, and the groups in the chapter are separated by a semi-colon, with the characters within each group separated by a comma. Each character is given by a two-letter abbreviation of their name. The list of abbreviations and the lines encoding the novel's 43 chapters look like:

```text
AB Abner Shackleford, friend of PW
AP Aunt Polly, aunt who raises TS
AS Aunt Sally Phelps, sister of AP
...
WW William Wilks, deaf and dumb brother of HW and PW
```

```text
1:TS,HF;JT;WD,HF,MW
2:JM,TS,HF;TS,HF,JH,BR,TB
3:WD,MW,HF;TS,HF,JH,BR;PA
...
43:HF,TS,AP,AS,SP,JM;PA,JM
```

For the construction of a social graph, we represent each character by a node, and place an edge between two characters who belong in a group, and set the weight of the edge to the number of groups they both belong to. A visualization employing a spring layout of this social graph is shown in Figure 11.2 (with and without node labels). In the version without labels, the visualization is less cluttered and it is easier to see the nodes. On the other hand, the labels help identify the character of a node.

Examining the visualization, we see that there is a large connected social subgraph, and two small isolated subgraphs. We can use the labels to get the names of the characters in the small subgraphs: The isolated subgraph with two nodes is {BD (Bud Grangerford), BS (Baldy Shepherdson)}. More interestingly, the subgraph with three nodes makes sense as its characters are all thieves: {BI (Bill, thief wants to shoot TU), JP (Jake Packard, thief wants to drown TU), TU (Jim Turner, tied-up thief)}.

![Figure 11.2](/bgdv-book/assets/images/book/plot_1x2_huck_entire_v01-e.png)  
*Figure 11.2. Left: A social graph for the novel Huckleberry Finn. Right: With node labels. Data source: Stanford GraphBase [2]*

Exploring further, let's look at the social graphs of the main characters Huck and Jim. One idea is to consider the characters who are in the same groups as Huck and as Jim. This is equivalent to finding the social subgraphs containing the neighboring nodes of Huck and of Jim. How to do this will be explained below. The resulting visualizations are shown in Figure 11.3. The positions of the nodes are kept the same in these subgraphs as in the entire graph in Figure 11.2.

Looking at these social subgraphs, we see that HF and JM are centrally positioned in their subgraphs (and also in the entire graph), which is consistent with the fact that they are main characters in the novel. Huck has 53 neighboring nodes, and Jim has 16; which makes sense since Huck is the title character.

We can also look at the common characters that both Huck and Jim are connected to; these are the orange nodes in the subgraphs. All except 3 of Jim's 16 neighbors are orange. The exceptions in blue are Jim's daughter LZ ('Lizabeth) and son JY (Johnny), and a minor character BU (Burton).

Another noticeable thing is that in the Huck social graph, there appears to be four clusters of characters. One could analyze these clusters further by asking whether some of them are fundamental objects in graph theory called *cliques* (a fully connected subgraph); see Exercise 11.3.

![Figure 11.3](/bgdv-book/assets/images/book/plot_1x2_huck_HF_JM_v01-e.png)  
*Figure 11.3. Left: A social graph of the neighboring nodes of HF (Huck Finn), with the HF node in red. Right: The corresponding graph for JM (Jim). The nodes common to both graphs are colored orange. Data source: Stanford GraphBase [2]*

In the remainder of this section, we go through our code for creating these visualizations from the Stanford GraphBase data for *Huckleberry Finn*. First, we parse the lines in the text file (huck.dat) to extract the groups. For example, the line `1:TS,HF;JT;WD,HF,MW` contains 3 groups from chapter 1 which can be extracted and represented by the list [["TS","HF"], ["JT"], ["WD',"HF","MW"]]. Here is the code for extracting all the groups from huck.dat:

```python
groups = []
with open("huck.dat", "r") as f:
    for line in f:
        line = line.strip()  # remove leading and trailing whitespace chars
        idx = line.find(":")
        if idx == -1:
            continue
        line = line[idx + 1: ]  # remove initial part including ':'
        s = line.split(";")
        groups.extend(s)
```

Next, we construct a graph G with edge weights, where the weight is the number of groups that contain the two nodes inducing that edge. For each group, we consider the edges formed by all pairs of nodes; these are given by the edges of the *complete graph* of the group's nodes. Then we check whether each of these edges matches an existing edge in G, and if so, we increment the weight of that edge; otherwise we create a new edge in G. We can also save this graph to a JSON file. The code is:

```python
G = nx.Graph()
for g in groups:
    nodes = g.split(",")
    cgraph = nx.complete_graph(nodes)
    for u, v in cgraph.edges():
        if G.has_edge(u, v):
            G[u][v]["weight"] += 1
        else:
            G.add_edge(u, v, weight=1)

# save to a JSON file
data = json_graph.node_link_data(G)
with open("huck_nx_graph_weights.json", "w") as f:
    json.dump(data, f, indent=2)
```

To plot the graph G without node labels (Figure 11.2-left) the code is:

```python
pos = nx.spring_layout(G, k=0.5, seed=3)
nx.draw(G, pos, with_labels=False, font_color="blue", node_color="blue", 
        node_size=20, alpha=0.5)
plt.show()
```

In the call to `spring_layout()`, we use the optional parameters `k` and `seed`. The `k` controls how far apart to place the nodes. The `seed` is for the underlying random process and using the same seed reproduces the same layout. For showing the node labels (Figure 11.2-right), set `with_labels=True` in the function `draw()`.

To make the two subgraphs in Figure 11.3 for the neighboring nodes of Huck (HF) and of Jim (JM), with their common neighbors colored orange, we proceed as follows. First, we use the `neighbors()` function to get the neighboring nodes of HF and of JM, and create subsets with these nodes including HF and JM, respectively. Then we take their intersection to get their common nodes.

```python
subset1 = set(G.neighbors("HF"))
subset1.add("HF")
subset2 = set(G.neighbors("JM"))
subset2.add(target2)
intersection = subset1.intersection(subset2)
```

We use the subsets and intersection to assign the node colors, so for HF we have:

```python
node_colors1 = ['red' if (n == "HF") else 
                orange if (n in intersection) else 
                blue for n in G.subgraph(subset1)]
```

Now we can plot the social graph for the neighbors of HF, where we use the same node positions `pos` that was computed for the entire graph above. The `draw()` function becomes:

```python
nx.draw(G.subgraph(subset1), pos,
        node_color=node_colors1,
        edge_color="gray",
        width=1,  # width of edges
        node_size=20, alpha=0.5)
```

Similarly, we can plot the social graph for neighbors of JM.

## Exercises

Ex.11.1. A JSON file can be formatted to improve human readability using the optional `indent` parameter in the function `json.dump(data, f, indent)`. Try saving the graph G with edge weights (Figure.11.1-right) to a JSON file with `indent=2`, and open the file to see how the text content is formatted into lines with indentation.

Ex.11.2. For Huck (HF) and Tom Sawyer (TS), make two social graphs with the neighbors of HF and of TS, with their common nodes in orange, similar to Figure.11.2.

\*Ex.11.3. In the Huck social graph (Figure.11.2-left), there appears to be four clusters of nodes in blue. A *clique* is a fully connected subgraph, and is a fundamental object in graph theory. Determine which of these clusters (including HF) are cliques. *Hint*: Start with the set of nodes that are neighbors of HF, and also add in HF. Find the cliques in this set (using the NetworkX function `find_cliques()`) and visualize them in different colors; this will reveal which clusters are cliques.
