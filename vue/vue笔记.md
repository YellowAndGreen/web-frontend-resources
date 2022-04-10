# vue.js

## 基础知识

Vue.config：全局配置

可通过`Vue.config.productionTip = false`阻止 vue 在启动时生成生产提示。

favicon.ico：网站图标，放在根目录下

### 创建Vue容器

使用el绑定dom元素，data为传入的数据，在dom使用{{name}}来获取数据

```vue
//创建Vue实例
new Vue({
el:'#demo', //el用于指定当前Vue实例为哪个容器服务，值通常为css选择器字符串。
data:{ //data中用于存储数据，数据供el所指定的容器去使用，值我们暂时先写成一个对象。
name:'atguigu',
address:'北京'
}
})  
```

或使用v.mount('#root')来绑定元素

```
		/* const v = new Vue({
			//el:'#root', //第一种写法
			data:{
				name:'尚硅谷'
			}
		})
		console.log(v)
		v.$mount('#root') //第二种写法 */
```

### 模版语法

#### 模版字符串${}

#### 插值语法{{data}}

使用``来构建字符串，里面可以使用vm的数据比如：

```js
axios.get(`https://api.github.com/search/users?q=${this.keyWord}`).then(
```



#### 指令语法

##### 绑定标签属性v-bind（缩写为:）

`<span v-bind:title="message">   11  </span>`

<mark>v-bind是单向绑定，数据只能从data中传入，而不能从dom中传出</mark>

##### 绑定表单类元素v-model

`<input type="text" v-model:value="name">`

缩写为：

`<input type="text" v-model="name"><br/>`

<mark>v-model是双向绑定，dom和data中的数据同步</mark>

使用`v-model.number`将表单值强制转换

##### 使用@xxx 绑定事件

​              1.事件的回调需要配置在methods对象中，最终会在vm上；

​              2.methods中配置的函数，不要用箭头函数！否则this就不是vm了；

​              3.methods中配置的函数，都是被Vue所管理的函数，this的指向是vm 或 组件实例对象；

​              4.@click="demo" 和 @click="demo($event)" 效果一致，但后者可以传参；

```html
		<div id="root">
			<h2>欢迎来到{{name}}学习</h2>
			<!-- <button v-on:click="showInfo">点我提示信息</button> -->
			<button @click="showInfo1">点我提示信息1（不传参）</button>
			<button @click="showInfo2($event,66)">点我提示信息2（传参）</button>
		</div>
```

```vue
		const vm = new Vue({
			el:'#root',
			data:{
				name:'尚硅谷',
			},
			methods:{
				showInfo1(event){
					// console.log(event.target.innerText)
					// console.log(this) //此处的this是vm
					alert('同学你好！')
				},
				showInfo2(event,number){
					console.log(event,number)
					// console.log(event.target.innerText)
					// console.log(this) //此处的this是vm
					alert('同学你好！！')
				}
			}
		})
```

###### 事件修饰符

@click.prevent.stop：可以连写

Vue中的事件修饰符：

​            1.prevent：阻止默认事件（常用）；

​            2.stop：阻止事件冒泡（常用）；

​            3.once：事件只触发一次（常用）；

​            4.capture：使用事件的捕获模式；

​            5.self：只有event.target是当前操作的元素时才触发事件；

​            6.passive：事件的默认行为立即执行，无需等待事件回调执行完毕；

示例：

```html
<div id="root">
	<h2>欢迎来到{{name}}学习</h2>
	<!-- 阻止默认事件（常用） -->
	<a href="http://www.atguigu.com" @click.prevent="showInfo">点我提示信息</a>

	<!-- 阻止事件冒泡（常用） -->
	<div class="demo1" @click="showInfo">
		<button @click.stop="showInfo">点我提示信息</button>
		<!-- 修饰符可以连续写 -->
		<!-- <a href="http://www.atguigu.com" @click.prevent.stop="showInfo">点我提示信息</a> -->
	</div>

	<!-- 事件只触发一次（常用） -->
	<button @click.once="showInfo">点我提示信息</button>

	<!-- 使用事件的捕获模式 -->
	<div class="box1" @click.capture="showMsg(1)">
		div1
		<div class="box2" @click="showMsg(2)">
			div2
		</div>
	</div>

	<!-- 只有event.target是当前操作的元素时才触发事件； -->
	<div class="demo1" @click.self="showInfo">
		<button @click="showInfo">点我提示信息</button>
	</div>

	<!-- 事件的默认行为立即执行，无需等待事件回调执行完毕； -->
	<ul @wheel.passive="demo" class="list">
		<li>1</li>
		<li>2</li>
		<li>3</li>
		<li>4</li>
	</ul>

