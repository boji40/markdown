#### 1、element ui -- 带自定义验证的表单 this.$refs[formName].validate()里面的内容不执行

解决：自定义验证的所有if都得闭环，也就是所有的 if 都得有else

```
let validatorContractId = (rule, value, callback) => {  
	setTimeout(() => {    
		if (value !== "") {      
			if (!/^[a-zA-Z0-9]{2,25}$/.test(value)) {        
				return callback(new Error('合同号只能为字母和数字'))      
			}else{        
				callback()      
			}    
		} else {      
			callback()    
		}  
	}, 500)
}
```

#### 2、element ui 带建议的input输入框，实现模糊查询，将查询到的字段填充到表格中，因为模糊查询的表格单元格为动态生成，就需要拿到 index 参数，官方文档中实例为：

```
<el-autocomplete  
	v-model="state"  
	:fetch-suggestions="querySearchAsync"  
	placeholder="请输入内容"  
	@select="handleSelect">
</el-autocomplete>
```

经过实验，这样的方法行不通，解决方法如下：

```
<el-autocomplete  
	v-model="state"  
	:fetch-suggestions="querySearchAsync"  
	placeholder="请输入内容"  
	@select="((item) => {handleSelect(item, index)})">
</el-autocomplete>
```

需求中表格中嵌入了input输入框，故需

```
@select="((item) => {handleSelectPro(item, scope.$index)})"
```

#### 3、ES6的Object.assign()基本用法、注意点

##### 基本用法

Object.assign()方法用于将所有可枚举属性的值从一个或多个源对象source复制到目标对象，他将返回目标对象target。

```
const target = {a:1,b:2}
const source = {b:4,c:5}
const returnedTarget = Object.assign(target,source) 

target //{a:1,b:2,b:4,c:5}
returnedTarget //{a:1,b:2,b:4,c:5}
```

Object.assign 方法的第一个参数是目标对象，后面的参数都是源对象。

注意：如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。

```
const target = {a:1,b:1}
const source1 = {b:2,c:2}
const source2 = {c:3}

Object.assign(target,source1,source2)
target  //{a:1,b:2,c:3}
```

如果只有一个参数，Object.assign会直接返回该参数，如果该参数不是对象，则会先转成对象，然后返回

```
typeof Object.assign(2) //"object"
```

由于undefined和null无法转成对象，如果他们作为首参数，就会报错。

```
Object.assign(undefined) //报错
Object.assign(null) //报错
```

如果非对象参数出现在源对象的位置（即非首参数），那么处理规则有所不同。首先，这些参数都会转成对象，就会跳过。这意味着，如果undefined和null不在首参数，就不会报错。

``` 
let obj = {a:1}
Object.assign(obj,undefined) === obj  //true
Object.assign(obj,null) === obj  //true
```

其他类型的值（即数值、字符串和布尔值）不在首参数，也不会报错。但是，除了字符串会以数组形式，拷贝入目标对象，其他值都不会产生效果。

```
const v1 = 'abc'
const v2 = true
const v3 = 10

const obj = Object.assign({},v1,v2,v3)
obj   //{"0":"a","1":"b","2":"c"}
```

上面代码中，v1，v2，v3分别是字符串、布尔值和数值，结果只有字符串合入目标对象（以字符数组的形式），数值和布尔值都会被忽略。这是因为只有字符串的包装对象，会产生可枚举属性

```
Object.assign(true) //{[[PrimitiveValue]]:true}
Object.assign(10) //{[[PrimitiveValue]]:10}
Object.assign('abc') //{"0":"a","1":"b","2":"c",length:3,[[PrimitiveValue]]:'abc'}
```

上面代码中，布尔值、数值、字符串分别转成对应的包装对象，可以看到他们的原始值都在包装对象的内部属性[[PrimitiveValue]]上面，这个属性是不会被Object.assign拷贝的。只有字符串的包装对象，会产生可枚举的实义属性，那些属性则会被拷贝。

Object.assign拷贝的属性是有限制的，只拷贝源对象的自身属性（不拷贝继承属性），也不拷贝不可枚举的属性(enumerable:false)

```
Object.assign({b: 'c'},
  Object.defineProperty({}, 'invisible', {
    enumerable: false,
    value: 'hello'
  })
)
// { b: 'c' }
```

上面代码中，`Object.assign`要拷贝的对象只有一个不可枚举属性`invisible`，这个属性并没有被拷贝进去。

属性名为 `Symbol` 值的属性，也会被`Object.assign`拷贝。

```
Object.assign({ a: 'b' }, { [Symbol('c')]: 'd' })
// { a: 'b', Symbol(c): 'd' }
```

