# kmeans
### javascript 实现k-means算法
- 写个类，填好k值和数据，方便调用
````javascript
var Kcluster = function(k , dataBbj){
	this.k = k ? k : 3;
	this.data = dataBbj;
}
````
- 计算距离并分类
````javascript
Kcluster.prototype.calcDistance = function(){
	for(var i = 0 ; i < this.data.length ; i ++ ){
		var minDis = 99999;
		var minArr = [];
		var minType = 9999;
		for(var j = 0 ; j < this.allKcluster.length ; j ++ ){
			var deltaX = this.data[i][0] - this.allKcluster[j].x
			var deltaY = this.data[i][1] - this.allKcluster[j].y
			var dis = Math.sqrt( deltaX*deltaX + deltaY*deltaY )

			if(dis < minDis ){
				minDis = dis;
				minArr = this.data[i]
				minType = j;
			}
		}

		for(var t =  0 ; t < this.typeAndData.length ; t ++){
			this.typeAndData[t].type == minType && this.typeAndData[t].data.push( minArr )
		}
	}
}
````
- 生成新的类中心点的位置
- 要是新的中心点的位置和旧的一样，即可以停止程序了。
- 有一点细节需要注意的是，要等到x和y都和上一次相等的时候才算是可以停止程序（而不是单一的判断x或者y相同就可以了）
````javascript
Kcluster.prototype.newKcluster = function(){
	var times = 0;
	for(var i = 0 ; i < this.allKcluster.length ; i ++){
		var newX = 0;
		var newY = 0;
		var dataArr = this.typeAndData[this.allKcluster[i].kType].data
		var dataLen = dataArr.length
		for(var j = 0 ; j < dataLen ; j ++){
			newX += dataArr[j][0]/dataLen
			newY += dataArr[j][1]/dataLen
		}
		newX = newX.toFixed(2)
		newY = newY.toFixed(2)
		console.log(newX ,newY)
		this.drawPoint( newX ,newY,'yellow')
		if( this.allKcluster[i].x == newX && this.allKcluster[i].y == newY ){
			times ++
		}
		this.allKcluster[i].x = newX;
		this.allKcluster[i].y = newY;
	}
	if(times == this.allKcluster.length){
		this.go = false;
		for(var i = 0 ; i < this.allKcluster.length ; i ++){
			this.drawPoint( this.allKcluster[i].x , this.allKcluster[i].y , 'red')
		}
	}
}
````
- 计算出数据的x方向和y方向的最大最小值
````javascript
Kcluster.prototype.calculateRanges = function(){
	this.allX = {min:1000000 , max : 0};
	this.allY = {min:1000000 , max : 0};

	for(var i in this.data ){
		if(  this.data[i][0] < this.allX.min ){
			this.allX.min = this.data[i][0];
		}

		if(  this.data[i][0] > this.allX.max ){
			this.allX.max = this.data[i][0];
		}

		if(  this.data[i][1] < this.allY.min ){
			this.allY.min = this.data[i][1];
		}

		if(  this.data[i][1] > this.allY.max ){
			this.allY.max = this.data[i][1];
		}
	}
	var perX = Math.floor( (this.canvas.width - 2*this.canvas.padding) / (this.allX.max - this.allX.min) )
	var perY = Math.floor( (this.canvas.height - 2*this.canvas.padding) / (this.allY.max - this.allY.min) )
	this.range = {
		perX : perX,
		perY : perY
	}
}
````
- 每个类中心点的类
````javascript
var PerKcluster = function(){
	this.kType = 999999;
}
````
- 生成类中心点的随机的x,y的位置（一定要在范围之内~）
````javascript
PerKcluster.prototype.random = function(){
	var rangX = this.range.allX.max - this.range.allX.min;
	var rangY = this.range.allY.max - this.range.allY.min;

	this.x =  Math.random()*rangX + this.range.allX.min
	this.y = Math.random()*rangY + this.range.allY.min

	this.x = this.x.toFixed(2)
	this.x = this.y.toFixed(2)
}
````
- action函数是包括 **计算距离**和**生成新的类中心点的位置** 这两个操作，而且要判断是否已经结束。
- this.go是指是否类中心点的位置已经不再变化
````javascript
Kcluster.prototype.action = function(){
	this.calcDistance()
	this.newKcluster()
	console.log(this.typeAndData)
	var self = this;
	if( this.go ){
		setTimeout(function() {
		self.action();
		}, 20);
	}
}
````
- init函数，主要逻辑在这里体现
- allKcluster，储存所有的类中心点的数据
- typeAndData，储存数据点的类型和数据结构为
	````javascript
	[
	{type:0,data:[[0,1],[],[],[]]},
	{type:1,data:[[0,1],[],[],[]]}
	.....
	]
	````

- this.canvas,储存canvas的数据的。
- this.go，是否结束程序的布尔值
- 这里有个特点就是生成不重复的随机点（要是重复的随机点就相当于少了一个类了）
````javascript
Kcluster.prototype.init = function(canvas){
	var self = this;
	self.allKcluster = [];
	self.typeAndData = [];
	self.go = true;
	self.canvas = {
		id:canvas,
		width : 600,
		height : 400,
		padding : 10
	}
	// 获得所有点的最大最小值
	self.calculateRanges()
	// 画好画布背景。
	self.showCanvas(canvas);
	// 画出现有的点
	for(var i = 0 ; i < self.data.length ; i ++ ){
		self.drawPoint( self.data[i][0] ,self.data[i][1])
	}
	PerKcluster.prototype.range = {
		allX:self.allX ,
		allY:self.allY 
	}
	for(var k = 0 ; k < self.k ; k ++){
		var perKcluster = new PerKcluster();
		perKcluster.kType = k;
		perKcluster.random();
		// 生成不重复的随机点
		for(var i in self.allKcluster ){
			if(self.allKcluster[i].x == perKcluster.x && self.allKcluster[i].y == perKcluster.y){
				perKcluster.random()
			}
		}

		self.drawPoint( perKcluster.x ,perKcluster.y , 'black')
		self.allKcluster.push(perKcluster);
		self.typeAndData.push({data:[],type:k})
	}
	self.action()
}
````

- 使用方法

````javascript
window.onload = function(){
	var data = [  
	    [1, 2],
	    [2, 1],
	    [2, 4], 
	    [1, 3],
	    [2, 2],
	    [3, 1],
	    [1, 1],

	    [7, 3],
	    [8, 2],
	    [6, 4],
	    [7, 4],
	    [8, 1],
	    [9, 2],

	    [10, 8],
	    [9, 10],
	    [7, 8],
	    [7, 9],
	    [8, 11],
	    [9, 9],
	];

	var kcluster = new Kcluster(3,data);
	kcluster.init('canvas');
}
````

### 关于画图展示方面，我不作解释了，不是重点。
### 大家可以自行下载代码进行研究

