# Расчетная работа 

## Цель: 

Ознакомиться с основами теории графов, способами представления графов, базовыми алгоритмами для работы с разными видами графов.

## Условие задания: 

>*Вариант 2.16*

Реализовать на C++ код, который может определить толщину неориентированного графа.

Граф представляется в виде списка смежности.
### Ключевые понятия 
`Граф` - математическая абстракция реальной системы любой природы, объекты которой обладают парными связями.

`Неориентированный граф` —  граф, рёбрам которого не присвоено направление.

`Список смежности(инцидентности)` - один из способов представления графа в виде коллекции списков вершин. Каждой вершине графа соответствует список, состоящий из «соседей» этой вершины.

`Подграф` — это часть графа, в которой мы берем некоторые его вершины и ребра.

`Цикл` — граф, состоящий из единственного цикла, или, другими словами, некоторого числа вершин, соединённых замкнутой цепью

`Смежность` — понятие, используемое в отношении только двух рёбер либо только двух вершин: Два ребра, инцидентные одной вершине, называются смежными; две вершины, инцидентные одному ребру, также называются смежными. 
`Толщина графа`- это наименьшее число плоских подграфов, на которые можно разложить рёбра графа.
`Планарный граф`- это граф, допускающий укладку на плоскости, т.е. он может быть изображен на плоскости так, что никакие ребра не имеют общих точек, кроме своих вершин.
## Теорема Понтрягина – Куратовского. 
Граф планарен тогда и только тогда, когда он не содержит подграфов, гомеоморфных K5 или K3,3.

## Описание алгоритма
Если граф планарен (не содержит 𝐾 5 или 𝐾 3,3), то его толщина равна 1.
Если граф непланарен, алгоритм пытается найти подграфы, которые можно разделить на два планарных подграфа. Для этого временно удаляются рёбра и проверяется планарность оставшегося подграфа.
Если удаётся найти два планарных подграфа, толщина графа равна 2.
Если не удаётся найти два планарных подграфа, толщина графа принимается равной 3 (в общем случае, возможно, потребуется более сложная логика для точного определения толщины графа выше 2).
## Реализация на C++
``` c++
Код, выполняющий алгоритм
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>
using namespace std;

class Graph {
public:
    Graph(int vertices) : vertices(vertices), adjList(vertices) {}

    void addEdge(int v, int w) {
        adjList[v].push_back(w);
        adjList[w].push_back(v);
    }

    bool containsK5() {
        if (vertices < 5) return false;
        for (int a = 0; a < vertices; a++) {
            for (int b = a + 1; b < vertices; b++) {
                for (int c = b + 1; c < vertices; c++) {
                    for (int d = c + 1; d < vertices; d++) {
                        for (int e = d + 1; e < vertices; e++) {
                            if (isCompleteGraph({ a, b, c, d, e })) {
                                return true;
                            }
                        }
                    }
                }
            }
        }
        return false;
    }

    bool containsK33() {
        if (vertices < 6) return false;
        vector<int> nodes(vertices);
        iota(nodes.begin(), nodes.end(), 0);
        for (int i = 0; i < (1 << vertices); i++) {
            vector<int> leftSet, rightSet;
            for (int j = 0; j < vertices; j++) {
                if (i & (1 << j)) {
                    leftSet.push_back(j);
                }
                else {
                    rightSet.push_back(j);
                }
            }
            if (leftSet.size() == 3 && rightSet.size() == 3) {
                if (isCompleteBipartite(leftSet, rightSet)) {
                    return true;
                }
            }
        }
        return false;
    }

    bool isPlanar() {
        if (vertices <= 4) return true;
        if (containsK5() || containsK33()) return false;
        return true;
    }

    int calculateThickness() {
        if (isPlanar()) {
            return 1;
        }

        for (int i = 0; i < vertices; i++) {
            for (int j = i + 1; j < vertices; j++) {
                if (!adjList[i].empty() && !adjList[j].empty()) {
                    vector<vector<int>> tempAdjList = adjList;
                    tempAdjList[i].erase(remove(tempAdjList[i].begin(), tempAdjList[i].end(), j), tempAdjList[i].end());
                    tempAdjList[j].erase(remove(tempAdjList[j].begin(), tempAdjList[j].end(), i), tempAdjList[j].end());

                    Graph tempGraph(vertices);
                    tempGraph.adjList = tempAdjList;

                    if (tempGraph.isPlanar()) {
                        return 2;
                    }
                }
            }
        }

        return 3;
    }

private:
    int vertices;
    vector<vector<int>> adjList;

    bool isCompleteGraph(const vector<int>& nodes) {
        for (size_t i = 0; i < nodes.size(); i++) {
            for (size_t j = i + 1; j < nodes.size(); j++) {
                if (find(adjList[nodes[i]].begin(), adjList[nodes[i]].end(), nodes[j]) == adjList[nodes[i]].end()) {
                    return false;
                }
            }
        }
        return true;
    }

    bool isCompleteBipartite(const vector<int>& leftSet, const vector<int>& rightSet) {
        for (int u : leftSet) {
            for (int v : rightSet) {
                if (find(adjList[u].begin(), adjList[u].end(), v) == adjList[u].end()) {
                    return false;
                }
            }
        }
        return true;
    }
};

int main() {
    setlocale(LC_ALL, "RU");
    int v, e;
    cout << "Введите количество вершин: ";
    cin >> v;
    cout << "Введите количество рёбер: ";
    cin >> e;

    Graph graph(v);
    cout << "Введите пары вершин (каждое ребро как пару вершин):" << endl;
    for (int i = 0; i < e; i++) {
        int u, w;
        cin >> u >> w;
        graph.addEdge(u - 1, w - 1);
    }
    int thickness = graph.calculateThickness();
    cout << "Толщина графа равна: " << thickness << endl;


    return 0;
}

