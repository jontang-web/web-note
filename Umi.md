### 创建应用
##### 安装umi
```
$ yarn global add umi
$ umi -v
```
##### 通过脚手架创建应用
```
$ mkdir umi-demo
$ cd umi-demo
$ yarn create umi
//依次选择app，antd等配置
$ yarn
$ yarn start
```
##### 应用目录结构
- mock：基于express，模拟服务接口数据；
- config/config.js：umi 配置，同 .umirc.js，优先级低于 .umirc.js，默认不生成此目录（可以忽略）；
- src/assets：应用静态资源（图片，字体等）；
- src/layouts/index.js：全局布局，相当于最外层路由；
- src/pages/.umi：umi dev 时生产的临时目录，umi 重启或者 pages 下的文件修改都会重新生成这个文件夹下的文件，不要直接在这里修改代码（可以忽略）；
- src/pages/document.ejs：应用的 HTML 模板，优先级高于 umi 默认模板，默认不生成此文件，可以手动添加覆盖umi的默认模板；
- src/pages/404.js：404 页面；
- src/global.css：全局样式文件；
- src/global.js：加入 polyfill，默认不生成此文件，可以手动添加；
- src/app.js：运行时配置文件，默认不生成此文件，可以手动添加；
- .umirc.js：umi的配置文件；
- .env：环境变量配置文件；

### 路由
> umi默认采用约定式路由，即动态遍历src/pages目录结构生成路由，可在src/pages/.umi/router.js查看umi自动生成的路由结构（切记不要手动修改.umi目录下的任何内容）

> 也可以手动配置路由，在.umirc.js或者src/config/config.js中配置。

##### 全局layout
约定 src/layouts/index.js 为全局路由，返回一个 React 组件，通过 props.children 渲染子组件，是整个应用的路由入口。常在此处做路由权限，登录验证等
```
export default function(props){
    const token = localStorage.getItem('_TOKEN');
    if(props.location.pathname==='/login'){
        return props.children;
    }else{
        if(token){
            return (
                <BaseLayout>
                    { props.children }
                </BaseLayout>
            );
        }else{
            router.push('/login');
            return (<Login/>);
        }
    }
}
```
##### 基础路由
假设 pages 目录结构如下：
```
+ pages/
  + users/
    - index.js
    - list.js
  - index.js
```
那么，umi 会自动生成路由配置如下：
```
[
  { path: '/', component: './pages/index.js' },
  { path: '/users/', component: './pages/users/index.js' },
  { path: '/users/list', component: './pages/users/list.js' },
]
```
##### 动态路由
umi 里约定，带 $ 前缀的目录或文件为动态路由。如：
```
+ pages/
  + $post/
    - index.js
    - comments.js
  + users/
    $id.js
  - index.js
```
生成路由配置如下：
```
[
  { path: '/', component: './pages/index.js' },
  { path: '/users/:id', component: './pages/users/$id.js' },
  { path: '/:post/', component: './pages/$post/index.js' },
  { path: '/:post/comments', component: './pages/$post/comments.js' },
]
```
> **实战技巧：路由匹配规则同react-router@4.x，在手动配置路由时，动态路由放置在同级路由的最后面**

##### 可选动态路由
umi 里约定动态路由如果带 $ 后缀，则为可选动态路由。如以下结构：
```
+ pages/
  + users/
    - $id$.js
  - index.js
```
生成路由配置如下：
```
[
  { path: '/': component: './pages/index.js' },
  { path: '/users/:id?': component: './pages/users/$id$.js' },
]
```
##### 嵌套路由
umi 里约定目录下有 _layout.js 时会生成嵌套路由，以 _layout.js 为该目录的 layout。如以下目录结构：
```
+ pages/
  + users/
    - _layout.js
    - $id.js
    - index.js
```
生成路由配置如下：
```
[
  { path: '/users', component: './pages/users/_layout.js',
    routes: [
     { path: '/users/', component: './pages/users/index.js' },
     { path: '/users/:id', component: './pages/users/$id.js' },
   ],
  },
]
```
##### 404路由
开发模式下，umi 会添加一个默认的 404 页面来辅助开发。生产环境下，没有404路由，有需要自己添加

