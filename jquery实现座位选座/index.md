# jQuery实现座位选座


##  引言
   快期末了，`JS`课要交一次大作业。从几个案例中挑选一个，我选的是一个座位预定程序。要求：单击要预定的座位，座位会以某种颜色标识，右侧显示计算的票款和座位信息等。
   使用了`html`+`js`+`css`+`jQuery`，有1说1，这是我第一次使用`jQuery`插件。
   参考资料来源：
   1、`jQuery-Seat-Charts` 官方使用文档 https://github.com/mateuszmarkowski/jQuery-Seat-Charts
   2、`jQuery`在线选座订座 https://www.cnblogs.com/helloweba/p/4108512.html

## 关于jQuery-Seat-Charts
   使用文档里面很详细了。但是是英文版的。稍微整理了一下。
   ### 例子
   基本设置：
   ```Js
$(document).ready(function() {

	var sc = $('#seat-map').seatCharts({
		map: [
			'aaaaaaaaaaaa',
			'aaaaaaaaaaaa',
			'bbbbbbbbbb__',
			'bbbbbbbbbb__',
			'bbbbbbbbbbbb',
			'cccccccccccc'
		],
		seats: {
			a: {
				price   : 99.99,
				classes : 'front-seat' //your custom CSS class
			}
		
		},
		click: function () {
			if (this.status() == 'available') {
				//do some stuff, i.e. add to the cart
				return 'selected';
			} else if (this.status() == 'selected') {
				//seat has been vacated
				return 'available';
			} else if (this.status() == 'unavailable') {
				//seat has been already booked
				return 'unavailable';
			} else {
				return this.style();
			}
		}
	});

	//Make all available 'c' seats unavailable
	sc.find('c.available').status('unavailable');
	
	/*
	Get seats with ids 2_6, 1_7 (more on ids later on),
	put them in a jQuery set and change some css
	*/
	sc.get(['2_6', '1_7']).node().css({
		color: '#ffcfcf'
	});
	
	console.log('Seat 1_2 costs ' + sc.get('1_2').data().price + ' and is currently ' + sc.status('1_2'));

});
   ```
   ### 座位图
   ```js
  map: [
	'ff__ff', 
    'ff__ff', 
    '______', 
    'eee_ee', 
    'eee_ee', 
    'eee_ee', 
    'eee_ee', 
    'eee_ee', 
    'eee_ee'
]
   ```
   每个字符代表一个不同类型的座位，除下划线`_`以外的任何人。下划线用于表示在特定位置不应有任何座位。
   ### 定义座位属性
   ```js
   seats: { //定义座位属性 
    f: { 
        price   : 100, 
        classes : 'first-class',  
        category: '一等座' 
    }, 
    e: { 
        price   : 40, 
        classes : 'economy-class',  
        category: '二等座' 
    }                     
}, 
   ```
   上面的代码定义了一等座和二等座的价格、类别名称以及对应的`css`样式。他们可以在后面通过`data()`方法调用
   ### 定义行列等信息
   `top`:是否显示顶部横坐标
   `columns`: 定义横坐标（行）
   `rows`: 定义纵坐标（列）的值
   `getLabe`: 用来返回座位信息。

   ```js
   naming : { //定义行列等信息 
    top : true, 
    columns: ['A', 'B', 'C', '', 'D','F'], 
    rows: ['01','02','','03','04','05','06','07','08','09'], 
    getLabel : function (character, row, column) { 
        return row+column; 
    } 
}, 
   ```
### 定义图例
      `node`: 使用`legend`来定义图例，对应图例关联的元素是`#legend`,并对座位类型定义对应的样式。
      `Items`: 每个数组元素应为三元素数组：[字符，状态，描述]。
   状态有三种：
      `available`: 可以坐的座位
      `unavailable`: 无法坐下的座位
      `selected`: 当前用户已坐下的座位

   ```js
   legend : { //定义图例 
    node : $('#legend'), 
    items : [ 
        [ 'f', 'available',   '一等座' ], 
        [ 'e', 'available',   '二等座'], 
        [ 'f', 'unavailable', '已售出'] 
    ]                     
}, 
   ```
   ### click事件
   `if (this.status() == 'available')`  如果可选座
   `else if (this.status() == 'selected')`  已选中
   `else if (this.status() == 'unavailable')` 已出售

