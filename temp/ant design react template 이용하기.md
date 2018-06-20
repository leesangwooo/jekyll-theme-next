ant design pro 
> *Getting Started*
> [https://pro.ant.design/docs/getting-started](https://pro.ant.design/docs/getting-started)

- 폴더 구조
```
├── mock                     # Local Mock Data
├── public
│   └── favicon.ico          # Favicon
├── src
│   ├── assets               # Local static files
│   ├── common               # Common Configuration like Navigation
│   ├── components           # Components
│   ├── e2e                  # Integrated Test Case
│   ├── layouts              # Common Layouts
│   ├── models               # dva Model
│   ├── routes               # Sub-pages and templates
│   ├── services             # Back-end Services
│   ├── utils                # Utility
│   ├── g2.js                # Dataviz Configuration
│   ├── theme.js             # Theme Configuration
│   ├── index.ejs            # HTML Entry
│   ├── index.js             # App Entry
│   ├── index.less           # Global Stylesheet
│   └── router.js            # Route Entry File
├── tests                    # Tests Configuration
├── README.md
└── package.json
```

- react의 동작 시스템
	1. index.ejs
		- root element를 갖고있는 html entry
		- `.ejs` : html template 생성을 위한 모듈 (jsp와 유사)

	2. index.js
     * dva를 사용하여 react app 만들기
     > dva 작동구조 : https://medium.com/@chenchengpro/dva-1-0-a-lightweight-framework-based-on-react-redux-and-redux-saga-eeeecb7a481d
     >dva docs : https://github.com/dvajs/dva/blob/master/docs/API.md
     >antd & dva : https://ant.design/docs/react/practical-projects
     > 라우트, 모델, 서비스 구조 : https://github.com/dvajs/dva/blob/master/docs/API.md#model
     > > /routes/ProjectManagement/Account.js
     > > /model/form.js
     > > /services/api.js
     
     
    ```javascript
    // 1. app 객체 생성 및 초기화
    // 임포트한 사용자 Browser의 History를 조회 기능의 스크립트에서  createHistory()를 호출하여
    // 라우터를 위한 history 옵션 초기화하기
    const app = dva({
      history: createHistory(),
    });

    // 2. app.use(hooks) : 글로벌 에러 핸들링 등을 위한 hooks 플러그인 사용
    app.use(createLoading());

    // 3. model 등록
    app.model(require('./models/global').default);
    // * javascript generator 개념 : http://meetup.toast.com/posts/73

    // 4. Router 등록
    app.router(require('./router').default);

    // 5. Start
    app.start('#root');
    ```
    
    - router config : `router.js`
    ```javascript
    import React from 'react';
    import { routerRedux, Route, Switch } from 'dva/router';
    import { LocaleProvider, Spin } from 'antd';
    import zhCN from 'antd/lib/locale-provider/zh_CN';
    import dynamic from 'dva/dynamic';
    import { getRouterData } from './common/router';
    import Authorized from './utils/Authorized';
    import styles from './index.less';

    const { ConnectedRouter } = routerRedux;
    const { AuthorizedRoute } = Authorized;
    dynamic.setDefaultLoadingComponent(() => {
      return <Spin size="large" className={styles.globalSpin} />;
    });

    function RouterConfig({ history, app }) {
      const routerData = getRouterData(app);
      const UserLayout = routerData['/user'].component;
      const BasicLayout = routerData['/'].component;
      return (
        <LocaleProvider locale={zhCN}>
          <ConnectedRouter history={history}>
            <Switch>
              // url path가 "/user"를 포함하는 경우 UserLayout 컴포넌트를 라우팅
              <Route path="/user" component={UserLayout} />
              
              // url path가 "/"를 포함하는 경우 BasicLayout 컴포넌트를 라우팅
              <AuthorizedRoute
                path="/"
                render={props => <BasicLayout {...props} />}
                authority={['admin', 'user']}
                redirectPath="/user/login"
              />
            </Switch>
          </ConnectedRouter>
        </LocaleProvider>
      );
    }

    export default RouterConfig;
    ```

    
- 알아야 할 것들 
    - javascript 문법
    	- import
    	- yeild : generator 개념
    	- 상태유지 : [localStorage](https://developer.mozilla.org/ko/docs/Web/API/Window/localStorage)

//basic layout
// sub-menu의 화면 이동 이벤트
// import { routerRedux } from 'dva/router';
  handleMenuClick = ({ key }) => {
    if (key === 'logout') {
      this.props.dispatch({
        // ../models/login.js 에 effect로 등록된 logout 함수 호출
        type: 'login/logout',
      });
    }
    if (key === 'account') {
      // ../common/router.js 에 매핑된 주소로 dispatch 이동
      this.props.dispatch(routerRedux.push('/projectmanagement/account'));
      return;
    }

  };
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    