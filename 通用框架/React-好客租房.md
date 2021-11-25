[TOC]

## 一. 项目初始化

1. 主要业务点

   ​	首页、登录注册、地图找房、列表查房、详情显示、个人收藏、发布房源。

2. 主要技术栈

   ​	react、react-dom、react-router-dom(前端核心框架)

   ​	create-react-app(脚手架)

   ​	axios(网络请求)

   ​	anti-mobile(UI组件库)

   ​	react-vitualized、formik+yup、react-spring(其他组件库)

3. 接口部署

   ​	新建DB-hkzf并导入，进入 数据库相关文件\hkzf_v1 新建终端并npm start来运行后台。后台数据库配置在config/mysql.js中。后台代码不是项目重点，不做讲解。测试地址默认为本地的8080端口，可以查看swagger部署

4. 初始化前端项目

   ​	anti-mobile是一个第三方的组件库，我们将会在项目中使用它：https://mobile.ant.design/guide/quick-start，注意使用V2的版本，文档看这里：https://antd-mobile-v2.surge.sh/docs/react/introduce-cn#%E7%89%88%E6%9C%AC。初始化完成后检查入口文件index.js和根组件App.js和默认项目结构，在src下新建assets, component, pages, utils。

   ```
   npx create-react-app hkzf-mobile
   npm start
   npm install --save antd-mobile-v2
   app.js导入组件库组件：import { Button } from 'antd-mobile-v2'，样式组件会自动导入，不用管。
   ```

5. 配置基础路由，先用旧路由版本。新的还是有改动，容易报错。

   ```
   yarn add react-router-dom@v4.3.1
   
   //App.js
   import {BrowserRouter as Router, Route, Link} from 'react-router-dom'...react路由Route+Link就行
   //Home/index.js created
   //CityList/index.js created 
   
   npx kill-port 3000
   npm start
   ```

6. 简单调整下index.css->全局样式（body高度100%，使用盒模型）和字体就行了。

​		注意导入顺序，antd的样式Import应该在index.css之后。	



## 二.项目布局

 1. 嵌套路由

    由一个包含四个标签的导航和四个子路由组成（里面的组件）

    ```
    //App.js
    <Router>
    	<div>
    		<Route path="/home" component={Home} />
    	<div/>
    </Router>
    //Home.js 嵌套中子路由的前置必须和父组件一致。原因是子组件必须基于父组件展示。
    //既子路由匹配时父路由必须匹配，基于的是react的模糊匹配
    const Home = () => (
    	<div>
    		home page
    		<Route path="/home/news" component={News} />
    	<div/>
    )
    ```

	2. TabBar组件使用

    ```
    //拷贝antd组件库中的核心代码到Home组件中 注意用V2版本（以后不提了哈）
    //修改样式，包括标题、颜色、大小和布局
    css：https://www.runoob.com/css3/css3-tutorial.html
    ```

    根据文档设置属性为不渲染内容，在点击事件中实现路由切换，配置路由信息和组件内容（本目1），通过自定义的一个selectedTab值控制点击时高亮（提供的selected-是否选中属性为item.path == selectedTab）。

    ```
    this.props.history.push('/home/list')
    ```

    读代码和写代码都要从render入手，所谓自上而下。

	3. 如何将默认路由匹配到组件，通过render属性接受一个函数可以指定直接的路由渲染内容，Redirect用于实现跳转到新的路由地址

    ```
    <Route path="/" exact render={() => <Redirect to="/home" />} />
    ```

    