</div>
```



###### 键盘事件

1.Vue中常用的按键别名：

​      回车 => enter

​      删除 => delete (捕获“删除”和“退格”键)

​      退出 => esc

​      空格 => space

​      换行 => tab (特殊，必须配合keydown去使用)

​      上 => up

​      下 => down

​      左 => left

​      右 => right

2.Vue未提供别名的按键，可以使用按键原始的key值去绑定，但注意要转为kebab-case（短横线命名）

3.系统修饰键（用法特殊）：ctrl、alt、shift、meta

​      (1).配合keyup使用：按下修饰键的同时，再按下其他键，随后释放其他键，事件才被触发。

​      (2).配合keydown使用：正常触发事件。

4.也可以使用keyCode去指定具体的按键（不推荐）

5.Vue.config.keyCodes.自定义键名 = 键码，可以去定制按键别名

6.可以组合多个键：@keydown.ctrl.y为ctrl+y

示例：

```html
	<div id="root">
		<h2>欢迎来到{{name}}学习</h2>
		<input type="text" placeholder="按下回车提示输入" @keydown.huiche="showInfo">
	</div>

<script type="text/javascript">
	Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。
	Vue.config.keyCodes.huiche = 13 //定义了一个别名按键

	new Vue({
		el:'#root',
		data:{
			name:'尚硅谷'
		},
		methods: {
			showInfo(e){
				// console.log(e.key,e.keyCode)
				console.log(e.target.value)
			}
		},
	})
</script>
```





### 属性

#### data

data属性有两种写法：

```
new Vue({
	el:'#root',
	//data的第一种写法：对象式
	/* data:{
		name:'尚硅谷'
	} */

	//data的第二种写法：函数式
	data(){
		console.log('@@@',this) //此处的this是Vue实例对象
		return{
			name:'尚硅谷'
		}
	}
})
```

#### methods

```vue
new Vue({
	el:'#root',
	data:{
		firstName:'张',
		lastName:'三'
	},
	methods: {
		fullName(){
			console.log('@---fullName')
			return this.firstName + '-' + this.lastName
		}
	},
})
```

#### 计算属性computed

  1.定义：要用的属性不存在，要通过已有属性计算得来。

  2.原理：底层借助了Objcet.defineproperty方法提供的getter和setter。

  3.get函数什么时候执行？

​        (1).初次读取时会执行一次。

​        (2).当依赖的数据发生改变时会被再次调用。

  4.优势：与methods实现相比，内部有缓存机制（复用），效率更高，调试方便。

  5.备注：

​      1.计算属性最终会出现在vm上，直接读取使用即可。

​      2.如果计算属性要被修改，那必须写set函数去响应修改，且set中要引起计算时依赖的数据发生改变。

​	   3.若不需要set函数修改，则可使用简写形式

```vue
const vm = new Vue({
	el:'#root',
	data:{
		firstName:'张',
		lastName:'三',
	},
	computed:{
		//完整写法
		/* fullName:{
			get(){
				console.log('get被调用了')
				return this.firstName + '-' + this.lastName
			},
			set(value){
				console.log('set',value)
				const arr = value.split('-')
				this.firstName = arr[0]
				this.lastName = arr[1]
			}
		} */
		//简写
		fullName(){
			console.log('get被调用了')
			return this.firstName + '-' + this.lastName
		}
	}
})
```

#### 监视属性watch

三元表达式：`isHot?"炎热":"凉爽`，若为真则炎热，假则凉爽

  1.当被监视的属性变化时, 回调函数自动调用, 进行相关操作

  2.监视的属性必须存在，才能进行监视！！

  3.监视的两种写法：

​      (1).new Vue时传入watch配置

​      (2).通过vm.$watch监视

示例：

```javascript
const vm = new Vue({
	el:'#root',
	data:{
		isHot:true,
	},
	computed:{
		info(){
			return this.isHot ? '炎热' : '凉爽'
		}
	},
	methods: {
		changeWeather(){
			this.isHot = !this.isHot
		}
	},
	/* watch:{
		isHot:{
			immediate:true, //初始化时让handler调用一下
			//handler什么时候调用？当isHot发生改变时。
			handler(newValue,oldValue){
				console.log('isHot被修改了',newValue,oldValue)
			}
		}
	} */
})

vm.$watch('isHot',{
	immediate:true, //初始化时让handler调用一下
	//handler什么时候调用？当isHot发生改变时。
	handler(newValue,oldValue){
		console.log('isHot被修改了',newValue,oldValue)
	}
})
```

##### 深度监视

​    (1).Vue中的watch默认不监测对象内部值的改变（一层）。

​    (2).配置deep:true可以监测对象内部值改变（多层）。

​    (3).Vue自身可以监测对象内部值的改变，但Vue提供的watch默认不可以！

​    (4).使用watch时根据数据的具体结构，决定是否采用深度监视，以增加效率

##### 简写

```javascript
watch:{
	//正常写法
	/* isHot:{
		// immediate:true, //初始化时让handler调用一下
		// deep:true,//深度监视
		handler(newValue,oldValue){
			console.log('isHot被修改了',newValue,oldValue)
		}
	}, */
	//简写
	/* isHot(newValue,oldValue){
		console.log('isHot被修改了',newValue,oldValue,this)
	} */
}
})

//正常写法
/* vm.$watch('isHot',{
	immediate:true, //初始化时让handler调用一下
	deep:true,//深度监视
	handler(newValue,oldValue){
		console.log('isHot被修改了',newValue,oldValue)
	}
}) */

//简写
/* vm.$watch('isHot',(newValue,oldValue)=>{
	console.log('isHot被修改了',newValue,oldValue,this)
}) */
```

##### computed和watch之间的区别