##### 注意点

（1）浅拷贝

Object.assign方法实行的是浅拷贝，而不是深拷贝。也就是说，如果源对象某个属性的值是对象，那么目标对象拷贝的到的是这个对象的引用。

```
const obj1 = {a: {b: 1}}
const obj2 = Object.assign({}, obj1)

obj1.a.b = 2
obj2.a.b // 2
```

 上面代码中，源对象`obj1`的`a`属性的值是一个对象，`Object.assign`拷贝得到的是这个对象的引用。这个对象的任何变化，都会反映到目标对象上面。 

（2）同名属性的替换

对于对象的拷贝，一旦遇到同名属性，Object.assign的处理方法是替换，而不是添加。

（3）数组的处理

Object.assign可以用来处理数组，但是会把数组视为对象

```
Object.assign([1, 2, 3], [4, 5])
// [4, 5, 3]
```

 上面代码中，`Object.assign`把数组视为属性名为 `0、1、2` 的对象，因此源数组的 `0` 号属性`4`覆盖了目标数组的 `0` 号属性`1`。 

#### 4、Object.defineProperty()方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象

注：应当直接在Object构造器对象上调用此方法，而不是在任意一个Object类型的实例上调用。

```
const object1 ={}
Object.defineProperty(object1,'property1',{
	value:42,
	writable:false
})
object1.property1 = 77
console.log(object1.property1)  //42
```

#### 5、可枚举属性

- 可枚举属性是指那些内部“可枚举”标志设置为true的属性。对于通过直接的赋值和属性初始化的属性，枚举标志默认为true。但是对于通过Object.defineProperty等定义的属性，该标识值默认为false
- js中基本包装类型的原型属性是不可枚举的，如Object，Array，Number等。
- 可枚举的属性可以通过for...in循环进行遍历（除非该属性名是一个Symbol），或者通过Object.keys()方法返回一个可枚举属性的数组。

枚举属性主要会影响几个方法：

- for...in   只遍历对象自身的和继承的可枚举的属性
- Object.keys()   返回对象自身的所有可枚举属性的键名
- JSON.stringify   用于将JavaScript值转换为JSON字符串
- Object.assign    会忽略enumerable为false的属性，只拷贝对象自身的可枚举的属性

#### 6、vue  父组件调用子组件的方法

父组件：

```
<el-dialog>
	<split-bill ref="SplitBill"></split-bill>
</el-dialog>

export default {
	...
	methods:{
		parent(){
			this.$refs.SplitBill.child()  //调用子组件方法
		}
	}
}
```

子组件：

```
methods:{
	child(){
		....
	}
}
```

#### 7、vue + axios 在IE页面下不刷新数据问题

​	今天在对表格数据进行编辑操作，发现编辑后的数据页面并不刷新显示出来，打开network监控请求，发现编辑后发送的请求是直接拿到上一次（get）请求的结果（老数据），这是由于IE缓存机制的问题。

​	ie浏览器会对相同请求的ajax,进行缓存，当你编辑数据后，在调用同样的接口，ie不会刷新，只会把第一次请求该接口的数据拿出来，所以看到的页面永远都是第一次请求的ajax返回的数据。

​	StackOverflow上有很多建议都是在请求后加上一个时间戳，保证请求的唯一性。但是由于本项目已经完成一大部分后需求才要兼容ie浏览器，一个一个改未免有一点太麻烦了。。。所以我采用了如下方法，把axios的缓存机制关掉，ie就可以正常刷新了。

```
axios.defaults.headers.get.Pragma = 'no-cache'
axios.defaults.headers.get['Cache-Control'] = 'no-cache, no-store'
```

#### 8、el-input框无法输入，获取不到焦点

今天写了一个数据展示的页面，用到复合型输入框，写完后发现input狂获取不到焦点，无法输入值，input框后面的button也无法点击

解决：input框外围层级使用了z-index层叠样式，导致输入框无法获取焦点

#### 9、element-ui的表单验证，填完数据后校验不消失

解决：el-form上一定要绑定数据，校验的prop属性和输入框绑定v-model的值一定要一样！！！今天代码看得眼花缭乱。。。

#### 10、小笔记

```
// 计算出现次数大于3次的元素返回成数组
	var arr = [1,2,3,1,2,2,2,4]
	// reduce:以一个参照物作为对比，将每个元素出现的次数进行累加
	// 将出现次数大于3次的item返回
	arr.filter(item => (arr.reduce(( total, now) => total+= item === now ? 1 : 0 , 0)) >= 3)
```

#### 11、使用命令行安装完成vue/cli后，使用vue无法创建项目，vue报错：无法加载文件 C:\Users\XXX\AppData\Roaming\npm\vue.ps1，因为在此系统上禁止运行脚本