##### 注释扩展路由
注释扩展路由是umi为了解决约定式路由的不灵活性而采取的折中解决方案。使用时注意：
1. 注释扩展路由只在约定式路由下有效，手动配置路由时无效；
2. 注释路由符合 yaml 格式，且必须在约定路由的第一行；

如 pages/index.js 里包含：
```
/**
 * title: Index Page
 * Routes:
 *   - ./src/routes/Authority.js
 */
```
会生成路由配置：
```
[
  { path: '/', component: './index.js',
    title: 'Index Page',
    Routes: [ './src/routes/Authority.js'],
  },
]
```
这样一来，当前组件（index.js）就会作为子组件嵌套在 Authority.js 中。常用来做登录验证，路由权限等
##### 路由守卫（权限路由）
注释路由中的Routes即是权限路由，可以在配置路由时添加，也可以通过yaml注释添加
### 集成dva
umi通过umi-plugin-react来集成dva，在.umirc.js的plugins中有相关配置：
```
export default {
  plugins: [
    [
      'umi-plugin-react',
      {
        dva: true  //可手动修改使用或者不使用
      }
    ],
  ],
};
```
集成dva后会在src/app.js下添加如下dva配置：
```
export const dva = {
  config: {
    onError(e) {
      e.preventDefault();
      console.error(e.message);
    },
  },
  plugins: [               //dva插件配置
    require('dva-logger')(),
  ],
};
```

##### model全局注册
> 全局 model 存于 /src/models/ 目录，所有页面都可引用。create umi 脚手架创建项目时，已经为我们生成了src/models/目录

##### model页面注册
> 页面 model 创建在pages/ 对应的页面目录下，且只能在页面中使用，不能被其他页面所引用。在 production 时按需载入，在 development 时全量载入。如：
```
+ src
 + models
    - g.js
 + pages
    + a
      - index.js  //（page，下同）
      + models
        - m1.js
        - m2.js
    + b
      - index.js
      + models
        - m3.js
      + c
        - index.js
        + models
          - m4.js
    + d
      - index.js
      - model.js
      + models
        - xx.js
```
model关系：
- 全局model是 src/models/g.js，任何地方都可以访问；
- /a 的model是m1和m2；
- /b 的model是m3；
- /b/c 的model是m3和m4（page model会向上查找，可以访问父路由的model；不能向下查找，即父路由不能访问子路由的model）；
- /d 的model是/d/model.js，models目录下的均无效（约定 model.js 为单文件 model，解决只有一个 model 时不需要建 models 目录的问题；页面下有model.js文件时，不再查找models目录）
##### model案例
不管是全局注册的model还是页面注册的model，定义方式都是一样的。例如：
```
// src/models/system.js
import { routerRedux } from 'dva/router';
import API from './../api';
import Util from './../util';
const { handleLocalStorage } = Util;
export default {
    namespace: 'system',
    state: {
        collapsed: false
    },
    effects: {
        *login({ payload }, { call, put }) {
            const res = yield call(API.login, payload);
            if(res){
                const { token } = res.data;
                handleLocalStorage.setItem('_TOKEN', token, 3*3600*1000).setItem('username', payload.username);
                yield put(routerRedux.push('/home'));
            }
        },
        *logout(_, { call }) {
            yield call(API.logout);
            handleLocalStorage.removeItem('_TOKEN');
        }
    },
    reducers: {
        toggleMenu(state, { payload }) {
            return {
                ...state,
                collapsed: payload,
            };
        }
    },
    subscriptions: {
        setupHistory({ history }){ //方法名可以随意命名，在model注册时执行监听方法
            console.log('setupHistory');
            history.listen((location) => {
                console.log(location)   //这里可以获取当前变化的history路径以及参数，hash所有值，这样就可以在路由地址变化后做处理
            });
        }
    }
}

// src/pages/user
import { connect } from 'dva';
@connect(({system, loading}) => ({
    system, //model的namespace
    loading :loading.effects['system/login']
}))
export default class extends Component{
    ...
}
//注意：通过装饰器调用connect连接，必须是class组件
```
说明：
- namespace：model的唯一标识，在store中通过state.[namespace]保存和访问状态数据；
- effects：异步action，相当于redux-saga的effects；
- reducers：同步action，与redux中的reducer一致；
- subscriptions：订阅状态变化，model注册时执行，可在订阅方法中对当前的时间、服务器的 websocket 连接、keyboard 输入、geolocation 变化、history 路由变化等等进行监听

