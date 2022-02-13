---
title: 最小生成树Kruskal算法和Prim算法 
date: 2013-03-25 22:02:05 +0800 
categories: algorithm
---

解决最小生成树问题的两种方法Kruskal和Prim算法都是贪心算法，所谓贪心算法就是在每一步选择中采取当前状态下最好或者最优的选择，从而希望导致结果最好或者最优的算法。  

贪心算法适合解决最优子结构问题，即局部最优解能决定全局最优解的问题。或者说问题能够分解成子问题来解决，子问题的最优解递推到最终问题的最优解。与动态规划不同的是它对于每个子问题的解决方案都作出选择，不能回退，而动态规划则会保存中间结果，并根据之前的结果作出选择，能够回退。  

下面分别是Kruskal和Prim算法的C语言实现：  
Kruskal：
```c  
#include<stdio.h>
#include<stdlib.h>
void printArray(int a[][100],int n);
void AdjacencyMatrix(int a[][100], int n){
    int i,j;
    for(i = 0;i < n; i++)
    {
        for(j = 0;j < i; j++)
        {
            a[i][j] = a[j][i]= rand()%50;
            if( a[i][j]>40)a[i][j]=a[j][i]=999;
        }
    a[i][i] = 999;
    }
    printArray(a,n);
}
void printArray(int a[][100],int n){
    int i,j;
    for(i=0;i<n;i++)
    {
        for(j=0;j<n;j++)
        {
            printf("%d\t",a[i][j]);
        }
        printf("\n");
    }
}
int root(int v,int p[]){
    while(p[v] != v){v = p[v];}
    return v;
}
void union_ij(int i,int j,int p[]){
    if(j > i)
        p[j] = i;
    else
        p[i] = j;
}
void kruskal(int a[][100],int n){
    int count, i, p[100], min, j, u, v, k, t[100][100], sum;
    count = k = sum = 0;
    for(i = 0; i < n; i++)
    {
        p[i] = i;
    }
    while(count < n)
    {
        min = 999;
        for(i = 0; i < n; i++)
        {
            for(j = 0;j < n; j++)
            {
                if(a[i][j] < min)
                {
                    min = a[i][j];
                    u = i;
                    v = j;
                }
            }
        }
        if(min != 999)
        {
            i = root(u, p);
            j = root(v, p);
            if (i != j)
            {
                t[k][0] = u;
                t[k][1] = v;
                k++;
                sum += min;
                union_ij(i,j,p);
            }
        a[u][v] = a[v][u] = 999；         
        }count +=1;
    }   
    if(count != n)
    {
        printf("spanning tree not exist\n");
    }
    if(count == n)
    {
        printf("Adges Spanning tree is\n");
        for(k = 0; k < n-1 ; k++)
        {
            printf(" %d -> %d ",t[k][0],t[k][1]);
        }
    printf("\ncost = %d \n",sum);
    }
}
int main()
{
    int a[100][100],n;
    printf("enter the number of vertices\n");
    scanf("%d",&n);
    AdjacencyMatrix(a,n);
    kruskal(a,n);
    return 0;
}
```
Prim：
```c
#include <stdio.h>
#include <stdlib.h>
#define MAX 100
#define MAXCOST 0x7fffffff
int graph[MAX][MAX];
int Prim(int graph[][MAX], int n)
{
	// lowcost[i]记录以i为终点的边的最小权值，当lowcost[i]=0时表示终点i加入生成树
	int lowcost[MAX];
	// mst[i]记录对应lowcost[i]的起点，当mst[i]=0时表示起点i加入生成树
	int mst[MAX];
	int i, j, min, minid, sum = 0;
	// 默认选择1号节点加入生成树，从2号节点开始初始化
	for (i = 2; i <= n; i++)
	{
		// 最短距离初始化为其他节点到1号节点的距离
		lowcost[i] = graph[1][i];
		// 标记所有节点的起点皆为默认的1号节点
		mst[i] = 1;
	}
	// 标记1号节点加入生成树
	mst[1] = 0;
	// n个节点至少需要n-1条边构成最小生成树
	for (i = 2; i <= n; i++)
	{
		min = MAXCOST;
		minid = 0;
		// 找满足条件的最小权值边的节点minid
		for (j = 2; j <= n; j++)
		{
			// 边权值较小且不在生成树中
			if (lowcost[j] < min && lowcost[j] != 0)
			{
				min = lowcost[j];
				minid = j;
			}
		}
		// 输出生成树边的信息:起点，终点，权值
		printf("%c - %c : %d\n", mst[minid] + 'A' - 1, minid + 'A' - 1, min);
		// 累加权值
		sum += min;
		// 标记节点minid加入生成树
		lowcost[minid] = 0;
				// 更新当前节点minid到其他节点的权值
		for (j = 2; j <= n; j++)
		{
			// 发现更小的权值
			if (graph[minid][j] < lowcost[j])
			{
				// 更新权值信息
				lowcost[j] = graph[minid][j];
				// 更新最小权值边的起点
				mst[j] = minid;
			}
		}
	}
	// 返回最小权值和
	return sum;
}
int main()
{
	int i, j, k, m, n;
	int x, y, cost;
	char chx, chy;
	// 读取节点和边的数目
	scanf("%d%d", &m, &n);
	getchar();
	// 初始化图，所有节点间距离为无穷大
	for (i = 1; i <= m; i++)
	{
		for (j = 1; j <= m; j++)
		{
			graph[i][j] = MAXCOST;
		}
	} 
	// 读取边信息
	for (k = 0; k < n; k++)
	{
		scanf("%c %c %d", &chx, &chy, &cost);
		getchar();
		i = chx - 'A' + 1;
		j = chy - 'A' + 1;
		graph[i][j] = cost;
		graph[j][i] = cost;
	}
		// 求解最小生成树
	cost = Prim(graph, m);
	// 输出最小权值和
	printf("Total:%d\n", cost);
    return 0;	
}

```  
参考：[http://www.slyar.com/blog/prim-simplicity-c.html](http://www.slyar.com/blog/prim-simplicity-c.html)