​    1.computed能完成的功能，watch都可以完成。

​    2.watch能完成的功能，computed不一定能完成，例如：watch可以进行异步操作。

### 绑定样式

​    1. class样式

​          写法:class="xxx" xxx可以是字符串、对象、数组。

​              字符串写法适用于：类名不确定，要动态获取。

​              对象写法适用于：要绑定多个样式，个数不确定，名字也不确定。

​              数组写法适用于：要绑定多个样式，个数确定，名字也确定，但不确定用不用。

示例：

```html
	<div id="root">
		<!-- 绑定class样式--字符串写法，适用于：样式的类名不确定，需要动态指定 -->
		<div class="basic" :class="mood" @click="changeMood">{{name}}</div> <br/><br/>

		<!-- 绑定class样式--数组写法，适用于：要绑定的样式个数不确定、名字也不确定 -->
		<div class="basic" :class="classArr">{{name}}</div> <br/><br/>

		<!-- 绑定class样式--对象写法，适用于：要绑定的样式个数确定、名字也确定，但要动态决定用不用 -->
		<div class="basic" :class="classObj">{{name}}</div> <br/><br/>

		<!-- 绑定style样式--对象写法 -->
		<div class="basic" :style="styleObj">{{name}}</div> <br/><br/>
		<!-- 绑定style样式--数组写法 -->
		<div class="basic" :style="styleArr">{{name}}</div>
	</div>
</body>

<script type="text/javascript">
	Vue.config.productionTip = false
	
	const vm = new Vue({
		el:'#root',
		data:{
			name:'尚硅谷',
			mood:'normal',
			classArr:['atguigu1','atguigu2','atguigu3'],
			classObj:{
				atguigu1:false,
				atguigu2:false,
			},
			styleObj:{
				fontSize: '40px',
				color:'red',
			},
			styleObj2:{
				backgroundColor:'orange'
			},
			styleArr:[
				{
					fontSize: '40px',
					color:'blue',
				},
				{
					backgroundColor:'gray'
				}
			]
		},
		methods: {
			changeMood(){
				const arr = ['happy','sad','normal']
				const index = Math.floor(Math.random()*3)
				this.mood = arr[index]
			}
		},
	})
```



​    2. style样式

​          :style="{fontSize: xxx}"其中xxx是动态值。

​          :style="[a,b]"其中a、b是样式对象。

### 条件渲染

​      1.v-if

​            写法：

​                (1).v-if="表达式" 

​                (2).v-else-if="表达式"

​                (3).v-else="表达式"

​            适用于：切换频率较低的场景。

​            特点：不展示的DOM元素直接被移除。

​            注意：v-if可以和:v-else-if、v-else一起使用，但要求结构不能被“打断”。

​      2.v-show

​            写法：v-show="表达式"

​            适用于：切换频率较高的场景。

​            特点：不展示的DOM元素未被移除，仅仅是使用样式隐藏掉        

​      3.备注：使用v-if的时，元素可能无法获取到，而使用v-show一定可以获取到。

示例：

```html
<div id="root">
	<h2>当前的n值是:{{n}}</h2>
	<button @click="n++">点我n+1</button>
	<!-- 使用v-show做条件渲染 -->
	<!-- <h2 v-show="false">欢迎来到{{name}}</h2> -->
	<!-- <h2 v-show="1 === 1">欢迎来到{{name}}</h2> -->

	<!-- 使用v-if做条件渲染 -->
	<!-- <h2 v-if="false">欢迎来到{{name}}</h2> -->
	<!-- <h2 v-if="1 === 1">欢迎来到{{name}}</h2> -->

	<!-- v-else和v-else-if -->
	<!-- <div v-if="n === 1">Angular</div>
	<div v-else-if="n === 2">React</div>
	<div v-else-if="n === 3">Vue</div>
	<div v-else>哈哈</div> -->

	<!-- v-if与template的配合使用，template不会实际渲染在页面中 -->
	<template v-if="n === 1">
		<h2>你好</h2>
		<h2>尚硅谷</h2>
		<h2>北京</h2>
	</template>

		</div>
```

### 列表渲染

#### v-for指令

​    1.用于展示列表数据

​    2.语法：v-for="(item, index) in xxx" :key="yyy"

​    3.可遍历：数组、对象、字符串（用的很少）、指定次数（用的很少）

示例：

```html
<div id="root">
	<!-- 遍历数组 -->
	<h2>人员列表（遍历数组）</h2>
	<ul>
		<li v-for="(p,index) of persons" :key="index">
			{{p.name}}-{{p.age}}
		</li>
	</ul>

	<!-- 遍历对象 -->
	<h2>汽车信息（遍历对象）</h2>
	<ul>
		<li v-for="(value,k) of car" :key="k">
			{{k}}-{{value}}
		</li>
	</ul>

	<!-- 遍历字符串 -->
	<h2>测试遍历字符串（用得少）</h2>
	<ul>
		<li v-for="(char,index) of str" :key="index">
			{{char}}-{{index}}
		</li>
	</ul>
	
	<!-- 遍历指定次数 -->
	<h2>测试遍历指定次数（用得少）</h2>
	<ul>
		<li v-for="(number,index) of 5" :key="index">
			{{index}}-{{number}}
		</li>
	</ul>
</div>

<script type="text/javascript">
	Vue.config.productionTip = false
	
	new Vue({
		el:'#root',
		data:{
			persons:[
				{id:'001',name:'张三',age:18},
				{id:'002',name:'李四',age:19},
				{id:'003',name:'王五',age:20}
			],
			car:{
				name:'奥迪A8',
				price:'70万',
				color:'黑色'
			},
			str:'hello'
		}
	})
</script>
```