解决方法：以管理员身份运行PowerShell，执行get-ExecutionPolicy，回复Restricted，表示状态是禁止的；然后执行set-ExecutionPolicy RemoteSigned，选择Y，重启就可以了。

#### 12、vue项目将token存在（vuex）store和sessionstorage中

token概念：token是客户端频繁向服务器请求数据，服务端频繁的去数据库查询用户名和密码并进行对比，判断用户名和密码正确与否，并作出相应提示。token是在服务端产生的一串字符串，以作客户端进行请求的一个令牌。如果前端使用用户名/密码向服务端请求认证，服务端认证成功，那么在服务端会返回token给前端。前端可以在每次请求的时候带上token证明自己的合法性，如果这个token在服务端持久化（比如存入数据库），那他就是一个永久的身份令牌（除非设置了有效期）

token优点：token完全由应用管理，所以它可以避开同源策略；token可以避免CSRF攻击；token可以是无状态的，可以在多个服务间共享；减轻服务器压力，减少频繁的查询数据库，使服务器更加健壮

大致思路：

1. 第一次登陆的时候，前端调后端登录接口，发送用户名和密码
2. 后端收到请求，验证用户名和密码，验证成功，给前端返回一个token
3. 前端拿到token，将token存到vuex和session中，并跳转路由页面
4. 前端每次跳转路由，就判断sessionstorage中有无token，没有就跳转登陆页面，有则跳转对应路由页面
5. 每次调用后端接口，都要在请求头中加token
6. 后端判断请求头中有无token，有token就拿到token并验证，验证成功就返回数据，验证失败就返回相应的状态码（401），没有token也返回相应状态码
7. 前端拿到状态码为401，就清除token信息并跳转到登陆页面

实现代码：

创建storage

```
import Vue from "vue";
import axios from "axios";
import router from "../router";
import store from "../store/index"


  //请求失败后的统一处理
  // const errorHandle = (status,other) => {
  //   //状态码判断
  //   switch (status) {
  //     case -1:
  //
  //   }
  // }

  //创建axios实例
  const instance = axios.create({
    baseURL: "/",
    timeout: 8000
  });
  //设置post请求头
  instance.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
  //设置ie浏览器缓存机制
  instance.defaults.headers.get.Pragma = "no-cache";
  instance.defaults.headers.get["Cache-Control"] = "no-cache, no-store";

  //axios request拦截器
  instance.interceptors.request.use(
      config => {
      //  每次发送请求前检测vuex是否存有token，存不存在都要放在请求头发送给服务器
        if (store.state.token) { //存在
          config.headers.Authorization = store.state.token
        }
        return config
      },
      error => {
        console.log('在request拦截器显示错误：',error.response)
        return Promise.reject(error)
      }
  )
  //axios response拦截器
  instance.interceptors.response.use(
      response => {
      //  在status正确的情况下，code不正确则返回对应的错误信息（后台自定义为200是正确，并且将错误信息写在message），正确则返回响应
        return response.data.code === '200' ? response : Promise.reject(response.data.message)
      },
      error => {
      //  在status不正确的情况下，判断状态码并给出对应响应
        if (error.response) {
          console.log("在respone拦截器显示错误：", error.response)
          switch (error.response.status) {
            case -1:
          //    还不确定状态码，可能是token过期，清除token，跳转到登录页面
                  store.commit('del_token')

                  router.push('/')
          }
        }
        console.log(error)
        return Promise.reject(error.response.data)
      }
  )

  export default instance
```

创建store

```
import Vue from "vue";
import Vuex from "vuex";
import { _session } from '../utils/storage'

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
    token:""
  },
  mutations: {
    //修改token，并将token存入localSrorage
    set_token(state,token) {
      state.token = token
      _session.set('token',token)
      console.log('session保存token成功！');
    },
    //删除token
    del_token(state) {
      state.token = ''
      _session.remove('token')
    }
  },
  actions: {},
  modules: {}
});
```

配置代理

```
module.exports = {
    lintOnSave: false,
    devServer: {
        open: true,
        port: 8080,
        https: false,
        proxy:{
            //配置跨域
            '/':{
            	//在本地会创建一个虚拟服务器，然后发送请求的数据，并同时接收请求的数据，这样服务端进行数据的交互时就不会出现跨域的问题
                target:'http://114.55.164.241/', // background nginx proxy
                ws:true,
                changeOrigin: true, //允许跨域
                pathRewrite:{  //路径重写
                    '^/': '' // 请求的时候使用这个api替换target中的请求地址
                }
            }
        }
    }
}
```