```js
   click: function () { 
    if (this.status() == 'available') {//可选座 
        $('<li>'+this.data().category+'<br/>01车'+this.settings.label+'号<br/>￥'+this.data().price+'</li>') 
        .attr('id', 'cart-item-'+this.settings.id) 
        .data('seatId', this.settings.id) 
        .appendTo($cart); 
        //更新票数 
        $counter.text(sc.find('selected').length+1); 
        //计算总计金额 
        $total.text(recalculateTotal(sc)+this.data().price); 
        return 'selected'; 
    } else if (this.status() == 'selected') {//已选中 
        $counter.text(sc.find('selected').length-1); 
        $total.text(recalculateTotal(sc)-this.data().price); 
        //删除已预订座位 
        $('#cart-item-'+this.settings.id).remove(); 
        return 'available'; 
    } else if (this.status() == 'unavailable') {//已售出 
        //已售出 
        return 'unavailable'; 
    } else { 
        return this.style(); 
    } 
},
```
   ### focus,blur事件
```js
focus  : function() {
	if (this.status() == 'available') {
		//if seat's available, it can be focused
		return 'focused';
	} else  {
		//otherwise nothing changes
		return this.style();
	}
},
blur   : function() {
	//The only place where you should return actual seat status
	return this.status();
},
```
`blur`:模糊处理程序。当座椅由于鼠标移动或箭头击中而失去焦点时触发。
`focus`:焦点处理程序。当座椅获得焦点时触发。

```js
//default handler
blur   : function() {
	return this.status();
},
//default handler
focus  : function() {

	if (this.status() == 'available') {
		return 'focused';
	} else  {
		return this.style();
	}
},
```
### 设定方法
   #### .status（ids，status）
   更新具有给定ID的座位组的状态。ids变量可以包含一个id或一个id数组。
   ```js
sc.status('2_15', 'unvailable'); //set status for one seat
sc.status(['2_15', '2_10'], 'unvailable'); //set status for two seats
   ```
   #### .status( status )
   更新当前集合中所有席位的状态。
   ```js
sc.find('unavailable').status('available'); //make all unvailable seats available
   ```
   #### .node( )
   返回jQuery座位节点引用集。
   ```js
sc.find('unavailable').node().fadeOut('fast'); //make all unavailable seats disappear
   ```
   #### .each( callback )
   遍历一个座位集，将在每个元素的上下文中触发回调。回调可以接受座位ID作为参数。
   ```js
sc.find('a.unavailable').each(function(seatId) {
	console.log(this.data()); //display seat data
}); 
   ```