#### key的原理

\1. 虚拟DOM中key的作用：

​        key是虚拟DOM对象的标识，当数据发生变化时，Vue会根据【新数据】生成【新的虚拟DOM】, 

​        随后Vue进行【新虚拟DOM】与【旧虚拟DOM】的差异比较，比较规则如下：

​        

2.对比规则：

​      (1).旧虚拟DOM中找到了与新虚拟DOM相同的key：

​            ①.若虚拟DOM中内容没变, 直接使用之前的真实DOM！

​            ②.若虚拟DOM中内容变了, 则生成新的真实DOM，随后替换掉页面中之前的真实DOM。



​      (2).旧虚拟DOM中未找到与新虚拟DOM相同的key

​            创建新的真实DOM，随后渲染到到页面。

​            

\3. 用index作为key可能会引发的问题：

​          \1. 若对数据进行：逆序添加、逆序删除等破坏顺序操作:

​                  会产生没有必要的真实DOM更新 ==> 界面效果没问题, 但效率低。



​          \2. 如果结构中还包含输入类的DOM：

​                  会产生错误DOM更新 ==> 界面有问题。



\4. 开发中如何选择key?:

​          1.最好使用每条数据的唯一标识作为key, 比如id、手机号、身份证号、学号等唯一值。

​          2.如果不存在对数据的逆序添加、逆序删除等破坏顺序操作，仅用于渲染列表用于展示，使用index作为key是没有问题的。

#### 列表的过滤和排序

```javascript
new Vue({
	el:'#root',
	data:{
		keyWord:'',
		sortType:0, //0原顺序 1降序 2升序
		persons:[
			{id:'001',name:'马冬梅',age:30,sex:'女'},
			{id:'002',name:'周冬雨',age:31,sex:'女'},
			{id:'003',name:'周杰伦',age:18,sex:'男'},
			{id:'004',name:'温兆伦',age:19,sex:'男'}
		]
	},
	computed:{
		filPerons(){
			const arr = this.persons.filter((p)=>{
				return p.name.indexOf(this.keyWord) !== -1
			})
			//判断一下是否需要排序
			if(this.sortType){
				arr.sort((p1,p2)=>{
					return this.sortType === 1 ? p2.age-p1.age : p1.age-p2.age
				})
			}
			return arr
		}
	}
}) 
```



### MVVM

1. M：模型(Model) ：对应data 中的数据
2. V：视图(View) ：模板
3. VM：视图模型(ViewModel) ： Vue 实例对象

因此一般用vm命名Vue 实例对象

![image-20211028211433269](.\img\image-20211028211433269.png)

​			5.data中所有的属性，最后都出现在了vm身上。

​            6.vm身上所有的属性 及 Vue原型上所有属性，在Vue模板中都可以直接使用。

### 数据代理

<mark>数据代理：通过一个对象代理对另一个对象中属性的操作（读/写）</mark>

#### Object.defineProperty方法

```
let number = 18
let person = {
	name:'张三',
	sex:'男',
}

Object.defineProperty(person,'age',{
	// value:18,
	// enumerable:true, //控制属性是否可以枚举，默认值是false
	// writable:true, //控制属性是否可以被修改，默认值是false
	// configurable:true //控制属性是否可以被删除，默认值是false

	//当有人读取person的age属性时，get函数(getter)就会被调用，且返回值就是age的值
	get(){
		console.log('有人读取age属性了')
		return number
	},

	//当有人修改person的age属性时，set函数(setter)就会被调用，且会收到修改的具体值
	set(value){
		console.log('有人修改了age属性，且值是',value)
		number = value
	}

})
```



#### Vue中的数据代理

通过Object.defineProperty方法来实现，因此获取数据不需要{{_data.name}}而是{{name}}

只有data里的数据才会进行数据代理

![image-20211028214313265](.\img\image-20211028214313265.png)

### Vue监视数据的原理

  \1. vue会监视data中所有层次的数据。

  \2. 如何监测对象中的数据？

​          通过setter实现监视，且要在new Vue时就传入要监测的数据。

​            (1).对象中后追加的属性，Vue默认不做响应式处理

​            (2).***已经废弃***！！！！如需给后添加的属性做响应式，请使用如下API：

​                    Vue.set(target，propertyName/index，value) 或 

​                    vm.$set(target，propertyName/index，value)

​			使用reactive()包裹对象即可开启响应式！！！

  \3. 如何监测数组中的数据？

​            通过包裹数组更新元素的方法实现，本质就是做了两件事：

​              (1).调用原生对应的方法对数组进行更新。

​              (2).重新解析模板，进而更新页面。

  4.在Vue修改数组中的某个元素一定要用如下方法：

​        1.使用这些API:push()、pop()、shift()、unshift()、splice()、sort()、reverse()

