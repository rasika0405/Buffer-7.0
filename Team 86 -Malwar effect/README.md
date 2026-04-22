Problem Statement :
Lateral malware movement forces organizations to rely on costly, system-wide network shutdowns, crippling automated business processes to protect main servers.

Data structure used :
Adjacency List (List<Edge>[]): To store the network nodes and their connections efficiently.

Custom Objects (class Edge): To hold specific link data like capacity, current flow, and "reverse edges" for backtracking.

Queue (LinkedList): To perform BFS (Breadth-First Search), which simulates how malware spreads layer-by-layer.

Arrays (boolean[], int[]): To track which devices are already infected and to remember the path back to the source.


video link:
https://drive.google.com/file/d/1kcr0YqnysN-YpUcRGskq15eww3DDs4uf/view?usp=sharing