## 三.首页

 1. 使用轮播图（Carousel）组件

    看前头提供的antd-mobile-v2文档和属性，在使用轮播图中，可能动态加载数据的异步会导致时自动轮播和高度出现问题。我们在state中加入表示轮播图加载完成的变量，在数据加载完成时将值设为ture，仅有当为true时才渲染轮播图组件（条件渲染）。

    数据渲染使用axios进行请求即可，注意阻塞和生命周期位置:

    ```
    /*1 安装 axios：yarn add axios 。
      2 在 Index 组件中导入 axios 。
      3 在 state 中添加轮播图数据：swipers 。
      4 新建一个方法 getSwipers 用来获取轮播图数据，并更新 swipers 状态。
      5 在 componentDidMount 钩子函数中调用该方法。
      6 使用获取到的数据渲染轮播图。
    */
    
    export default class Index extends React.Component {
      state = {
        swipers: []
        isSwiperLoaded: false
      }
      async getSwipers() {
        const res = await axios.get('http://localhost:8080/home/swiper')
        this.setState({
          swipers: res.data.body
        })
      }
    
      componentDidMount() {
        this.getSwipers()
      }
      
      render() {
        return (
          <div className="index">
            {this.state.isSwiperLoaded?
            (<Carousel autoplay infinite autoplayInterval={5000}>
              {this.renderSwipers()}
            </Carousel>):('')}
           ...
    }
    最后提一个小问题：需要在X方向上禁用滑动，否则会出现滑动冲突，运用到哦touch-action属性：
    https://www.cnblogs.com/phoebeyue/p/9583967.html
    ```

	2. 使用Flex组件

    同目1：https://antd-mobile-v2.surge.sh/components/flex-cn/

    ```
    import { Carousel, Flex } from 'antd-mobile'
    // 导入axios 菜单图片 导入样式文件...
    // 导航菜单数据
    const navs = [
      {
        id: 1,
        img: Nav1,
        title: '整租',
        path: '/home/list'
      },
      {
        id: 2,...
    ]
    
    export default class Index extends React.Component {
      // 渲染轮播图...
      // 渲染导航菜单 返回Item数组
      renderNavs() {
        return navs.map(item => (
          <Flex.Item
            key={item.id}
            onClick={() => this.props.history.push(item.path)}
          >
            <img src={item.img} alt="" />
            <h2>{item.title}</h2>
          </Flex.Item>
        ))
      }
    
      render() {
        return (...
            {/* 导航菜单 */}
            <Flex className="nav">{this.renderNavs()}</Flex>
        ...
    //你完全可以在Flex下面加Flex.Item子标签。用上面这种属性map的方式可以更好地利用navs数据数组复用view
    ```

    问题是如何实现点击flex之后下面tabbar随之高亮: 利用在路由切换时componentDidUpdate会被调用，判断路由是否发生变化作为基线：

    ```
    export default class Home extends React.Component {
      state = {
        selectedTab: this.props.location.pathname
      }
      // 组件接收到新的props（此处，实际上是路由信息）就会触发该钩子函数
      componentDidUpdate(prevProps) {
        // prevProps 上一次的props，此处也就是上一次的路由信息
        // this.props 当前最新的props，此处也就是最新的路由信息
        // 注意：在该钩子函数中更新状态时，一定要在 条件判断 中进行，否则会造成递归更新的问题
        if (prevProps.location.pathname !== this.props.location.pathname) {
          // 此时，就说明路由发生切换了
          this.setState({
            selectedTab: this.props.location.pathname
          })
        }
      }
    ```

	3. 在脚手架中使用sass:

    什么是sass:  强化 CSS 的辅助工具，它在 CSS 语法的基础上增加了变量、嵌套、混合 、导入等高级功能有助于更好地组织管理样式文件。https://www.sass.hk/docs/

    如何使用:https://create-react-app.dev/docs/adding-a-sass-stylesheet/

    ```
    npm install node-sass
    create .scss/sass and import
    ```

    

## 四. 租房功能和城市选择

1. Grid组件和WingBlank（两翼留白）组件

2. H5地图API调用方法

   https://developer.mozilla.org/zh-CN/docs/Web/API/Geolocation_API

   ```
   //获取到的地址和GPS，IP，WIFI/Bluetooth的mac地址，GSM/CDMS的ID有关系。需要请求权限。
   navigator.geolocation.getCurrentPosition(position => {...})
   ```

3. 百度地图API调用（实际应用中的方式，不同公司往往不同）

   https://lbsyun.baidu.com/index.php?title=jspopularGL

   开发指南-定位。控制台创建应用，白名单配置为*。之后使用ak就可以了。

   因为我们做的是一个单页应用，所以在index.html中引用js就行了，注意地图实例对象需要在div container渲染完之后写，即写在componentDidMount。

   ```
   //react脚手架中全局对象需要使用window来访问，否则会造成ESLint报错
   const map = new window.BMap.Map("container")
   const point = new window.BMap.Point(116.404, 39.915)
   map.centerAndZoom(point, 15)
   ```

4. 顶部导航栏-NavBar组件（还是antd里面的）

   

   