​        2.Vue.set() 或 vm.$set()

  特别注意：Vue.set() 和 vm.$set() 不能给vm 或 vm的根数据对象 添加属性！！！

示例：

```html
	<div id="root">
		<h1>学生信息</h1>
		<button @click="student.age++">年龄+1岁</button> <br/>
		<button @click="addSex">添加性别属性，默认值：男</button> <br/>
		<button @click="student.sex = '未知' ">修改性别</button> <br/>
		<button @click="addFriend">在列表首位添加一个朋友</button> <br/>
		<button @click="updateFirstFriendName">修改第一个朋友的名字为：张三</button> <br/>
		<button @click="addHobby">添加一个爱好</button> <br/>
		<button @click="updateHobby">修改第一个爱好为：开车</button> <br/>
		<button @click="removeSmoke">过滤掉爱好中的抽烟</button> <br/>
		<h3>姓名：{{student.name}}</h3>
		<h3>年龄：{{student.age}}</h3>
		<h3 v-if="student.sex">性别：{{student.sex}}</h3>
		<h3>爱好：</h3>
		<ul>
			<li v-for="(h,index) in student.hobby" :key="index">
				{{h}}
			</li>
		</ul>
		<h3>朋友们：</h3>
		<ul>
			<li v-for="(f,index) in student.friends" :key="index">
				{{f.name}}--{{f.age}}
			</li>
		</ul>
	</div>
</body>

<script type="text/javascript">
	Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

	const vm = new Vue({
		el:'#root',
		data:{
			student:{
				name:'tom',
				age:18,
				hobby:['抽烟','喝酒','烫头'],
				friends:[
					{name:'jerry',age:35},
					{name:'tony',age:36}
				]
			}
		},
		methods: {
			addSex(){
				// Vue.set(this.student,'sex','男')
				this.$set(this.student,'sex','男')
			},
			addFriend(){
				this.student.friends.unshift({name:'jack',age:70})
			},
			updateFirstFriendName(){
				this.student.friends[0].name = '张三'
			},
			addHobby(){
				this.student.hobby.push('学习')
			},
			updateHobby(){
				// this.student.hobby.splice(0,1,'开车')
				// Vue.set(this.student.hobby,0,'开车')
				this.$set(this.student.hobby,0,'开车')
			},
			removeSmoke(){
				this.student.hobby = this.student.hobby.filter((h)=>{
					return h !== '抽烟'
				})
			}
		}
	})
</script>
```

### 收集表单数据

​    若：<input type="text"/>，则v-model收集的是value值，用户输入的就是value值。

​    若：<input type="radio"/>，则v-model收集的是value值，且要给标签配置value值。

​    若：<input type="checkbox"/>

​        1.没有配置input的value属性，那么收集的就是checked（勾选 or 未勾选，是布尔值）

​        2.配置input的value属性:

​            (1)v-model的初始值是非数组，那么收集的就是checked（勾选 or 未勾选，是布尔值）

​            (2)v-model的初始值是数组，那么收集的的就是value组成的数组

​    备注：v-model的三个修饰符：

​            lazy：失去焦点再收集数据

​            number：输入字符串转为有效的数字

​            trim：输入首尾空格过滤

示例：

```html
<div id="root">
<form @submit.prevent="demo">
	账号：<input type="text" v-model.trim="userInfo.account"> <br/><br/>
	密码：<input type="password" v-model="userInfo.password"> <br/><br/>
	年龄：<input type="number" v-model.number="userInfo.age"> <br/><br/>
	性别：
	男<input type="radio" name="sex" v-model="userInfo.sex" value="male">
	女<input type="radio" name="sex" v-model="userInfo.sex" value="female"> <br/><br/>
	爱好：
	学习<input type="checkbox" v-model="userInfo.hobby" value="study">
	打游戏<input type="checkbox" v-model="userInfo.hobby" value="game">
	吃饭<input type="checkbox" v-model="userInfo.hobby" value="eat">
	<br/><br/>
	所属校区
	<select v-model="userInfo.city">
		<option value="">请选择校区</option>
		<option value="beijing">北京</option>
		<option value="shanghai">上海</option>
		<option value="shenzhen">深圳</option>
		<option value="wuhan">武汉</option>
	</select>
	<br/><br/>
	其他信息：
	<textarea v-model.lazy="userInfo.other"></textarea> <br/><br/>
	<input type="checkbox" v-model="userInfo.agree">阅读并接受<a href="http://www.atguigu.com">《用户协议》</a>
	<button>提交</button>
</form>
</div>
```

### 过滤器

  定义：对要显示的数据进行特定格式化后再显示（适用于一些简单逻辑的处理）。

  语法：

​      1.注册过滤器：Vue.filter(name,callback) 或 new Vue{filters:{}}

​      2.使用过滤器：{{ xxx | 过滤器名}}  或  v-bind:属性 = "xxx | 过滤器名"

  备注：

​      1.过滤器默认接收|之前的变量作为参数，并且始终是第一个参数

​	  2.过滤器也可以接收额外参数、多个过滤器也可以串联

​      3.并没有改变原本的数据, 是产生新的对应的数据

示例：

