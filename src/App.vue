<template>
	<div id="app" @mouseup="up()" @mousemove="move()">
		<!--<ul class="box-select" v-html="bsarea">-->
		<ul class="box-select">
			<li v-for="(todo, index) in todos" :key='index'>
				<button type="button" v-on:click="changeBox(todo)"><svg v-html="madeSVG(index)"></svg></button>
			</li>
			<li id="canvasCustomArea">
				<div style="width:100%;line-height:30px;">自訂圖片配置</div>
				<div v-for="i in 10" :key='i'>
					<button v-for="j in 10" :key='j' :id="'canvasCustom'+(j-1)+(i-1)" type="button" v-on:click="canvasCustom(j-1,i-1)"></button>
				</div>
			</li>
			<li><button type="button" v-on:click="addText()">TEXT</button></li>
		</ul>
		<div class="edit-box-wrap">
			<div class="edit-box" v-bind:style="earea_style">
				<div class="canvas-area" v-html="canvas_area"></div>
				<div class="control-area">
					<div v-for="(control, index) in controls" :key='control.id' :id="'control'+(index+1)" 
					@mousedown="control_down(index+1)" 
					@wheel="MouseWheel(index+1)" 
					@mouseout="up()" 
					:style="control.style">
						<button type="button" 
						v-on:click="searchimg(index+1)" 
						:style="'transform:scale('+control.scale+')'">變更圖片</button>
					</div>
					<div v-for="(pointxy, index) in pointxys" :key='pointxy.id' 
					@mousedown="ponitxy_down(index+1)" 
					class="pointxy" 
					:id="'pointxy'+(index+1)" 
					:style="pointxy.style"></div>
				</div>
				<div class="text-area"></div>
			</div>
		</div>
		<div class="button-area">
			圖片尺寸 
			寬 <input type="number" name="canvasWidth" v-on:change='canvas_Width()' min="10" max="3000" v-model="canvasWidth">
			高 <input type="number" name="canvasHeight" v-on:change='canvas_Height()' min="10" max="3000" v-model="canvasHeight">
			間距 <input type="number" name="canvaslinewidth" v-on:change='canvas_line_Width()' min=0 v-model="canvasSet.spacing">
			<button type="button" v-on:click="changeTEST">測試</button>
			<button type="button" v-on:click="getBase64()">完成送出</button>
			<button type="button" v-on:click="resetCanvas(this.canvasSet.canvasPosition.length);">重新整理</button>
		</div>
		
		<!--<textarea id="canvasSetData" style="display:none;">
			{"point":[[0,480,960],[0,480,960]],"canvasPosition":[[0,0,1,1],[1,0,2,1],[0,1,1,2],[1,1,2,2]],"spacing":0,"imgPosition":[["images/5415_P_20200307012921_1.jpg",-21,-108,1.2,348],["images/5428_P_20200307002947_1.jpg",-18,-92,1.15,0],["images/5345_P_20200220221542_1.jpg",1,-70,1,0],["images/5488_P_20200330124416_1.jpg",-25,-113,1.2,12]],"text":[{"text":"文字","x":304,"y":565,"size":36,"font":"'Noto\\ Sans\\ TC'","color":"#cf610c","weight":"400"}]}
		</textarea>-->
		<textarea id="canvasSetData" style="display:none;" v-model="canvasSet"></textarea>
		<div id="tempSelectImg">
			<div class="selectimg-area">
				<div class="selectimg">
					<button type="button" onclick="document.body.removeChild(document.getElementById('selectimg'))">關閉</button>
					<input type="text" name="keyword" size="20" id="keyword">
					<button type="button" v-on:click="getImg()">搜尋</button>
					<ul>
						<li><img src="images/5522_P_20200328085257_1.jpg" v-on:click="addimg(tempSet.m_nth,'images/5522_P_20200328085257_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="images/5523_P_20200325183143_1.jpg" v-on:click="addimg(tempSet.m_nth,'images/5523_P_20200325183143_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="images/5524_P_20200417151430_1.jpg" v-on:click="addimg(tempSet.m_nth,'images/5524_P_20200417151430_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="images/5525_P_20200328083559_1.jpg" v-on:click="addimg(tempSet.m_nth,'images/5525_P_20200328083559_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="images/5526_P_20200328084253_1.jpg" v-on:click="addimg(tempSet.m_nth,'images/5526_P_20200328084253_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="images/5527_P_20200330181837_1.jpg" v-on:click="addimg(tempSet.m_nth,'images/5527_P_20200330181837_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="images/5528_P_20200328082251_1.jpg" v-on:click="addimg(tempSet.m_nth,'images/5528_P_20200328082251_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="images/5529_P_20200328082627_1.jpg" v-on:click="addimg(tempSet.m_nth,'images/5529_P_20200328082627_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="images/5536_P_20200407212522_1.jpg" v-on:click="addimg(tempSet.m_nth,'images/5536_P_20200407212522_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
					</ul>
				</div>
			</div>
		</div>
		<div id="tempSearchImg">
			<div class="selectimg-area">
				<div class="selectimg">
					<button type="button" onclick="document.body.removeChild(document.getElementById('selectimg'))">關閉</button>
					<input type="text" name="keyword" size="20" id="keyword">
					<button type="button" v-on:click="getImg()">搜尋</button>
					<ul>
						<li><img src="./assets/5719_P_20200806154558_1.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5719_P_20200806154558_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="./assets/5720_P_20200806172459_1.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5720_P_20200806172459_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="./assets/5732_P_20200804164529_1.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5732_P_20200804164529_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="./assets/5741_P_20200731175711_1.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5741_P_20200731175711_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="./assets/5742_P_20200731174705_1.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5742_P_20200731174705_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="./assets/5749_P_20200731181755_1.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5749_P_20200731181755_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="./assets/5751_P_20200803164106_5.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5751_P_20200803164106_5.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="./assets/5752_P_20200803173334_1.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5752_P_20200803173334_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="./assets/5754_P_20200731212125_1.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5754_P_20200731212125_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="./assets/5763_P_20200731183842_5.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5763_P_20200731183842_5.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="./assets/5765_P_20200803182012_1.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5765_P_20200803182012_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
						<li><img src="./assets/5767_P_20200804154721_1.jpg" v-on:click="addimg(tempSet.m_nth,'./assets/5767_P_20200804154721_1.jpg',0,0,1,0);document.body.removeChild(document.getElementById('selectimg'))"></li>
					</ul>
				</div>
			</div>
		</div>
		<div id="tempShow">
			<div class="selectimg-area">
				<div class="selectimg">
					<button type="button" onclick="document.body.removeChild(document.getElementById('selectimg'))">關閉</button>
					<img>
				</div>
			</div>
		</div>
		<div id="tempSelectText">
			<input type="text" class="text" style="font-weight: 400;font-size: 36px;line-height: 36px;height: 36px;color:#f00;font-family: Noto Sans TC;" size="3">
			<div class="text-control">
				<button type="button">拖曳移動</button><br>
				<input class="size" type="number" min="6" value="36" title="文字大小"><br>
				<select class="font-type" title="字體">
					<option value="'Noto\ Sans\ TC'" style="font-family:'Noto Sans TC',sans-serif;">Noto Sans TC-範例Sample</option>
					<option value="'Noto\ Serif\ TC'" style="font-family:'Noto Serif TC',sans-serif;">Noto Serif TC-範例Sample</option>
					<option value="'STXihei'" style="font-family:STXihei;">華文細黑-範例Sample</option>
					<option value="'STHeiti'" style="font-family:STHeiti;">華文黑體-範例Sample</option>
					<option value="'STKaiti'" style="font-family:STKaiti;">華文楷體-範例Sample</option>
					<option value="'STSong'" style="font-family:STSong;">華文宋體-範例Sample</option>
					<option value="'STFangsong'" style="font-family:STFangsong;">華文仿宋-範例Sample</option>
					<option value="'LiHei Pro Medium'" style="font-family:LiHei Pro Medium;">儷黑 Pro-範例Sample</option>
					<option value="'LiSong Pro Light'" style="font-family:LiSong Pro Light;">儷宋 Pro-範例Sample</option>
					<option value="'BiauKai'" style="font-family:BiauKai;">標楷體-範例Sample</option>
					<option value="'Apple LiGothic Medium'" style="font-family:Apple LiGothic Medium;">蘋果儷中黑-範例Sample</option>
					<option value="'Apple LiSung Light'" style="font-family:Apple LiSung Light;">蘋果儷細宋-範例Sample</option>

					<option value="'PMingLiU'" style="font-family:PMingLiU;">新細明體-範例Sample</option>
					<option value="'MingLiU'" style="font-family:MingLiU;">細明體-範例Sample</option>
					<option value="'DFKai-SB'" style="font-family:DFKai-SB;">標楷體-範例Sample</option>
					<option value="'SimHei'" style="font-family:SimHei;">黑體-範例Sample</option>
					<option value="'NSimSun'" style="font-family:NSimSun;">新宋體-範例Sample</option>
					<option value="'FangSong'" style="font-family:FangSong;">仿宋-範例Sample</option>
					<option value="'KaiTi'" style="font-family:KaiTi;">楷體-範例Sample</option>
					<option value="'FangSong_GB2312'" style="font-family:FangSong_GB2312;">仿宋_GB2312-範例Sample</option>
					<option value="'KaiTi_GB2312'" style="font-family:KaiTi_GB2312;">楷體_GB2312-範例Sample</option>
					<option value="'Microsoft JhengHei'" style="font-family:Microsoft JhengHei;">微軟正黑體-範例Sample</option>
					<option value="'Microsoft YaHei'" style="font-family:Microsoft YaHei;">微軟雅黑體-範例Sample</option>
				</select><br>
				<select class="font-weight" title="文字粗細">
					<option value="100">細</option>
					<option value="400" selected="selected">標準</option>
					<option value="900">粗</option>
				</select>
				<div class="picker" style="height:20px;width: 20px;background-color:#f00;" title="文字顏色"></div>
				<!--<input type="text" name="" id="">-->
			</div>
		</div>
	</div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'

export default {
	name: 'App',
	/*components: {
		HelloWorld
	}*/
	//props:['prop1','prop2'],
	//created: function () {
	mounted: function() {
		//alert('{{prop1}} created.');
		//if(!this.canvasSet.canvasPosition == false)this.resetCanvas(this.canvasSet.canvasPosition.length);
		//this.addboxSelect();
		//if(!canvasSetData.value == false){
		//console.log(this.todos);
		if(this.canvasSet.canvasPosition.length > 0){
			//載入原始設定
			//canvasSet = JSON.parse(canvasSetData.value);
			this.resetCanvas(this.canvasSet.canvasPosition.length);
			//setTimeout('this.resetCanvas(this.canvasSet.canvasPosition.length)',500);
			//建立文字資料
			/*if(!this.canvasSet.text)this.canvasSet.text = [];
			if(this.canvasSet.text.length > 0){
				for(let i=0;i<this.canvasSet.text.length;i++){
					this.addText(this.canvasSet.text[i],i);
				}
			}*/
		}
		/*window.onresize = function(){
			if(!this.canvasSet.canvasPosition == false)this.resetCanvas(this.canvasSet.canvasPosition.length);
		}*/
	},
	/*computed: {
		// a computed getter
		reversedMessage: function () {
			// `this` points to the vm instance
			return this.message.split('').reverse().join('')
		},
		bsarea() {
			//return '<button></button >'
			return this.addboxSelect();
		}
	},*/
	data(){
		return{
			controls:[],
			pointxys:[],
			todos: [
				{"point":[[0,480,960],[0,960]],"canvasPosition":[[0,0,1,1],[1,0,2,1]]},
				{"point":[[0,960],[0,480,960]],"canvasPosition":[[0,0,1,1],[0,1,1,2]]},
				{"point":[[0,240,480,960],[0,960]],"canvasPosition":[[0,0,1,1],[1,0,2,1],[2,0,3,1]]},
				{"point":[[0,960],[0,240,480,960]],"canvasPosition":[[0,0,1,1],[0,1,1,2],[0,2,1,3]]},
				{"point":[[0,480,960],[0,480,960]],"canvasPosition":[[0,0,1,1],[0,1,1,2],[1,0,2,2]]},
				{"point":[[0,480,960],[0,480,960]],"canvasPosition":[[0,0,1,1],[1,0,2,1],[0,1,2,2]]},
				{"point":[[0,480,960],[0,480,960]],"canvasPosition":[[0,0,2,1],[0,1,1,2],[1,1,2,2]]},
				{"point":[[0,480,960],[0,480,960]],"canvasPosition":[[0,0,1,2],[1,0,2,1],[1,1,2,2]]},
				{"point":[[0,240,480,720,960],[0,960]],"canvasPosition":[[0,0,1,1],[1,0,2,1],[2,0,3,1],[3,0,4,1]]},
				{"point":[[0,960],[0,240,480,720,960]],"canvasPosition":[[0,0,1,1],[0,1,1,2],[0,2,1,3],[0,3,1,4]]},
				{"point":[[0,480,960],[0,480,960]],"canvasPosition":[[0,0,1,1],[1,0,2,1],[0,1,1,2],[1,1,2,2]]},
				{"point":[[0,480,960],[0,320,640,960]],"canvasPosition":[[0,0,1,1],[0,1,1,2],[0,2,1,3],[1,0,2,3]]},
				{"point":[[0,320,640,960],[0,480,960]],"canvasPosition":[[0,0,1,1],[1,0,2,1],[2,0,3,1],[0,1,3,2]]},
				{"point":[[0,320,640,960],[0,480,960]],"canvasPosition":[[0,0,3,1],[0,1,1,2],[1,1,2,2],[2,1,3,2]]},
				{"point":[[0,480,960],[0,320,640,960]],"canvasPosition":[[0,0,1,3],[1,0,2,1],[1,1,2,2],[1,2,2,3]]}
			],
			//carea: '',
			//bsarea: '',
			canvas_area:'',
			earea_style:'transform: scale(1)',
			canvasWidth:0,
			canvasHeight:0,
			//canvaslinewidth:0,
			//canvasSet: {"point":[[0,480,960],[0,480,960]],"canvasPosition":[[0,0,2,1],[0,1,1,2],[1,1,2,2]],"spacing":0,"imgPosition":[["images/5383_P_20200320122209_5.jpg",0,0,1,0],["images/5415_P_20200307012921_1.jpg",0,0,1,0],["images/5428_P_20200307002947_1.jpg",0,-52,1,0]]},
			canvasSet:{
				"point":[],
				"canvasPosition":[],
				"spacing":0,
				"imgPosition":[]
			},
			tempSet: {
				"canvasArea":[],
				"canvasPositionCustom":[],
				//紀錄滑鼠操作事件
				"text_isDown":false,
				"control_isDown":false,
				"ponitxy_isDown":false,
				"canvasCustom_isDown":false,
				//紀錄滑鼠操作數據-自訂區塊原始起點
				"canvasCustom_x":0,
				"canvasCustom_y":0,
				//紀錄滑鼠操作數據
				"m_nth":0,
				"m_move_x":0,
				"m_move_y":0,
				"m_down_x":0,
				"m_down_y":0,
				"m_nth_x":0,
				"m_nth_y":0
			}
		}
	},
	methods:{
		addComment:function (event) {
			if(event.target.nodeName === 'BUTTON'){
				// 获取触发事件对象的属性
				console.log(123);
			}
		},
		resetCanvas: function(size,imgReposition){
			imgReposition = imgReposition || false;
			let barea = document.getElementsByClassName("button-area")[0];// eslint-disable-line no-unused-vars
			this.canvasWidth = this.canvasSet.point[0][this.canvasSet.point[0].length-1];
			this.canvasHeight = this.canvasSet.point[1][this.canvasSet.point[1].length-1];
			this.canvas_area='';
			console.log(this.canvasWidth);
			//設定畫布顯示縮放倍率
			let scale = 1;
			
			let earea_wrap_w = window.innerWidth - 300;
			let earea_wrap_h = window.innerHeight;
			if(this.canvasWidth>earea_wrap_w*0.9){
				this.earea_style ='transform: scale('+(earea_wrap_w*0.9/this.canvasWidth)+')';
				scale = 1/(earea_wrap_w*0.9/this.canvasWidth);
			}
			if(this.canvasHeight*(earea_wrap_w*0.9/this.canvasWidth)>earea_wrap_h*0.9){
				this.earea_style ='transform: scale('+(earea_wrap_h*0.9/this.canvasHeight)+')';
				scale = 1/(earea_wrap_h*0.9/this.canvasHeight);
			}
			for(let i=0;i<document.getElementsByClassName("text-control").length;i++){
				document.getElementsByClassName("text-control")[i].setAttribute("style",'transform: scale('+scale+')');
			}
			//取得畫布可見區域起點終點寬高
			for(let i=0;i<size;i++){
				this.tempSet.canvasArea[i]=[
					this.canvasSet.point[0][this.canvasSet.canvasPosition[i][0]],
					this.canvasSet.point[1][this.canvasSet.canvasPosition[i][1]],
					this.canvasSet.point[0][this.canvasSet.canvasPosition[i][2]],
					this.canvasSet.point[1][this.canvasSet.canvasPosition[i][3]],
					this.canvasSet.point[0][this.canvasSet.canvasPosition[i][2]]-this.canvasSet.point[0][this.canvasSet.canvasPosition[i][0]],
					this.canvasSet.point[1][this.canvasSet.canvasPosition[i][3]]-this.canvasSet.point[1][this.canvasSet.canvasPosition[i][1]]
				];
				//console.log(this.canvasSet.point[0]);
			}
			//console.log(this.canvasSet.point);
			this.controls = [];
			this.pointxys = [];
			for(let i=1;i<=size+1;i++){
				//建立畫布
				let box = document.createElement("canvas");
					if(i<=size){
						//console.log(this.tempSet.canvasArea[i-1][4]);
						box.id = "canvas"+i;
						box.width = this.tempSet.canvasArea[i-1][4]-(+this.canvasSet.spacing*2);
						box.height = this.tempSet.canvasArea[i-1][5]-(+this.canvasSet.spacing*2);
						box.style.left = (this.tempSet.canvasArea[i-1][0]+(+this.canvasSet.spacing)) + "px";
						box.style.top = (this.tempSet.canvasArea[i-1][1]+(+this.canvasSet.spacing)) + "px";
					}else{
						box.id = "finalCanvas";
						box.width = this.canvasWidth;
						box.height = this.canvasHeight;
					}
				let wrap = document.createElement("div");
					wrap.appendChild(box);
					this.canvas_area+=wrap.innerHTML;
				if(i<=size){
					//建立緩衝畫布
					let box = document.createElement("canvas");
						box.id = "tempcanvas"+i;
						box.width = this.tempSet.canvasArea[i-1][4]-(+this.canvasSet.spacing*2);
						box.height = this.tempSet.canvasArea[i-1][5]-(+this.canvasSet.spacing*2);
						box.style.left = (this.tempSet.canvasArea[i-1][0]+(+this.canvasSet.spacing)) + "px";
						//console.log(this.tempSet.canvasArea[i-1][0],this.canvasSet.spacing,this.tempSet.canvasArea[i-1][0]+(+this.canvasSet.spacing))
						box.style.top = (this.tempSet.canvasArea[i-1][1]+(+this.canvasSet.spacing)) + "px";
					let wrap = document.createElement("div");
						wrap.appendChild(box);
						this.canvas_area+=wrap.innerHTML;
					//建立圖片定位紀錄
					if(!this.canvasSet.imgPosition)this.canvasSet.imgPosition=[];
					if(!this.canvasSet.imgPosition[i-1]){
						this.canvasSet.imgPosition.push([]);
					}else{
						this.addimg(i,this.canvasSet.imgPosition[i-1][0],this.canvasSet.imgPosition[i-1][1],this.canvasSet.imgPosition[i-1][2],this.canvasSet.imgPosition[i-1][3],this.canvasSet.imgPosition[i-1][4],imgReposition);
					}
					//建立畫布控制區
					let box2 = {style:"",scale:scale};
					let x = this.tempSet.canvasArea[i-1][0],
						y = this.tempSet.canvasArea[i-1][1],
						w = this.tempSet.canvasArea[i-1][4],
						h = this.tempSet.canvasArea[i-1][5];
						box2.style='top:'+y+'px;left:'+x+'px;width:'+w+'px;height:'+h+'px;';
						this.controls.push(box2);
				}
			}
			//建立畫布可見範圍控制按鈕
			for(let i=1;i<size+1;i++){
				let box = {style:""};
				let style = '';
						if(scale != 1)style='transform:scale('+scale+');';
					box.style='top:'+this.tempSet.canvasArea[i-1][3]+'px;left:'+this.tempSet.canvasArea[i-1][2]+'px;'+style;
					this.pointxys.push(box);
			}
		},
		//置入圖片,
		addimg: function(nth,imgName,imgx,imgy,imgz,imgr,imgReposition){
			imgReposition = imgReposition || false;
			if(!imgName)return false;
			let cvs = document.getElementById("canvas"+nth),
				cvstemp = document.getElementById("tempcanvas"+nth),
				ctx = cvs.getContext("2d"),
				ctxtemp = cvstemp.getContext("2d");
			ctx.globalCompositeOperation="copy";
			let img = new Image();
				img.src=imgName;
				img.setAttribute('crossorigin', 'anonymous');
				img.crossOrigin = 'anonymous';
			imgx = imgx || 0;
			imgy = imgy || 0;
			imgz = imgz || 1;
			imgr = imgr || 0;
			//建立圖片緩衝後清除圖片
			ctxtemp.drawImage(cvs, 0, 0);
			ctx.clearRect(0,0,cvs.width,cvs.height);
			img.onload=function(){
				let imgw = this.tempSet.canvasArea[nth-1][4],
					imgh = img.height*imgw/img.width;
				if(imgh<this.tempSet.canvasArea[nth-1][5]){
					imgh = this.tempSet.canvasArea[nth-1][5];
					imgw = img.width*imgh/img.height;
				}
				//限制移動區域不可超出圖片,增加縮小圖片功能後停用,除非畫布變更
				if(imgReposition==true){
					if(imgx>0 && imgy>0){
						imgx = 0;
						imgy = 0;
					}else if(imgx>0){
						imgx = 0;
					}else if(imgy>0){
						imgy = 0;
					}
					if(imgx+(imgw*imgz)<this.tempSet.canvasArea[nth-1][4] && imgy+(imgh*imgz)<this.tempSet.canvasArea[nth-1][5]){
						imgx = this.tempSet.canvasArea[nth-1][4]-(imgw*imgz);
						imgy = this.tempSet.canvasArea[nth-1][5]-(imgh*imgz);
					}else if(imgx+(imgw*imgz)<this.tempSet.canvasArea[nth-1][4]){
						imgx = this.tempSet.canvasArea[nth-1][4]-(imgw*imgz);
					}else if(imgy+(imgh*imgz)<this.tempSet.canvasArea[nth-1][5]){
						imgy = this.tempSet.canvasArea[nth-1][5]-(imgh*imgz);
					}
				}
				
				//紀錄圖片數據
				this.canvasSet.imgPosition[nth-1]=[imgName,imgx,imgy,imgz,imgr];
				ctx.translate(imgx-(+this.canvasSet.spacing)+(imgw*imgz/2),imgy-(+this.canvasSet.spacing)+(imgh*imgz/2));
				ctx.rotate(imgr/180*Math.PI);
				ctx.drawImage(img,imgx-(imgw*imgz/2), imgy-(imgh*imgz/2), imgw * imgz, imgh * imgz);
				ctx.rotate(-imgr/180*Math.PI);
				ctx.translate(+this.canvasSet.spacing-imgx-(imgw*imgz/2),+this.canvasSet.spacing-imgy-(imgh*imgz/2));//重設參考點為0,0
				//清理重疊區域
				ctxtemp.clearRect(0,0,cvs.width,cvs.height);
			}
		},
		//各圖合併一張圖,
		getBase64: function(){// eslint-disable-line no-unused-vars
			//數據資料轉成json
			//canvasSetData.value = JSON.stringify(this.canvasSet);
			let domContext = finalCanvas.getContext('2d');
				domContext.clearRect(0,0,finalCanvas.width,finalCanvas.height);
				domContext.fillStyle="#ffffff";
				domContext.fillRect(0,0,finalCanvas.width,finalCanvas.height);
			//domContext.scale(2,2);
			for(let i=1;i<=this.tempSet.canvasArea.length;i++){
				let cvs = document.getElementById("canvas"+i);
				domContext.drawImage(cvs, this.tempSet.canvasArea[i-1][0]+(+this.canvasSet.spacing), this.tempSet.canvasArea[i-1][1]+(+this.canvasSet.spacing));
			}
			if(!this.canvasSet.text==false){
				for(let i=0;i<this.canvasSet.text.length;i++){
					domContext.textBaseline = "top";//canvas文字定位從左下角改成上方
					domContext.font = this.canvasSet.text[i].weight + " " + this.canvasSet.text[i].size + "px " + this.canvasSet.text[i].font;
					domContext.fillStyle = this.canvasSet.text[i].color;
					domContext.fillText(this.canvasSet.text[i].text,this.canvasSet.text[i].x,this.canvasSet.text[i].y);
				}
			}
			this.tempSet.dataURL = finalCanvas.toDataURL();   //get base64 img
			let box = document.createElement("div");
				box.id = "selectimg";
				box.innerHTML = document.getElementById("tempShow").innerHTML;
				box.getElementsByTagName("img")[0].setAttribute("src",this.tempSet.dataURL);
			document.body.appendChild(box);
		},
		//設定畫布顯示縮放倍率,
		setScale: function(){
			let earea_wrap = document.getElementsByClassName("edit-box-wrap")[0],
				canvasWidth = this.canvasSet.point[0][this.canvasSet.point[0].length-1],
				canvasHeight = this.canvasSet.point[1][this.canvasSet.point[1].length-1];
			let scale = 1;
			if(canvasWidth>earea_wrap.clientWidth*0.9){
				scale = 1/(earea_wrap.clientWidth*0.9/canvasWidth);
			}
			if(canvasHeight*(earea_wrap.clientWidth*0.9/canvasWidth)>earea_wrap.clientHeight*0.9){
				scale = 1/(earea_wrap.clientHeight*0.9/canvasHeight);
			}
			return scale;
		},
		//拖曳圖片,
		control_down: function(nth){// eslint-disable-line no-unused-vars
			if(this.tempSet.text_isDown == true)return false;
			if(!this.canvasSet.imgPosition[nth-1][0]){
				//searchimg(nth);
				return false;
			}
			this.tempSet.m_nth = nth;
			this.tempSet.control_isDown = true;
			//獲取滑鼠按下時座標
			this.tempSet.m_down_x = event.clientX;
			this.tempSet.m_down_y = event.clientY;
			this.tempSet.m_nth_x = this.canvasSet.imgPosition[this.tempSet.m_nth-1][1];
			this.tempSet.m_nth_y = this.canvasSet.imgPosition[this.tempSet.m_nth-1][2];
		},
		//滑鼠移動,
		move: function(){// eslint-disable-line no-unused-vars
			//獲取滑鼠移動實時座標
			this.tempSet.m_move_x = event.clientX;
			this.tempSet.m_move_y = event.clientY;
			//設定畫布顯示縮放倍率
			//if(this.canvasSet.point.length > 0)var scale = this.setScale();
			//滑鼠按下時移動才觸發
			if(this.tempSet.control_isDown){
				//把新div座標值賦給div物件
				let x = Math.floor(+this.tempSet.m_nth_x + (this.tempSet.m_move_x - this.tempSet.m_down_x)*this.setScale()),
					y = Math.floor(+this.tempSet.m_nth_y + (this.tempSet.m_move_y - this.tempSet.m_down_y)*this.setScale());
				this.addimg(this.tempSet.m_nth,this.canvasSet.imgPosition[this.tempSet.m_nth-1][0],x,y,this.canvasSet.imgPosition[this.tempSet.m_nth-1][3],this.canvasSet.imgPosition[this.tempSet.m_nth-1][4]);
			}
			if(this.tempSet.text_isDown){
				//把新div座標值賦給div物件
				let x = Math.floor(+this.tempSet.m_nth_x + (this.tempSet.m_move_x - this.tempSet.m_down_x)*this.setScale()),
					y = Math.floor(+this.tempSet.m_nth_y + (this.tempSet.m_move_y - this.tempSet.m_down_y)*this.setScale());
				document.getElementById("textBlock"+this.tempSet.m_nth).style.left = x + "px";
				document.getElementById("textBlock"+this.tempSet.m_nth).style.top = y + "px";
				this.canvasSet.text[this.tempSet.m_nth-1].x = x;
				this.canvasSet.text[this.tempSet.m_nth-1].y = y;
			}
			if(this.tempSet.ponitxy_isDown){
				//把新div座標值賦給div物件
				let x = Math.floor(+this.tempSet.m_nth_x + (this.tempSet.m_move_x - this.tempSet.m_down_x)*this.setScale()),
					y = Math.floor(+this.tempSet.m_nth_y + (this.tempSet.m_move_y - this.tempSet.m_down_y)*this.setScale()),
					thisPointxy = document.getElementById("pointxy"+this.tempSet.m_nth);
				//檢查與座標間的距離
				if(
					x<this.canvasSet.point[0][this.canvasSet.canvasPosition[this.tempSet.m_nth-1][2]-1]+10 || 
					y<this.canvasSet.point[1][this.canvasSet.canvasPosition[this.tempSet.m_nth-1][3]-1]+10 || 
					(!this.canvasSet.point[0][this.canvasSet.canvasPosition[this.tempSet.m_nth-1][2]+1] != true && x>this.canvasSet.point[0][this.canvasSet.canvasPosition[this.tempSet.m_nth-1][2]+1]-10) || 
					(!this.canvasSet.point[1][this.canvasSet.canvasPosition[this.tempSet.m_nth-1][3]+1] != true && y>this.canvasSet.point[1][this.canvasSet.canvasPosition[this.tempSet.m_nth-1][3]+1]-10) ||
					x>3000 || y>3000
					)return false;
				thisPointxy.setAttribute("style",'top:'+y+'px;left:'+x+'px;');
				//let style = '';
				//if(scale != 1)style='transform:scale('+scale+');';
				//this.pointxys[this.tempSet.m_nth-1].style=
				console.log(this.pointxys[this.tempSet.m_nth-1].style);
				this.canvasSet.point[0][this.canvasSet.canvasPosition[this.tempSet.m_nth-1][2]]=x;
				this.canvasSet.point[1][this.canvasSet.canvasPosition[this.tempSet.m_nth-1][3]]=y;
				this.resetCanvas(this.canvasSet.canvasPosition.length);
			}
		},
		up: function(){// eslint-disable-line no-unused-vars
			this.tempSet.control_isDown = false;
			this.tempSet.ponitxy_isDown = false;
			this.tempSet.text_isDown = false;
		},
		ponitxy_down: function(nth){// eslint-disable-line no-unused-vars
			if(this.tempSet.text_isDown == true)return false;
			this.tempSet.m_nth = nth;
			this.tempSet.ponitxy_isDown = true;
			//獲取滑鼠按下時座標
			this.tempSet.m_down_x = event.clientX;
			this.tempSet.m_down_y = event.clientY;
			this.tempSet.m_nth_x = this.tempSet.canvasArea[this.tempSet.m_nth-1][2];
			this.tempSet.m_nth_y = this.tempSet.canvasArea[this.tempSet.m_nth-1][3];
		},
		text_down: function(nth){// eslint-disable-line no-unused-vars
			this.tempSet.m_nth = nth;
			this.tempSet.text_isDown = true;
			//獲取滑鼠按下時座標
			this.tempSet.m_down_x = event.clientX;
			this.tempSet.m_down_y = event.clientY;
			this.tempSet.m_nth_x = document.getElementById("textBlock"+nth).offsetLeft;
			this.tempSet.m_nth_y = document.getElementById("textBlock"+nth).offsetTop;
		},
		//event.clientX、event.clientY
		//圖片縮放,
		MouseWheel: function(nth,e){// eslint-disable-line no-unused-vars
			//window.event.shiftKey==1
			if(!this.canvasSet.imgPosition[nth-1][0]){
				return false;
			}
			e = e || window.event;
			if(e.shiftKey){
				let imgr = this.canvasSet.imgPosition[nth-1][4];
				( e.wheelDelta <= 0 || e.detail > 0) ? imgr-=3 : imgr+=3;
				if(imgr<0)imgr+=360;
				if(imgr>360)imgr-=360;
				this.addimg(nth,this.canvasSet.imgPosition[nth-1][0],this.canvasSet.imgPosition[nth-1][1],this.canvasSet.imgPosition[nth-1][2],this.canvasSet.imgPosition[nth-1][3],imgr);
			}else{
				let imgz=this.canvasSet.imgPosition[nth-1][3];
				let imgzo = imgz;
				( e.wheelDelta <= 0 || e.detail > 0) ? imgz-=0.05 : imgz+=0.05;
				imgz = Math.round(imgz*100)/100;
				if(imgz<0.1)imgz=0.1;
				let conterx = this.tempSet.canvasArea[nth-1][4]/2,
					contery = this.tempSet.canvasArea[nth-1][5]/2,
					imgx = Math.round(conterx - ((conterx-this.canvasSet.imgPosition[nth-1][1])*imgz/imgzo)),
					imgy = Math.round(contery - ((contery-this.canvasSet.imgPosition[nth-1][2])*imgz/imgzo));
				this.addimg(nth,this.canvasSet.imgPosition[nth-1][0],imgx,imgy,imgz,this.canvasSet.imgPosition[nth-1][4]);
			}
		},
		//選擇圖片,
		searchimg: function(nth){// eslint-disable-line no-unused-vars
			this.tempSet.m_nth = nth;
			let box = document.createElement("div");
				box.id = "selectimg";
				box.innerHTML = document.getElementById("tempSearchImg").innerHTML;
			document.body.appendChild(box);
		}
		/*var selectimg: function(nth){
			this.tempSet.m_nth = nth;
			let box = document.createElement("div");
				box.id = "selectimg";
				box.innerHTML = document.getElementById("tempSelectImg").innerHTML;
			document.body.appendChild(box);
		}*/,
		canvas_Width: function(){// eslint-disable-line no-unused-vars
			if(this.canvas_check(this.canvasWidth)){
				this.canvasSet.point[0][this.canvasSet.point[0].length-1] = this.canvasWidth;
				this.resetCanvas(this.canvasSet.canvasPosition.length);
			}
			this.canvasWidth = this.canvasSet.point[0][this.canvasSet.point[0].length-1];
		},
		canvas_Height: function(){// eslint-disable-line no-unused-vars
			if(this.canvas_check(this.canvasHeight)){
				this.canvasSet.point[1][this.canvasSet.point[1].length-1] = this.canvasHeight;
				this.resetCanvas(this.canvasSet.canvasPosition.length);
			}
			this.canvasHeight = this.canvasSet.point[1][this.canvasSet.point[1].length-1];
		},
		canvas_line_Width: function(){// eslint-disable-line no-unused-vars
			if(this.canvas_check()){
				//this.canvasSet.spacing = this.canvaslinewidth;
				this.resetCanvas(this.canvasSet.canvasPosition.length);
			}else{
				if(this.canvasSet.spacing>3000){
					this.canvasSet.spacing=3000;
				}else if(this.canvasSet.spacing<0){
					this.canvasSet.spacing=0;
				}
			}
			console.log(this.canvasSet.spacing);
			//this.canvaslinewidth = this.canvasSet.spacing;
		},
		canvas_check: function(value){
			if(value>3000){
				alert("勿超過3000");
				return false;
			}else if(value<0){
				alert("勿為負數");
				return false;
			}
			return true;
		},
		changeTEST: function(){
			console.log("start");
		},
		changeBox: function(arr){
			//arr = JSON.parse(JSON.stringify(arr));
			this.tempSet.canvasPositionCustom=[];
			//this.addboxSelect();
			this.canvasSet.point = arr.point;
			//console.log(arr.point[0]);
			this.canvasSet.canvasPosition = arr.canvasPosition;
			//if(!document.getElementsByName('canvasWidth')[0].value == false){
			if(this.canvasWidth!=0){
				let mag = this.canvasWidth / this.canvasSet.point[0][this.canvasSet.point[0].length-1];
				console.log(this.canvasWidth);
				console.log(mag);
				for(let i=0;i<this.canvasSet.point[0].length;i++){
					this.canvasSet.point[0][i]=this.canvasSet.point[0][i]*mag;
				}
			}else{
				this.canvasWidth = this.canvasSet.point[0][this.canvasSet.point[0].length-1];
			}
			//if(!document.getElementsByName('canvasHeight')[0].value == false){
			if(this.canvasHeight!=0){
				let mag = this.canvasHeight / this.canvasSet.point[1][this.canvasSet.point[1].length-1];
				console.log(mag);
				for(let i=0;i<this.canvasSet.point[1].length;i++){
					this.canvasSet.point[1][i]=this.canvasSet.point[1][i]*mag;
				}
			}else{
				this.canvasHeight = this.canvasSet.point[1][this.canvasSet.point[1].length-1];
			}
			//if(!this.canvasSet.spacing)this.canvasSet.spacing = 0;
			this.resetCanvas(this.canvasSet.canvasPosition.length,true);
		},
		makeSVG: function(tag, attrs) {
			let box = document.createElement("div");
			let el= document.createElementNS('http://www.w3.org/2000/svg', tag);
			for (let k in attrs)
				el.setAttribute(k, attrs[k]);
				box.appendChild(el);
			return box.innerHTML;
		},
		canvasCustom: function(x,y){// eslint-disable-line no-unused-vars
			if(document.getElementById("canvasCustom"+x+y).className == 'lock')return false;
			if(!this.tempSet.canvasCustom_isDown){
				this.tempSet.canvasCustom_x = x;
				this.tempSet.canvasCustom_y = y;
				this.tempSet.canvasCustom_isDown=true;
			}else{
				let tmp_x = this.tempSet.canvasCustom_x,
					tmp_y = this.tempSet.canvasCustom_y,
					temp_error = false;
				if(this.tempSet.canvasCustom_x>x){
					this.tempSet.canvasCustom_x = x;
					x = tmp_x;
				}
				if(this.tempSet.canvasCustom_y>y){
					this.tempSet.canvasCustom_y = y;
					y = tmp_y;
				}
				for(let i=this.tempSet.canvasCustom_x;i<=x;i++){
					for(let j=this.tempSet.canvasCustom_y;j<=y;j++){
						if(document.getElementById("canvasCustom"+i+j).className == 'lock'){
							temp_error = true;
						}
					}
				}
				if(temp_error){
					alert("選取範圍重複");
					this.tempSet.canvasCustom_isDown=false;
					return false;
				}
				let colorStr="";
				let r = function(){return (Math.floor(Math.random()*224+16)).toString(16)};
				//產生一個六位的字串
				colorStr = "#"+r()+r()+r();
				for(let i=this.tempSet.canvasCustom_x;i<=x;i++){
					for(let j=this.tempSet.canvasCustom_y;j<=y;j++){
						document.getElementById("canvasCustom"+i+j).className = 'lock';
						document.getElementById("canvasCustom"+i+j).style.backgroundColor=colorStr;
						document.getElementById("canvasCustom"+i+j).disabled="disabled";
					}
				}
				this.tempSet.canvasPositionCustom.push([this.tempSet.canvasCustom_x,this.tempSet.canvasCustom_y,x+1,y+1])
				this.tempSet.canvasCustom_isDown=false;
				if(document.getElementsByClassName("lock").length==100){
					let point=[[0,96,192,288,384,480,576,672,768,864,960],[0,96,192,288,384,480,576,672,768,864,960]];
					let canvasPosition=[[],[]];
					let arr=[];
					for(let i=0;i<this.tempSet.canvasPositionCustom.length;i++){
						arr = arr.concat(this.tempSet.canvasPositionCustom[i]);
					}
					for(let i=0;i<arr.length/2;i++){
						canvasPosition[0].push(arr[i*2]);
						if(i*2+1<arr.length)canvasPosition[1].push(arr[i*2+1]);
					}
					canvasPosition[0]=canvasPosition[0].filter( (el, i, arr) => arr.indexOf(el) === i).sort(function (a, b) {return a - b});
					canvasPosition[1]=canvasPosition[1].filter( (el, i, arr) => arr.indexOf(el) === i).sort(function (a, b) {return a - b});
					arr = {point:[[],[]]};
					for(let i=0;i<canvasPosition[0].length;i++){
						arr.point[0].push(point[0][canvasPosition[0][i]]);
					}
					for(let i=0;i<canvasPosition[1].length;i++){
						arr.point[1].push(point[1][canvasPosition[1][i]]);
					}
					for(let i=0;i<this.tempSet.canvasPositionCustom.length;i++){
						for(let j=0;j<canvasPosition[0].length;j++){
							if(this.tempSet.canvasPositionCustom[i][0]==canvasPosition[0][j])this.tempSet.canvasPositionCustom[i][0]=j;
							if(this.tempSet.canvasPositionCustom[i][2]==canvasPosition[0][j])this.tempSet.canvasPositionCustom[i][2]=j;
						}
						for(let j=0;j<canvasPosition[1].length;j++){
							if(this.tempSet.canvasPositionCustom[i][1]==canvasPosition[1][j])this.tempSet.canvasPositionCustom[i][1]=j;
							if(this.tempSet.canvasPositionCustom[i][3]==canvasPosition[1][j])this.tempSet.canvasPositionCustom[i][3]=j;
						}
					}
					arr.canvasPosition=this.tempSet.canvasPositionCustom;
					this.changeBox(arr);
					for(let k=0;k<100;k++){
						document.getElementsByClassName("lock")[0].removeAttribute('style');
						document.getElementsByClassName("lock")[0].removeAttribute('disabled');
						document.getElementsByClassName("lock")[0].removeAttribute('class');
					}
				}
			}
		},
		madeSVG: function(nth){
			//return "456";
			//console.log(this.todos[nth])
			//return nth;
			let boxpos = this.todos[nth].point;
			let boxset = this.todos[nth].canvasPosition;
			let boxzoomx = 50/boxpos[0][boxpos[0].length-1];
			let boxzoomy = 50/boxpos[1][boxpos[1].length-1];
			let box = "";
			for(let j=0;j<this.todos[nth].canvasPosition.length;j++){
				let rect = this.makeSVG('rect', {x:boxpos[0][boxset[j][0]]*boxzoomx+1, y:boxpos[1][boxset[j][1]]*boxzoomy+1, height:(boxpos[1][boxset[j][3]]-boxpos[1][boxset[j][1]])*boxzoomx-2, width:(boxpos[0][boxset[j][2]]-boxpos[0][boxset[j][0]])*boxzoomy-2, fill:"#666"});
				//box.getElementsByTagName("svg")[0].appendChild(rect);
				box += rect;
			}
			//console.log(this.todos[nth].point);
			return box;
		},
		addboxSelect: function(){
			//let box = document.createElement("li");
			let box = '<li id="canvasCustomArea">';
				//box.setAttribute("style",'width:220px;height:220px;');
			let htmltmp = '<div style="width:100%;line-height:30px;">自訂圖片配置</div>';
			for(let i=0;i<10;i++){
				for(let j=0;j<10;j++){
					let button = '<button id="canvasCustom'+j+i+'" type="button" v-on:click=\'canvasCustom('+j+','+i+')\'></button>';
					htmltmp += button;
				}
			}
			//box.id = 'canvasCustomArea';
			//box.innerHTML = htmltmp;
			box += htmltmp +'</li><li><button type="button" v-on:click="addText()">TEXT</button></li>';
			//this.bsarea.appendChild(box);
			this.bsarea += box;
			/*let box2 = document.createElement("li");
				box2.innerHTML = '<button type="button" v-on:click="addText()">TEXT</button>';
			this.bsarea.appendChild(box2);*/
		},
		//新增文字,
		addText: function(arr,nth){
			if(this.canvasSet.point.length == 0){
				alert('請先建立畫布');
				return false;
			}
			arr = arr || {text:"",x:0,y:0,size:36,font:"'Noto\\ Sans\\ TC'",color:"#000",weight:400};
			nth = nth+1||document.getElementsByClassName("text-block").length+1;
			let box = document.createElement("div");
				box.id = "textBlock"+nth;
				box.className="text-block";
				box.innerHTML = document.getElementById("tempSelectText").innerHTML;
				box.style.top = "0";
				box.style.left = "0";
				box.getElementsByTagName("Button")[0].setAttribute("onmousedown",'text_down('+nth+')');
				box.getElementsByClassName("picker")[0].setAttribute("class","picker picker"+nth);
				box.getElementsByClassName("picker")[0].setAttribute("data-nth",nth);
				box.setAttribute("onmouseup",'up()');
			let text = box.getElementsByClassName("text")[0];
				text.setAttribute("onchange",'textValue('+nth+')');
				text.value = arr.text;
				text.style.color = arr.color;
			let size = box.getElementsByClassName("size")[0];
				size.setAttribute("onchange",'textSize('+nth+')');
			let type = box.getElementsByClassName("font-type")[0];
				type.setAttribute("onchange",'textFont('+nth+')');
			let weight = box.getElementsByClassName("font-weight")[0];
				weight.setAttribute("onchange",'textWeight('+nth+')');
			let scale = this.setScale();
				box.getElementsByClassName("text-control")[0].setAttribute("style",'transform: scale('+scale+')');
			document.getElementsByClassName("text-area")[0].appendChild(box);
			if(!this.canvasSet.text)this.canvasSet.text = [];
			(this.canvasSet.text.length<nth)?this.canvasSet.text.push(arr):this.canvasSet.text[nth-1]=arr;
			this.textValue(nth);
			this.textSize(nth,arr.size);
			this.textFont(nth,arr.font);
			this.textWeight(nth,arr.wdight);
			document.getElementById("textBlock"+nth).getElementsByClassName("picker")[0].style.backgroundColor = arr.color;
			document.getElementById("textBlock"+nth).style.top = arr.y + "px";
			document.getElementById("textBlock"+nth).style.left = arr.x + "px";
			/*Colorpicker.create({
				bindClass:'picker'+nth, // 这里的picker可随意填 不须要跟我同样
				initColor:arr.color,
				change: function(elem,hex){
				  // elem: 绑定的元素
				  // hex：当前选中颜色的hex值
				  elem.style.backgroundColor = hex;
				  let nth = elem.getAttribute("data-nth");
				  document.getElementById("textBlock"+nth).getElementsByClassName("text")[0].style.color = hex;
				  this.canvasSet.text[nth-1].color = hex;
				}
			})*/
		},
		textValue: function(nth){
			let text = document.getElementById("textBlock"+nth).getElementsByClassName("text")[0];
				text.setAttribute("size",text.value.length*2);
				if(text.value.length==0)text.setAttribute("size",5);
			this.canvasSet.text[nth-1].text = text.value;
		},
		textSize: function(nth,size){
			let text = document.getElementById("textBlock"+nth).getElementsByClassName("text")[0];
				size = size||document.getElementById("textBlock"+nth).getElementsByClassName("size")[0].value;
			this.canvasSet.text[nth-1].size = size;
			text.style.fontSize = size+"px";
			text.style.height = size+"px";
			text.style.lineHeight = size+"px";
		},
		textFont: function(nth,font){
			let text = document.getElementById("textBlock"+nth).getElementsByClassName("text")[0];
				font = font||document.getElementById("textBlock"+nth).getElementsByClassName("font-type")[0].value;
			this.canvasSet.text[nth-1].font = font;
			text.style.fontFamily = font;
		},
		textWeight: function(nth,weight){
			let text = document.getElementById("textBlock"+nth).getElementsByClassName("text")[0];
				weight = weight||document.getElementById("textBlock"+nth).getElementsByClassName("font-weight")[0].value;
			this.canvasSet.text[nth-1].weight = weight;
			text.style.fontWeight = weight;
		},
		getImg: function() {
			// 發送 Ajax 查詢請求並處理
			var request = new XMLHttpRequest();
			request.open("GET", "img.php?img=" + document.getElementById("keyword").value);
			request.send();
			document.body.removeChild(document.getElementById('selectimg'));
			request.onreadystatechange = function() {
				// 伺服器請求完成
				if (request.readyState === 4) {
					// 伺服器回應成功
					if (request.status === 200) {
						var type = request.getResponseHeader("Content-Type");   // 取得回應類型

						// 判斷回應類型
						if (type.indexOf("application/json") === 0) {			   
							let data = JSON.parse(request.responseText);
							let box = document.createElement("div");
								box.id = "selectimg";
								box.innerHTML = document.getElementById("tempSelectImg").innerHTML;
							if (data.images) {
								let images = "";
								for(let i=0;i<data.images.length;i++){
									images+='<li><img src="images/'+data.images[i]+'" v-on:click="addimg(tempSet.m_nth,\''+data.images[i]+'\',0,0,1,0);document.body.removeChild(document.getElementById(\'selectimg\'))"></li>';
								}
								box.getElementsByTagName("ul")[0].innerHTML = images;
							} else {
								box.getElementsByTagName("ul")[0].innerHTML = data.msg;
							}
							document.body.appendChild(box);
						}
					} else {
						alert("發生錯誤: " + request.status);
					}
				}
			}
		}
	}
}
		//var arr = '{"point":[[0,400,800],[0,400,800]],"canvasPosition":[[0,0,1,1],[0,1,1,2],[1,0,2,2]],"imgPosition":[[0,0,1,"img.jpg"],[0,0,1,"img2.jpg"]],"spacing":1}';
		/*var canvasSet = {};
		var tempSet = {
			"canvasArea":[],
			"canvasPositionCustom":[],
			//紀錄滑鼠操作事件
			"text_isDown":false,
			"control_isDown":false,
			"ponitxy_isDown":false,
			"canvasCustom_isDown":false,
			//紀錄滑鼠操作數據-自訂區塊原始起點
			"canvasCustom_x":0,
			"canvasCustom_y":0,
			//紀錄滑鼠操作數據
			"m_nth":0,
			"m_move_x":0,
			"m_move_y":0,
			"m_down_x":0,
			"m_down_y":0,
			"m_nth_x":0,
			"m_nth_y":0
		};
		this.addboxSelect();
		if(!canvasSetData.value == false){
			//載入原始設定
			canvasSet = JSON.parse(canvasSetData.value);
			this.resetCanvas(this.canvasSet.canvasPosition.length);
			//建立文字資料
			if(!this.canvasSet.text)this.canvasSet.text = [];
			if(this.canvasSet.text.length > 0){
				for(let i=0;i<this.canvasSet.text.length;i++){
					this.addText(this.canvasSet.text[i],i);
				}
			}
		}
		window.onresize = function(){
			if(!this.canvasSet.canvasPosition == false)this.resetCanvas(this.canvasSet.canvasPosition.length);
		}*/
</script>

<style>
	body{
		margin: 0;
		position: relative;
		background-color: #666;
	}
	ul{
		margin: 0;
		padding: 0;
	}
	canvas:not(#finalCanvas){
		display: block;
		position: absolute;
		top: 0;
		left: 0;
	}
	#finalCanvas{
		vertical-align: middle;
		opacity: 0;
	}
	#canvasCustomArea{
		width: 220px;
		/*height: 220px;*/
		padding: 10px;
		flex-wrap: wrap;
		overflow: auto;
		height: auto;
	}
	#canvasCustomArea button{
		width: 22px;
		height: 22px;
		float: left;
		border-width: 1px;
	}
	.box-select{
		width: 240px;
		height: 100vh;
		padding: 0 15px;
		float: left;
		border-right: 1px solid #999;
		background-color: #fff;
	}
	.box-select li{
		width: 80px;
		height: 80px;
		float: left;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.box-select button{
		width: 60px;
		height: 60px;
		padding: 0;
		cursor: pointer;
	}
	.box-select svg{
		width: 50px;
		height: 50px;
		margin: 3px;
	}
	.edit-box-wrap{
		width: calc(100vw - 300px);
		height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
		float: left;
	}
	.edit-box{
		position: relative;
		box-shadow: 0 0 5px #666;
		background-color: #fff;
	}
	.button-area{
		width: calc(100vw - 301px);
		position: fixed;
		right: 0;
		top: 0;
		z-index: 100;
		padding: 10px 15px;
		background-color: #fff;
		box-shadow: 0 2px 2px rgba(0,0,0,0.2);
	}
	.button-area button, .button-area input{
		width: 80px;
		text-align: center;
		/*transform: skewY(30deg);
		transform-origin: top left;*/
	}
	.button-area button{
		float: right;
	}
	.control-area, .text-area{
		position: absolute;
		top: 0;
		left: 0;
	}
	.text-area{
		width: 0;
		height: 0;
	}
	.text-block:hover{
		z-index: 100;
	}
	.control-area div, .text-area .text-block{
		position: absolute;
	}
	.control-area div:not(.pointxy){
		border: 1px dashed rgba(0,0,0,0.2);
		box-sizing: border-box;
	}
	.control-area div button{
		position: absolute;
		top: 5px;
		right: 5px;
		display: none;
		background-color: #2aa4b9;
		color: #fff;
		cursor: pointer;
		border-width: 0;
		box-shadow: 0 1px 2px #666;
		transform-origin:right top;
	}
	.text-area div button{
		background-color: #2aa4b9;
		color: #fff;
		cursor: pointer;
		border-width: 0;
		box-shadow: 0 1px 2px #666;
	}
	.control-area div:hover button, .text-block:hover .text-control, .text-area input[type="text"]:focus + .text-control{
		display: block;
	}
	.text-area input[type="text"]{
		width: auto;
		height: auto;
		padding: 0;
		border: 0;
		outline: 1px dashed rgba(0,0,0,0.4);
		background-color: transparent;
	}
	.text-control{
		display: none;
		background-color: rgba(0,0,0,0.4);
		padding: 10px;
		font-size: 24px;
		transform-origin: 0 0;
		width: 160px;
	}
	.text-control input, .text-control select{
		width: 160px;
		box-sizing: border-box;
		text-align: center;
	}
	.selectimg-area{
		position: fixed;
		height: 100vh;
		width: 100vw;
		z-index: 102;
		display: flex;
		justify-content: center;
		align-items: center;
		top: 0;
		left: 0;
		background-color: rgba(0,0,0,0.5);
	}
	.selectimg{
		position: relative;
		width: 800px;
		padding: 30px 50px;
		background-color: rgba(0,0,0,0.4);
		box-shadow: 0 0 2px rgba(0,0,0,0.5);
		max-height: 640px;
		overflow-y: auto;
	}
	.selectimg li{
		float: left;
		width: 20%;
		padding: 10px;
		box-sizing: border-box;
		list-style: none;
	}
	.selectimg img{
		width: 100%;
	}
	.selectimg button:first-of-type{
		position: absolute;
		top: 0;
		right: 0;
	}
	.pointxy{
		width: 8px;
		height: 8px;
		border: 4px solid #999;
		border-radius: 8px;
		margin-top: -8px;
		margin-left: -8px;
		background-color: #fff;
		position: absolute;
	}
	.pointxy:hover{
		cursor: pointer;
		border: 4px solid #2aa4b9;
		box-shadow: 0 1px 3px #333;
	}
	div.canvas-text{
		position: relative;
	}
	#tempSelectImg,#tempSearchImg,#tempShow,#tempSelectText{
		display:none;
	}
</style>
