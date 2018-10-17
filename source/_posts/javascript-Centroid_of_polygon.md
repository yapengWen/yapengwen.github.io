---
title: 多边形的质心
date: 2016-12-23 11:04:02
tags: arithmetic
categories: javascript
---


这个公式可以计算出，一个非自相交多变行的质点，设定多边形有n个顶点，(x0,y0), (x1,y1), ..., (xn−1,yn−1)

![polygonCentroid](https://yapengwen.github.io/img/polygonCentroid.png)

多边形的面积

![polygonCentroid](https://yapengwen.github.io/img/polygonarea.png)

(Cx, Cy)就是多边形的质心他永远落在多边形内

```javascript
//根据围栏信息计算质点
function CentroidOfPolygon(coords){
//	var coords = [23, 20, 23, 160, 70, 93, 150, 109, 290, 139, 270, 93]
	var xarr = new Array();
	var yarr = new Array();
	for(var i = 0; i < coords.length; i++){
		if(i%2 ==0){
			xarr.push(coords[i]);
		}else{
			yarr.push(coords[i]);
		}
	}
	var A = 0;
	var cx = 0;
	var cy = 0;
	var n = xarr.length;
	for(var i = 0; i < n-1; i++){
		var Aitem = xarr[i]*yarr[i+1] - xarr[i+1]*yarr[i]
		A = A + Aitem;
		var cxItem = (xarr[i]+xarr[i+1])*(xarr[i]*yarr[i+1]-xarr[i+1]*yarr[i])
		cx = cx + cxItem;
		var cyItem = (yarr[i]+yarr[i+1])*(xarr[i]*yarr[i+1]-xarr[i+1]*yarr[i])
		cy = cy + cyItem;
	}
	A = A/2;
	cx = cx/(6*A);
	cy = cy/(6*A);

	var retarr = new Array();
	retarr.push(cx);
	retarr.push(cy);
	return retarr;
}
```

参考文章[https://en.wikipedia.org/wiki/Centroid#Centroid_of_polygon](https://en.wikipedia.org/wiki/Centroid#Centroid_of_polygon)