### Mock数据
umi 里约定 mock 文件夹下的文件或者 src/pages 目录下的 _mock 文件即 mock 文件，文件导出接口定义。一般定义在mock目录下，因为开发完成打包后，mock都会失效，即mock只在development模式下有效。如：
```
import { delay } from 'roadhog-api-doc';
//模拟接口延迟
export default delay({
    //可指定请求类型 GET POST 等
    // 支持值为 Object ，Array，Function
    'POST /api/login': (req, res) => {
        const { password, username } = req.body;
        if ((password === 'admin' && username === 'admin') || (password === 'guest' && username === 'guest')) {
            res.send({
                code: 200,
                data: {
                    token: 'kshfj=shbdf=sdfjb3251',
                    currentAuthority: username
                },
                message: 'success'
            });
            return;
        }
        res.send({
            code: 500,
            message: '用户名或者密码错误'
        });
    },
    'GET /api/users': { users: [1, 2] },
    // GET POST 可省略
    '/api/users/1': { id: 1 }
}, 1000);
```
可以采用[mock.js](http://mockjs.com/)动态生成模拟数据
```
import mockjs from 'mockjs';
export default {
  // 使用 mockjs 等三方库
  'GET /api/tags': mockjs.mock({
    'list|100': [{ name: '@city', 'value|1-100': 50, 'type|0-2': 1 }],
  }),
};
```
### 配置
#### 运行时配置
umi 约定 src 目录下的 app.js 为运行时的配置文件，与dva配置在同一个文件
##### patchRoutes
动态修改路由，参数是应用的路由配置，直接修改routes，不做任何返回
```
export function patchRoutes(routes) {
  routes[0].unshift({
    path: '/foo',
    component: require('./routes/foo').default,
  });
}
```
应用场景：
- 和 render 配合使用，请求服务端根据响应动态更新路由；
- 修改全部路由，加上某个前缀
##### render

```
export function render(oldRender) {
  setTimeout(oldRender, 1000);
}
```
应用场景：
- 渲染应用之前做权限校验，不通过则跳转到登录页
##### onRouteChange
监听路由初始创建和切换，相当于vue-router的beforeEach路由守卫钩子
```
export function onRouteChange({ location, routes, action }) {
  
}
```
##### rootContainer
用于修饰react最外层根组件或者在根组件外面再包裹一层。参数container是根组件，且必须返回一个react组件（常用React.createElement创建）
```
export function rootContainer(container) {
  const DvaContainer = require('@tmp/DvaContainer').default;
  return React.createElement(DvaContainer, null, container);
}
```
应用场景：
- dva、intl 等需要在外层有个 Provider 的场景

##### modifyRouteProps
修改传给路由组件的 props 。常用于对特定的路由参数处理，参数props是原始props，{ route } 是当前路由配置，且必须返回新的props
```
export function modifyRouteProps(props, { route }) {
  return { ...props, foo: 'bar' };
}
```

#### 基本配置
##### plugins: []
插件配置，如umi-plugin-react。支持npm 依赖、相对路径或绝对路径。相对路径是相对项目根目录，绝对路径采用${__dirname}

##### routes: []
配置式路由，与react-router@4配置基本一致。

注：
- 嵌套路由，父路由可不指定component，默认匹配第一个子路由对应的component；

##### disableRedirectHoist: Boolean
禁用 redirect 上提，即禁止把所有 redirect 声明提到路由最前面进行匹配

##### history: String
指定 history 类型，可选 browser、hash 和 memory

##### outputPath: String
指定打包输出路径，默认值 ./dist

##### base: String
指定 react-router 的 base，部署到非根目录时需要配置。默认值 /

##### publicPath: String
指定 webpack 的 publicPath，指向静态资源文件所在的路径。默认值 /

##### context: {}
配置全局 context，会覆盖到每个 pages 里的 context

#### webpack配置
umi集成了webpack-chain，通过chainWebpack扩展或修改 webpack 配置。如：
```
chainWebpack(config, { webpack }) {
  // 设置 alias
  config.resolve.alias.set('a', 'path/to/a');

  // 删除进度条插件
  config.plugins.delete('progress');
  
  //css的修改
  config.plugin('extract-css').use(require('mini-css-extract-plugin'), [
    	{
    		filename: `[name].css`,
    		chunkFilename: `[name].[contenthash:8].chunk.css`,
    	},
    ]);
  //js的修改
  config.output.filename('[name].js');
}
```
同时，umi把webpack基本配置单独抛出，可以直接在.umirc.js中配置。如：
```
import path from 'path';
import pxToViewPort from 'postcss-px-to-viewport';
export default {
  treeShaking: true,
  plugins: [
    // ref: https://umijs.org/plugin/umi-plugin-react.html
    ['umi-plugin-react', {
      antd: true,
      dva: true,
      dynamicImport: { webpackChunkName: true },
      title: 'umi-admin',
      dll: false,
      
      routes: {
        exclude: [
          /models\//,
          /services\//,
          /model\.(t|j)sx?$/,
          /service\.(t|j)sx?$/,
          /components\//,
        ],
      },
    }],
  ],
  alias: {
    Components: path.resolve(__dirname, 'src/components/'),
    Util: path.resolve(__dirname, 'src/util/'),
    Api: path.resolve(__dirname, 'src/api/')
  },
  //postCss插件配置
  extraPostCSSPlugins: [
    pxToViewPort({
      viewportWidth: 375,
      viewportHeight: 667,
      unitPrecision: 5,
      viewportUnit: 'vw',
      selectorBlackList: [],
      minPixelValue: 1,
      mediaQuery: false,
    }),
  ],
  devServer：{
    ...
  },
  "proxy": {
    "/api": {
      "target": "http://localhost:8000/",
      "changeOrigin": true,
      "pathRewrite": { "^/api" : "" }
    }
  }
}
```
#### 环境变量
umi环境变量是UMI_ENV，可以指定UMI_ENV的值获取不同的配置文件。如：
```
UMI_ENV=local
.umirc.local.js

UMI_ENV=cloud
.umirc.local.js
```
**不管什么环境下的配置文件，最终都会和.umirc.js合并后返回。.umirc.js的优先级最高**
### 部署
##### 部署 html 到非根目录
项目采用browserRouter且部署到非根目录时，路由匹配不到，导致页面没有任何展示也不报错。如果想把项目部署到自定义目录，需要配置base。如：
```
export default {
  base: '部署目录',  //默认是 / ，根目录
};
```
还可以修改项目路由方式，改外hashRouter。如：
```
export default {
  history: 'hash',
};
```
##### 静态资源加载
如果项目的静态资源放在非跟目录，或者通过cdn访问，需要通过webpack的publicPath指定静态资源访问路径。如：
```
export default {
  publicPath: "静态资源访问路径、cdn"
}
```



### [API介绍](https://umijs.org/zh/api/)
##### 路由相关
```
//Link：声明式路由跳转
import Link from 'umi/link';
<Link to="/list">Go to list page</Link>
//或者NavLink
import NavLink from 'umi/link';
<NavLink to="/list" activeClassName="active">Go to list page</NavLink>


//编程式路由跳转
import router from 'umi/router';
router.push('/list');
router.replace('/list');
router.go(-1);
router.goBack();


//重定向
import Redirect from 'umi/redirect';
<Redirect to="/login" />;


//路由询问prompt
import Prompt from 'umi/prompt';
<Prompt
    when={status}
    message={location => {
        return window.confirm(`confirm to leave to     ${location.pathname}?`);
        }}
/>

//withRouter
import withRouter from 'umi/redirect';
export default withRouter(Component);
```
##### 按需加载
###### 组件按需加载
```
import dynamic from 'umi/dynamic';

const delay = (timeout) => new Promise(resolve => setTimeout(resolve, timeout));
const App = dynamic({
  loader: async function() {
    await delay(/* 1s */1000);
    return () => <div>I will render after 1s</div>;
  },
});
```
###### 模块按需加载
```
import('g2').then(() => {
  // do something with g2
});
```