### 案例代码
（js+html部分）
```html
<!doctype html>
<html>
<head>
<title>选座系统</title>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<link rel="stylesheet" type="text/css" href="css/jquery.seat-charts.css">
<link rel="stylesheet" type="text/css" href="css/style.css">
</head>
<body>
<div class="wrapper">
  <div class="container">
    <div id="seat-map">
      <div class="front-indicator">座位表</div>
    </div>
    <div class="booking-details">
      <h3>已选中的座位 (<span id="counter">0</span>):</h3>
      <ul id="selected-seats">
      </ul>
      <p>总价: <b>$<span id="total">0</span></b></p>
      <p><button class="checkout-button">结算</button></p>      
      <div id="legend"></div>
    </div>
  </div>
</div>
<script src="js/jquery-1.11.0.min.js"></script> 
<script src="js/jquery.seat-charts.min.js"></script> 
<script>
			var firstSeatLabel = 1; //定义第一个座位的序号
		
			$(document).ready(function() {
				var $cart = $('#selected-seats'),
					$counter = $('#counter'),
					$total = $('#total'),
					sc = $('#seat-map').seatCharts({
                      //座位图
					map: [
						'fffff',
						'fffff',
						'eeeee',
						'eeeee',
						'eeeee',
						'ee_ee',
						
					],
                      //座位属性
					seats: {
						f: {
							price   : 100,
							classes : 'first-class', //对应的css样式
						},
						e: {
							price   : 40,
							classes : 'economy-class', //对应的css样式
							category: '普通座位'
						}					
					
					},
                      //定义行列等信息，每一个数字自加操作
					naming : {
						top : false,
						getLabel : function (character, row, column) {
							return firstSeatLabel++;
						},
					},
					legend : {
						node : $('#legend'),
					    items : [
							[ 'f', 'available',   'VIP座位' ],
							[ 'e', 'available',   '普通座位'],
							[ 'f', 'unavailable', '已预定']
					    ]					
					},
					click: function () {
						if (this.status() == 'available') {
							$('<li>'+this.data().category+this.settings.label+'号座位'+'：<br/>价格：<b>$'+this.data().price+'</b> <a href="#" class="cancel-cart-item">[删除]</a></li>')
								.attr('id','cart-item-'+this.settings.id)
								.data('seatId', this.settings.id)
								.appendTo($cart);
							$counter.text(sc.find('selected').length+1);
							$total.text(recalculateTotal(sc)+this.data().price);
							
							return 'selected';
						} else if (this.status() == 'selected') {
							//update the counter
							$counter.text(sc.find('selected').length-1);
							//and total
							$total.text(recalculateTotal(sc)-this.data().price);
						
							//remove the item from our cart
							$('#cart-item-'+this.settings.id).remove();
						
							//seat has been vacated
							return 'available';
						} else if (this.status() == 'unavailable') {
							//seat has been already booked
							return 'unavailable';
						} else {
							return this.style();
						}
					}
				});

				//this will handle "[cancel]" link clicks
				$('#selected-seats').on('click', '.cancel-cart-item', function () {
					//let's just trigger Click event on the appropriate seat, so we don't have to repeat the logic here
					sc.get($(this).parents('li:first').data('seatId')).click();
				});

				//let's pretend some seats have already been booked
				sc.get(['1_2', '4_1', '7_1', '7_2']).status('unavailable');
		
		});

		function recalculateTotal(sc) {
			var total = 0;
			//basically find every selected seat and sum its price
			sc.find('selected').each(function () {
				total += this.data().price;
			});
			
			return total;
		}
		
		</script>
</body>
</html>
```
(css部分)
```css
body {
	font-family: 'Lato', sans-serif;
}
a {
	color: #b71a4c;
}
.front-indicator {
	width: 145px;
	margin: 5px 32px 15px 32px;
	background-color: #f6f6f6;	
	color: #adadad;
	text-align: center;
	padding: 3px;
	border-radius: 5px;
}
.wrapper {
	width: 100%;
	text-align: center;
}
.container {
	margin: 0 auto;
	width: 500px;
	text-align: left;
}
.booking-details {
	float: left;
	text-align: left;
	margin-left: 35px;
	font-size: 12px;
	position: relative;
	height: 401px;
}
.booking-details h2 {
	margin: 25px 0 20px 0;
	font-size: 17px;
}
.booking-details h3 {
	margin: 5px 5px 0 0;
	font-size: 14px;
}
div.seatCharts-cell {
	color: #182C4E;
	height: 25px;
	width: 25px;
	line-height: 25px;
	
}
div.seatCharts-seat {
	color: #FFFFFF;
	cursor: pointer;	
}
div.seatCharts-row {
	height: 35px;
}
div.seatCharts-seat.available {
	background-color: #B9DEA0;

}
div.seatCharts-seat.available.first-class {
/* 	background: url(vip.png); */
	background-color: #3a78c3;
}
div.seatCharts-seat.focused {
	background-color: #76B474;
}
div.seatCharts-seat.selected {
	background-color: #E6CAC4;
}
div.seatCharts-seat.unavailable {
	background-color: #ccc;
}
div.seatCharts-container {
	border-right: 1px dotted #adadad;
	width: 200px;
	padding: 20px;
	float: left;
}
div.seatCharts-legend {
	padding-left: 0px;
	bottom: 0;
}
ul.seatCharts-legendList {
	padding-left: 0px;
}
span.seatCharts-legendDescription {
	margin-left: 5px;
	line-height: 30px;
}
.checkout-button {
	display: block;
	margin: 10px 0;
	font-size: 14px;
}
#selected-seats {
	max-height: 200px;
	overflow-y: scroll;
	overflow-x: hidden;
	width: 170px;
}
```

## 目录结构
![](https://gitee.com/lonercci/picbed/raw/master/img/20201224002840.png)
## 运行结果
![](https://gitee.com/lonercci/picbed/raw/master/img/20201224002949.png)