```html
<div id="root">
	<h2>显示格式化后的时间</h2>
	<!-- 计算属性实现 -->
	<h3>现在是：{{fmtTime}}</h3>
	<!-- methods实现 -->
	<h3>现在是：{{getFmtTime()}}</h3>
	<!-- 过滤器实现 -->
	<h3>现在是：{{time | timeFormater}}</h3>
	<!-- 过滤器实现（传参） -->
	<h3>现在是：{{time | timeFormater('YYYY_MM_DD') | mySlice}}</h3>
	<h3 :x="msg | mySlice">尚硅谷</h3>
</div>

<div id="root2">
	<h2>{{msg | mySlice}}</h2>
</div>
</body>

<script type="text/javascript">
Vue.config.productionTip = false
//全局过滤器
Vue.filter('mySlice',function(value){
	return value.slice(0,4)
})

new Vue({
	el:'#root',
	data:{
		time:1621561377603, //时间戳
		msg:'你好，尚硅谷'
	},
	computed: {
		fmtTime(){
			return dayjs(this.time).format('YYYY年MM月DD日 HH:mm:ss')
		}
	},
	methods: {
		getFmtTime(){
			return dayjs(this.time).format('YYYY年MM月DD日 HH:mm:ss')
		}
	},
	//局部过滤器
	filters:{
		timeFormater(value,str='YYYY年MM月DD日 HH:mm:ss'){
			// console.log('@',value)
			return dayjs(value).format(str)
		}
	}
})
```

### 内置指令

#### 前面的指令

​    v-bind  : 单向绑定解析表达式, 可简写为 :xxx

​    v-model : 双向数据绑定

​    v-for  : 遍历数组/对象/字符串

​    v-on   : 绑定事件监听, 可简写为@

​    v-if     : 条件渲染（动态控制节点是否存存在）

​    v-else  : 条件渲染（动态控制节点是否存存在）

​    v-show  : 条件渲染 (动态控制节点是否展示)

#### v-text指令

​    1.作用：向其所在的节点中渲染文本内容。

​    2.与插值语法的区别：v-text会替换掉节点中的内容，{{xx}}则不会。

#### v-html指令

​    1.作用：向指定节点中渲染包含html结构的内容。

​    2.与插值语法的区别：

​          (1).v-html会替换掉节点中所有的内容，{{xx}}则不会。

​          (2).v-html可以识别html结构。

​    3.严重注意：v-html有安全性问题！！！！

​          (1).在网站上动态渲染任意HTML是非常危险的，容易导致XSS攻击。

​          (2).一定要在可信的内容上使用v-html，永不要用在用户提交的内容上！

#### v-cloak指令（没有值）

​    1.本质是一个特殊属性，Vue实例创建完毕并接管容器后，会删掉v-cloak属性。

​    2.使用css配合v-cloak可以解决网速慢时页面展示出{{xxx}}的问题。

#### v-once指令

​      1.v-once所在节点在初次动态渲染后，就视为静态内容了。

​      2.以后数据的改变不会引起v-once所在结构的更新，可以用于优化性能。

#### v-pre指令

​    1.跳过其所在节点的编译过程。

​    2.可利用它跳过：没有使用指令语法、没有使用插值语法的节点，会加快编译。

#### 自定义指令

​    一、定义语法：

​          (1).局部指令：

`new Vue({  directives:{指令名:配置对象} })`  或      `new Vue({directives{指令名:回调函数}})`

​           (2).全局指令：

Vue.directive(指令名,配置对象) 或  Vue.directive(指令名,回调函数)

​		   (3)指令有element和binding两个参数，element是指令所在的元素，binding是指令所绑定的数据（vm中）

​    二、配置对象中常用的3个回调：

​          (1).bind：指令与元素成功绑定时调用。

​          (2).inserted：指令所在元素被插入页面时调用。

​          (3).update：指令所在模板结构被重新解析时调用。

​    三、备注：

​          1.指令定义时不加v-，但使用时要加v-；

​          2.指令名如果是多个单词，要使用kebab-case命名方式，不要用camelCase命名。

示例：

```html
<div id="root">
		<h2>{{name}}</h2>
		<h2>当前的n值是：<span v-text="n"></span> </h2>
		<!-- <h2>放大10倍后的n值是：<span v-big-number="n"></span> </h2> -->
		<h2>放大10倍后的n值是：<span v-big="n"></span> </h2>
		<button @click="n++">点我n+1</button>
		<hr/>
		<input type="text" v-fbind:value="n">
	</div>
</body>

<script type="text/javascript">
	Vue.config.productionTip = false

	//定义全局指令
	/* Vue.directive('fbind',{
		//指令与元素成功绑定时（一上来）
		bind(element,binding){
			element.value = binding.value
		},
		//指令所在元素被插入页面时
		inserted(element,binding){
			element.focus()
		},
		//指令所在的模板被重新解析时
		update(element,binding){
			element.value = binding.value
		}
	}) */

	new Vue({
		el:'#root',
		data:{
			name:'尚硅谷',
			n:1
		},
		directives:{
			//big函数何时会被调用？1.指令与元素成功绑定时（一上来）。2.指令所在的模板被重新解析时。
			/* 'big-number'(element,binding){
				// console.log('big')
				element.innerText = binding.value * 10
			}, */
			big(element,binding){
				console.log('big',this) //注意此处的this是window
				// console.log('big')
				element.innerText = binding.value * 10
			},
			fbind:{
				//指令与元素成功绑定时（一上来）
				bind(element,binding){
					element.value = binding.value
				},
				//指令所在元素被插入页面时
				inserted(element,binding){
					element.focus()
				},
				//指令所在的模板被重新解析时
				update(element,binding){
					element.value = binding.value
				}
			}
		}
	})
	
</script>
```

