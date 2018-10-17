---
title: 计算多边形不超出矩形范围的最大缩放比例
date: 2016-12-29 09:16:29
tags: arithmetic
categories: javascript
---

计算方法
1. 获取矩形的width,height也就是x,y
2. 循环多边形的每个坐标点，取到最大的xmax,ymax
3. 计算scalex的缩放比例=xmax/x;计算scaley的缩放比例=ymax/y
4. 最大缩放比例就是scalex和scaley中最小的比例

__js代码__

```javascript
//根据围栏信息计算围栏的最大X和最大Y根据矩形大小计算缩放比例
function PolygonMaxXY(coords){
	var xarr = new Array();
	var yarr = new Array();
	for(var i = 0; i < coords.length; i++){
		if(i%2 ==0){
			xarr.push(coords[i]);
		}else{
			yarr.push(coords[i]);
		}
	}
	var maxx = 0;
	var maxy = 0;
	for(key in xarr){
		if(xarr[key]>maxx){
			maxx = xarr[key];
		}
	}
	for(key in yarr){
		if(yarr[key]>maxy){
			maxy = yarr[key];
		}
	}
	var width = $("#container").width();
	var height = $("#container").height();
	var stant = 0
	var stantx = width/maxx;
	var stanty = height/maxy;
	if(stantx<stanty){
		stant = stantx;
	}else{
		stant = stanty;
	}
	$("#mapscalestand").val(stant);
	$("#mapscale").val(stant);
}
```
按照计算比例放大缩小还原多边形
---

放大  
```javascript
function bebig(){
	var mapscalestand = parseFloat($("#mapscalestand").val());
	var scale=parseFloat($("#mapscale").val());
	if(scale==(mapscalestand*2)){
		return;
	}else{
		scale=scale+(mapscalestand/10);
	}
	if(scale>(mapscalestand*2)){
		scale=mapscalestand*2;
	}
	$("#mapscale").val(scale);
	$("#container").html("");

	xianshi();

}

```

 缩小
```javascript
function besmall(){
	var mapscalestand = parseFloat($("#mapscalestand").val());
	var scale= parseFloat($("#mapscale").val());
	if(scale==(mapscalestand/2)){
		return;
	}else{
		scale=scale-(mapscalestand/10);
	}
	if(scale<(mapscalestand/2)){
		scale=mapscalestand/2;
	}
	$("#mapscale").val(scale);
	$("#container").html("");
	xianshi();
}
```
还原
```javascript
function reload(){
	var mapscalestand = parseFloat($("#mapscalestand").val());
	$("#mapscale").val(mapscalestand);
	$("#container").html("");
	xianshi();
}
```
滚动
```javascript
function wheelload(num){
	var mapscalestand = parseFloat($("#mapscalestand").val());
	var scale = parseFloat($("#mapscale").val());
	if(num>0){
		if(scale==(mapscalestand*2)){
			return;
		}else{
			scale=scale+(mapscalestand/10);
		}
		if(scale>(mapscalestand*2)){
			scale=mapscalestand*2;
		}
		$("#mapscale").val(scale);
		$("#container").html("");

		xianshi();
	}else{
		if(scale==(mapscalestand/2)){
			return;
		}else{
			scale=scale-(mapscalestand/10);
		}
		if(scale<(mapscalestand/2)){
			scale=mapscalestand/2;
		}
		$("#mapscale").val(scale);
		$("#container").html("");
		xianshi();
	}
}
```
