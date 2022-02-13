---
title: 最短路径
date: 2014-08-22 17:52:14 +0800
categories: algorithm
---

最短路径问题：
给定带权的有向图G=(V,E)，从某顶点出发，沿图的边到达另一个顶点所经过的路径中，各边上权值之和最小的一条路径称为最短路径。
解决最短路径问题可以采用Dijstra算法和Floyd算法，其中Floyed算法可以求解任意两点间最短路径的长度。

## 一.Dijstra算法
Dijstra算法是按路径长度的非递减次序逐一产生最短路径的算法：首先求得长度最短的一条最短路径，再求得长度次短的一条最短路径，依次类推，直到从源点到其它所有顶点之间的最短路径都已求得为止。

设集合S存放已经求得最短路径的终点，则V－S为尚未求得最短路径的终点。初始状态下集合S中只有一个源点，设为v0。Dijstra的具体做法是：首先产生源点v0到自身的路径，长度为0，将v0加入S；算法的每一步，按照最短路径值的非减次序产生下一条最短路径，并将该路径的终点t加入S；直到S＝V，算法结束。

1.数据结构：
a) 一维数组d,d[i]中存放从源点v0到i的当前最短路径的长度，该路径上除顶点i以外，其余顶点都属于S，并且这是所有这些路径中的最短者。
b) 一维整型数组path，path[i]中存放从v0到顶点i的最短路径上，位于顶点i前面的那个顶点。
c) 一维布尔数组s,若s[i]为true表示顶点i再S中，否则表示i再V－S中。

2.算法实现(Java)：
```
public class Dijkstra {
    //选出最小的d[k]
	public static int choose(double[] d,boolean[] s){
		double min=Double.POSITIVE_INFINITY;
		int minpos=-1;
		int n=d.length;
		for(int i=0;i<n;i++){
			if(d[i]<=min && !s[i]){
				min=d[i];
				minpos=i;
			}
		}
		return minpos;
	}
	//v表示源点，a为图的邻接矩阵，d和path含义如上所述
	public static void dijkstra(int v,double[][] a,double[] d,int[] path){
		if(d==null)return;
		int n=a.length;
		if(n==0||n!=a[0].length||v<0||v>n-1)return;
		//s[i]为true表示顶点i在S中，否则表示i在V－S中
		boolean[] s=new boolean[n];
		//初始化操作s[i]初始化为false，d[i]=a[v][i]
		for(int i=0;i<n;i++){
			s[i]=false;
			d[i]=a[v][i];
			if(i!=v && d[i]<Double.POSITIVE_INFINITY){
				path[i]=v;
			}else{
				path[i]=-1;
			}
		}
		//将顶点v加入S
		s[v]=true;
		d[v]=0;
		for(int i=1;i<n;i++){
			int k=choose(d,s);
			s[k]=true;
			for(int w=0;w<n;w++){
				if(!s[w] && d[k]+a[k][w]<d[w]){
					d[w]=d[k]+a[k][w];
					path[w]=k;
				}
			}
		}
	}
	
	public static void main(String[] args) {
		double INFI=Double.POSITIVE_INFINITY;
                double[][] a={ { 0,50,10,INFI,70,INFI},{INFI,0,15,INFI,10,INFI},{20,INFI,0,15,INFI,INFI},{INFI,20,INFI,0,35,INFI},{INFI,INFI,INFI,30,0,INFI},{INFI,INFI,INFI,3,INFI,0 } };
		double[] d=new double[a.length];
		int[] path=new int[a.length];
		dijkstra(0,a,d,path);
		int k=path[4];
		System.out.println(k);
		while(k!=0){
			k=path[k];
			System.out.println(k+"\t");
		}
	}
}
```

时间复杂度为O(N^2)

## 二.Floyd算法
Floyd算法对于求任意两个顶点间的最短距离更加直接。
1.数据结构：
a) d[i][j]表示从顶点i到j中，只经过S中的顶点的，所有可能的路径中的最短路径的长度。如果从i到j，中间只经过S中的顶点的当前没有路径想通，那么d[i][j]为一个大值INFINITY。可以称此时的d[i][j]中保存的是从i到j的“当前最短路径”的长度。随着S中顶点数增加，d[i][j]的值不断修正， 当S＝V的时候，d[i][j]中的值就是从i到j的最短路径长度。
b) path[i][j]表示从顶点i到j的最短路径上，顶点j的前一个顶点。

2.算法实现(Java)：

```
public class Floyd {
	public static void floyd(double[][] d,int[][] path){
		if(d==null)return;
		int n=d.length;
		if(n==0||n!=d[0].length)return;
		//初始化
		for(int i=0;i<n;i++){
			for(int j=0;j<n;j++){
				if(i!=j && d[i][j]<Integer.MAX_VALUE){
					path[i][j]=i;
				}else{
					path[i][j]=-1;
				}
			}
		}
		//更新d和path
		for(int k=0;k<n;k++){
			for(int i=0;i<n;i++){
				for(int j=0;j<n;j++){
					if(d[i][k]+d[k][j]<d[i][j]){
						d[i][j]=d[i][k]+d[k][j];
						path[i][j]=path[k][j];
					}
				}
			}
		}
	}
	
	public static void main(String[] args) {
		double INFINITY=Double.POSITIVE_INFINITY;
		double[][] d={ { 0,1,INFINITY,4},{INFINITY,0,9,2},{3,5,0,8},{INFINITY,INFINITY,6,0 } };
		int[][] path=new int[4][4];
		floyd(d,path);
		//0-2的最短路径
		System.out.print(2+"\t");
		int k=path[0][2];
		System.out.print(k+"\t");
		while(k!=0){
			k=path[0][k];
			System.out.print(k+"\t");
		}
	}
}
```