### 生命周期

![img](img\12190c2a-673b-4e5d-8298-d9e9efda5592-651814.jpg)

常用的生命周期钩子：

​    1.mounted: 发送ajax请求、启动定时器、绑定自定义事件、订阅消息等【初始化操作】。

​    2.beforeDestroy: 清除定时器、解绑自定义事件、取消订阅消息等【收尾工作】。***已经改名为beforeUnmount！！！***



关于销毁Vue实例

​    1.销毁后借助Vue开发者工具看不到任何信息。

​    2.销毁后自定义事件会失效，但原生DOM事件依然有效。

​    3.一般不会在beforeDestroy操作数据，因为即便操作数据，也不会再触发更新流程了。

示例：

```javascript
	<script type="text/javascript">
		Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

		 new Vue({
			el:'#root',
			data:{
				opacity:1
			},
			methods: {
				stop(){
					this.$destroy()
				}
			},
			//Vue完成模板的解析并把初始的真实DOM元素放入页面后（挂载完毕）调用mounted
			mounted(){
				console.log('mounted',this)
				this.timer = setInterval(() => {
					console.log('setInterval')
					this.opacity -= 0.01
					if(this.opacity <= 0) this.opacity = 1
				},16)
			},
			beforeDestroy() {
				clearInterval(this.timer)
				console.log('vm即将驾鹤西游了')
			},
		})

	</script>
```

### 非单文件组件

Vue中使用组件的三大步骤：

​    一、定义组件(创建组件)

​    二、注册组件

​    三、使用组件(写组件标签)



一、如何定义一个组件？

​      使用Vue.extend(options)创建，其中options和new Vue(options)时传入的那个options几乎一样，但也有点区别；

​      区别如下：

​          1.el不要写，为什么？ ——— 最终所有的组件都要经过一个vm的管理，由vm中的el决定服务哪个容器。

​          2.data必须写成函数，为什么？ ———— 避免组件被复用时，数据存在引用关系。

​      备注：使用template可以配置组件结构。



二、如何注册组件？

​        1.局部注册：靠new Vue的时候传入components选项

​        2.全局注册：靠Vue.component('组件名',组件)



三、编写组件标签：

​        <school></school>

示例：

```html
		<div id="root">
			<hello></hello>
			<hr>
			<h1>{{msg}}</h1>
			<hr>
			<!-- 第三步：编写组件标签 -->
			<school></school>
			<hr>
			<!-- 第三步：编写组件标签 -->
			<student></student>
		</div>

		<div id="root2">
			<hello></hello>
		</div>
	</body>

	<script type="text/javascript">
		Vue.config.productionTip = false

		//第一步：创建school组件
		const school = Vue.extend({
			template:`
				<div class="demo">
					<h2>学校名称：{{schoolName}}</h2>
					<h2>学校地址：{{address}}</h2>
					<button @click="showName">点我提示学校名</button>	
				</div>
			`,
			// el:'#root', //组件定义时，一定不要写el配置项，因为最终所有的组件都要被一个vm管理，由vm决定服务于哪个容器。
			data(){
				return {
					schoolName:'尚硅谷',
					address:'北京昌平'
				}
			},
			methods: {
				showName(){
					alert(this.schoolName)
				}
			},
		})

		//第一步：创建student组件
		const student = Vue.extend({
			template:`
				<div>
					<h2>学生姓名：{{studentName}}</h2>
					<h2>学生年龄：{{age}}</h2>
				</div>
			`,
			data(){
				return {
					studentName:'张三',
					age:18
				}
			}
		})
		
		//第一步：创建hello组件
		const hello = Vue.extend({
			template:`
				<div>	
					<h2>你好啊！{{name}}</h2>
				</div>
			`,
			data(){
				return {
					name:'Tom'
				}
			}
		})
		
		//第二步：全局注册组件
		Vue.component('hello',hello)

		//创建vm
		new Vue({
			el:'#root',
			data:{
				msg:'你好啊！'
			},
			//第二步：注册组件（局部注册）
			components:{
				school,
				student
			}
		})

		new Vue({
			el:'#root2',
		})
	</script>
```

#### 几个注意点

​    1.关于组件名:

​          一个单词组成：

​                第一种写法(首字母小写)：school

​                第二种写法(首字母大写)：School

​          多个单词组成：

​                第一种写法(kebab-case命名)：my-school

​                第二种写法(CamelCase命名)：MySchool (需要Vue脚手架支持)

​          备注：

​              (1).组件名尽可能回避HTML中已有的元素名称，例如：h2、H2都不行。

​              (2).可以使用name配置项指定组件在开发者工具中呈现的名字。



​    2.关于组件标签:

