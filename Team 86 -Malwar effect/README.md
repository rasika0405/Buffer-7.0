Problem Statement :
Lateral malware movement forces organizations to rely on costly, system-wide network shutdowns, crippling automated business processes to protect main servers.

Data structure used :
Adjacency List (List<Edge>[]): To store the network nodes and their connections efficiently.

Custom Objects (class Edge): To hold specific link data like capacity, current flow, and "reverse edges" for backtracking.

Queue (LinkedList): To perform BFS (Breadth-First Search), which simulates how malware spreads layer-by-layer.

Arrays (boolean[], int[]): To track which devices are already infected and to remember the path back to the source.




import java.util.*;

class NetworkSecurityAnalyzer {

    static class Edge {
        int to, capacity, flow;
        Edge reverse;

        Edge(int to, int capacity) {
            this.to = to;
            this.capacity = capacity;
            this.flow = 0;
        }
    }

    static class Network {
        int nodes;
        List<Edge>[] adj; // used for BOTH spread + flow

        Network(int n) {
            nodes = n;
            adj = new ArrayList[n];
            for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
        }

        // single unified edge creation
        void connect(int u, int v, int capacity) {
            // forward edge
            Edge fwd = new Edge(v, capacity);
            Edge rev = new Edge(u, 0);

            fwd.reverse = rev;
            rev.reverse = fwd;

            adj[u].add(fwd);
            adj[v].add(rev);

            // ALSO add reverse for undirected spread (infinite-like capacity)
            Edge fwd2 = new Edge(u, capacity);
            Edge rev2 = new Edge(v, 0);

            fwd2.reverse = rev2;
            rev2.reverse = fwd2;

            adj[v].add(fwd2);
            adj[u].add(rev2);
        }
    }

    // Malware Spread (ignores capacity, uses structure only)
    static void simulateSpread(Network net, int source) {
        boolean[] visited = new boolean[net.nodes];
        Queue<Integer> q = new LinkedList<>();

        q.add(source);
        visited[source] = true;

        System.out.println("\n[Malware Propagation Order]");

        while (!q.isEmpty()) {
            int node = q.poll();
            System.out.print(node + " ");

            for (Edge e : net.adj[node]) {
                if (!visited[e.to]) {
                    visited[e.to] = true;
                    q.add(e.to);
                }
            }
        }
        System.out.println();
    }

    static boolean bfs(Network net, int s, int t, int[] parent) {
        Arrays.fill(parent, -1);
        Queue<Integer> q = new LinkedList<>();
        q.add(s);
        parent[s] = -2;

        while (!q.isEmpty()) {
            int u = q.poll();
            for (Edge e : net.adj[u]) {
                if (parent[e.to] == -1 && e.capacity - e.flow > 0) {
                    parent[e.to] = u;
                    if (e.to == t) return true;
                    q.add(e.to);
                }
            }
        }
        return false;
    }

    static int computeMaxFlow(Network net, int source, int sink) {
        int flow = 0;
        int[] parent = new int[net.nodes];

        while (bfs(net, source, sink, parent)) {
            int pathFlow = Integer.MAX_VALUE;

            int v = sink;
            while (v != source) {
                int u = parent[v];
                for (Edge e : net.adj[u]) {
                    if (e.to == v && e.capacity - e.flow > 0) {
                        pathFlow = Math.min(pathFlow, e.capacity - e.flow);
                    }
                }
                v = u;
            }

            v = sink;
            while (v != source) {
                int u = parent[v];
                for (Edge e : net.adj[u]) {
                    if (e.to == v && e.capacity - e.flow > 0) {
                        e.flow += pathFlow;
                        e.reverse.flow -= pathFlow;
                        break;
                    }
                }
                v = u;
            }

            flow += pathFlow;
        }

        return flow;
    }

    static void identifyCriticalLinks(Network net, int source) {
        boolean[] visited = new boolean[net.nodes];
        dfs(net, source, visited);

        System.out.println("\n[Critical Links to Isolate Infection]");
        for (int u = 0; u < net.nodes; u++) {
            for (Edge e : net.adj[u]) {
                if (visited[u] && !visited[e.to] && e.capacity > 0) {
                    System.out.println(u + " -> " + e.to);
                }
            }
        }
    }

    static void dfs(Network net, int u, boolean[] visited) {
        visited[u] = true;
        for (Edge e : net.adj[u]) {
            if (!visited[e.to] && e.capacity - e.flow > 0) {
                dfs(net, e.to, visited);
            }
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of devices (nodes): ");
        int n = sc.nextInt();

        Network network = new Network(n);

        System.out.print("Enter number of network links: ");
        int links = sc.nextInt();

        System.out.println("Enter links (source destination bandwidth):");
        for (int i = 0; i < links; i++) {
            int u = sc.nextInt();
            int v = sc.nextInt();
            int cap = sc.nextInt();

            if (u < 0 || u >= n || v < 0 || v >= n) {
                System.out.println("Invalid link skipped: " + u + " " + v);
                continue;
            }

            network.connect(u, v, cap);
        }

        System.out.print("Enter infected entry node: ");
        int source = sc.nextInt();

        System.out.print("Enter critical server node: ");
        int sink = sc.nextInt();

        // Run everything in one go
        simulateSpread(network, source);

        int maxFlow = computeMaxFlow(network, source, sink);
        System.out.println("\n[Maximum Infection Throughput]: " + maxFlow);

        identifyCriticalLinks(network, source);

        sc.close();
    }
}