封装路由，并设置路由守卫

```
//设置路由守卫，在进页面之前，判断有token，才进入页面，否则返回登陆页面
if (_session.get('token')) {
  store.commit('set_token',_session.get('token'))
}
router.beforeEach((to,from,next) => {
//  判断要去的路由有没有requiresAuth
  if (to.matched.some( r => r.meta.requireAuth)) {   //需要token
    if (store.state.token) {  //有token
      next();
    } else {
      next({path:'/login'})
    }
  } else {  //不需要token
    next()
  }
})
```

在main.js中引入router和store

```
import router from "./router";
import store from "./store";

new Vue({
  router,
  store,
  render: h => h(App)
}).$mount("#app");
```

#### 13、vue的MVVM理解

#### ![image-20200818102522951](C:\Users\35747\AppData\Roaming\Typora\typora-user-images\image-20200818102522951.png)

View层：视图层，在前端中通常指DOM层，主要的作用是给用户展示各种信息   -->html

Model层：数据层，数据可能是我们固定的死数据，更多的是来自服务器，从网络上请求下来的数据  -->data

ViewModel层：视图模型层，视图模型层是View和Model沟通的桥梁。一方面它实现了Data Binding，也就是数据绑定，将Model的改变实时的反应到View中；另一方面它实现了DOMListener，也就是DOM监听，当DOM发生一些事件（点击，滚动，hover）时可以监听到，并在需要的情况下改变对应的Data     --> new Vue({...})

#### 14、安装mongodb数据库

1. 下载安装（记住安装路径）
2. 配置环境变量，在系统变量path中添加mongodb安装路径的bin目录![image-20200819181639413](C:\Users\35747\AppData\Roaming\Typora\typora-user-images\image-20200819181639413.png)

![image-20200819181733509](C:\Users\35747\AppData\Roaming\Typora\typora-user-images\image-20200819181733509.png)

![image-20200819181908425](C:\Users\35747\AppData\Roaming\Typora\typora-user-images\image-20200819181908425.png)

3.以管理员身份打开Windows PowerShell连接数据库，输入命令mongod --dbpath=D:/data/db，看到27017表示连接成功

![image-20200819182135851](C:\Users\35747\AppData\Roaming\Typora\typora-user-images\image-20200819182135851.png)



#### 15、创建项目推到github上报错：

#### remote: Permission to userA/jike.git denied to userB.
fatal: unable to access 'https://github.com/boji40/jike.git/': The requested URL returned error: 403

原因：由于该电脑使用git bash配过SSH，系统已经指向github.com的用户设置为了userB，每次push操作的时候，都将读取到userB的用户信息，类似于记住密码，这个错误就是userB没有对userA的jike进行push

解决：

1. 对userA生成SSH公钥，添加到userB的github的后台
2. 将userB添加为userA项目的contributer
3. 移除计算机中的userB

对于1、2操作起来会比较麻烦，因为一旦使用SSH，以后所有的clone、pull、push等操作都将使用SSH传输，对以往使用过https传输的项目也得更新更改传输方式

第三种方式解决：

打开电脑控制面板 -> 用户 -> 证书管理 -> 系统证书，展开git:https://github.com并删除之，然后重新push项目就可以了

![image-20200828165150524](C:\Users\35747\AppData\Roaming\Typora\typora-user-images\image-20200828165150524.png)

![image-20200828165221327](C:\Users\35747\AppData\Roaming\Typora\typora-user-images\image-20200828165221327.png)

![image-20200828165404657](C:\Users\35747\AppData\Roaming\Typora\typora-user-images\image-20200828165404657.png)

#### 16、uniapp 的 scroll-into-view无效

问题：

```
<scroll-view class="chat" scroll-y="true" scroll-with-animation="true" :scroll-into-view="scrollToView">
	<view class="chat-ls" v-for="(item,index) in msgs" :key="index" :id="'msg'+item.tip">
</scroll-view>

height: 100%;
```

scroll-into-view的值为字符串，对应相应的id，达到进到页面时显示到此id对应的位置的效果，官方文档有一句话说：使用竖向滚动时，需要给 `<scroll-view>` 一个固定高度，通过 css 设置 height。

但是！奇怪的是，明明都已经设置了高度，然而还是没有效果。

看一了篇文章，说是小程序的navBar是系统自带的，设置height：100%是能正确展示的；在新版本里面，app.json（uniapp是pages.json）里面设置了"navigationStyle":"custom"，自定义样式导航栏，在调整样式的时候，设置了padding-top，为了不与自定义的navBar冲突。

解决：可以设置height: calc(100vh - 20rpx)   /或者直接height:100vh   就可以了