​          第一种写法：<school></school>

​          第二种写法：<school/>

​          备注：不用使用脚手架时，<school/>会导致后续组件不能渲染。



​    3.一个简写方式：

​          const school = Vue.extend(options) 可简写为：const school = options

#### 组件嵌套

```html
<body>
<!-- 准备好一个容器-->
<div id="root">
	
</div>
</body>

<script type="text/javascript">
Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

//定义student组件
const student = Vue.extend({
	name:'student',
	template:`
		<div>
			<h2>学生姓名：{{name}}</h2>	
			<h2>学生年龄：{{age}}</h2>	
		</div>
	`,
	data(){
		return {
			name:'尚硅谷',
			age:18
		}
	}
})

//定义school组件
const school = Vue.extend({
	name:'school',
	template:`
		<div>
			<h2>学校名称：{{name}}</h2>	
			<h2>学校地址：{{address}}</h2>	
			<student></student>
		</div>
	`,
	data(){
		return {
			name:'尚硅谷',
			address:'北京'
		}
	},
	//注册组件（局部）
	components:{
		student
	}
})

//定义hello组件
const hello = Vue.extend({
	template:`<h1>{{msg}}</h1>`,
	data(){
		return {
			msg:'欢迎来到尚硅谷学习！'
		}
	}
})

//定义app组件
const app = Vue.extend({
	template:`
		<div>	
			<hello></hello>
			<school></school>
		</div>
	`,
	components:{
		school,
		hello
	}
})

//创建vm
new Vue({
	template:'<app></app>',
	el:'#root',
	//注册组件（局部）
	components:{app}
})
</script>
```

#### VueComponent的原理

  1.school组件本质是一个名为VueComponent的构造函数，且不是程序员定义的，是Vue.extend生成的。

  2.我们只需要写<school/>或<school></school>，Vue解析时会帮我们创建school组件的实例对象，

​    即Vue帮我们执行的：new VueComponent(options)。

  3.特别注意：每次调用Vue.extend，返回的都是一个全新的VueComponent！！！！

  4.关于this指向：

​      (1).组件配置中：

​            data函数、methods中的函数、watch中的函数、computed中的函数 它们的this均是【VueComponent实例对象】。

​      (2).new Vue(options)配置中：

​            data函数、methods中的函数、watch中的函数、computed中的函数 它们的this均是【Vue实例对象】。

  5.VueComponent的实例对象，以后简称vc（也可称之为：组件实例对象）。Vue的实例对象，以后简称vm。

  6.一个重要的内置关系：VueComponent.prototype.__proto__ === Vue.prototype

  7.为什么要有这个关系：让组件实例对象（vc）可以访问到 Vue原型上的属性、方法。

### 单文件组件

![image-20211030191519456](E:\All Files\Git\web-frontend-resources\vue\img\image-20211030191519456.png)

1. 在main.js里面定义vm，在App.vue里定义根组件，并绑定其他子组件，在index.html中映入main.js
2. 组件使用`export default {...}`来导出，用`import School from './School.vue'`来导入
3. 在组件文件里面有三个标签，分别为template，script和style，可以在安装Vetur插件后使用Vu来快速创建这三个标签
4. 注意template标签里必须有根元素（如<div>）

## vue-cli

### app.js

```javascript
new Vue({
  render: h => h(App),
}).$mount('#app')
```

render是将组件渲染到模版文件上去，h是render函数传入的一个参数，实际作用为创建元素createElement函数，此函数接收两个参数`q=> q('h1','你好啊')`，第一个为标签，第二个为值。

## 注意

1. js表达式 和 js代码(语句)的区别：

   (1).表达式：<mark>**一个表达式会产生一个值**</mark>，可以放在任何一个需要值的地方：

   ​                  (1). a

   ​                  (2). a+b

   ​                  (3). demo(1)

   ​                  (4). x === y ? 'a' : 'b'

   (2).js代码(语句)

   ​                  (1). if(){}

   ​                  (2). for(){}

   
   
2. 所被Vue管理的函数，最好写成普通函数，这样this的指向才是vm 或 组件实例对象。

3. 所有不被Vue所管理的函数（定时器的回调函数、ajax的回调函数等、Promise的回调函数），最好写成箭头函数，这样this的指向才是vm 或 组件实例对象。

## 其他

### API请求

1. 需要配置代理服务器，否则为跨域

```javascript
    axios.get(`http://localhost:8080/pingTime`).then(
    response => {
        console.log('请求成功了')
        //请求成功后更新List的数据
        console.log('数据为：'+response.data['serverTime'])
    },
    error => {
        //请求后更新List的数据
        console.log('请求失败'+error)
    }
)
```



### 使用PWA

```
// 加入pwa功能
vue add pwa

// 若pwa无法使用则更新
vue update
// 全局安装服务器
npm install –g browser-sync

// cd到build后的dist文件，在浏览器加入应用即可
browser-sync
```



### 使用ElementPlus

```js
// npm install element-plus
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import './registerServiceWorker'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'


const app = createApp(App)
app.use(ElementPlus)
app.mount('#app')
```



### 使用VuePress

```bash
// 运行测试服务器
npm run docs:dev
// 构建静态文件
npm run docs:build
```



