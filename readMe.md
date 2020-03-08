# React 환경에서의 webpack 정리(1)

react 개발을 진행하면서 웹팩에 대한 개념을 정리하고 이를 통해서 손쉽게 웹팩을 다룰수 있도록 목표를 갖고 작성하였습니다.

## 모듈 번들러란?

- 여러개의 나누어져 있는 파일들을 하나의 파일로 만들어주는 라이브러리를 말함
- 웹팩, parcel등이 이에 해당함.
- 모듈 번들러를 통해서 여러개의 자바 스크립트 파일을 하나의 파일로 묶어 한번의 요청에 모두 가져올수 있는 장점이 생김
- 또한, 최신 자바스크립트 문법을 브라우저에서 사용할수 있게 해주었음.
- 마지막으로, 모듈 번들러를 통해서 자바스크립트 코드를 압축하고 최적화 할수 있어서 로딩속도를 높일수 있음.
- 물론, 수많은 자바 스크립트 파일이 하나의 파일로 묶인다면 초기 로딩속도가 오래걸린다
- 이러한 문제 해결을 위해서 청크, 캐시, 코드 스플릿 등의 개념이 생김



# 실습

### 프로젝트 설정

```
mkdir webpack-react
cd webpack-react
yarn init -y

//실습에 사용할 라이브러리 모두 설치 진행
yarn add -D @babel/core @babel/preset-env @babel/preset-react babel-loader clean-webpack-plugin css-loader html-loader html-webpack-plugin mini-css-extract-plugin node-sass react react-dom sass-loader style-loader webpack webpack-cli webpack-dev-server

```

### 웹팩으로 자바스크립트 파일 빌드

- src 디렉터리를 만들고 아래와 같이 진행

```
//src/test.js
console.log("webpack - test")

//최상위 디렉토리이동후 webpack.config.js 파일 작성
const path = require("path");

module.exports = {
  entry: "./src/test.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname + "/build")
  },
  mode: "none"
};

```

- entry 는 웹팩이 빌드할 파일을 알려주는 역할을 함.
- output은 웹팩에서 빌드를 완료하면 output에 명시되어 있는 정보를 통해 빌드파일을 생성
- mode는 웹팩의 옵션
  - production은 최적화 되어 빌드되어지는 특징을 가짐
  - development는 빠르게 빌드하는 특징
  - none 같은 경우 아무 기능 없이 웹팩으로 빌드

```
//package.json 파일 이동 하여 아래 코드를 추가
  "scripts": {
    "build": "webpack"
  },
```

- 명령어 webpack 실행시 디폴트로 실행할 파일은 같은 경로에 있는 webpack.config.js 이다.

```
//터미널
yarn build
```

- 명령어 실행후 build/bundle.js 파일이 생성된다.





### 웹팩을 통해서 HTML 파일 빌드하기

- 웹팩은 로더(loader)라는 기능을 통해서 자바스크립트 파일이 아닌 파일을 웹팩이 이해할수 있도록 도와준다.

```
module : {
	rules: {
		test: '가지고올 파일 정규식',
		use: [
			{
				loader: '사용할 로더 이름',
				options: { 사용할 로더 옵션 }
			}
		]
	}
}
```

```
//public/index.html 파일 생성

<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="utf-8" />
    <title>WEBPACK4-REACT</title>
  </head>
  <body>
    <noscript>스크립트가 작동되지 않습니다!</noscript>
    <div id="root"></div>
  </body>
</html>
```

- 그후 webpack.config.js 파일에 아래와 같은 html 관련 코드 추가(module 및 plugins 부분)

```
const path = require("path");
const HtmlWebPackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/test.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname + "/build")
  },
  mode: "none",
  module: {
    rules: [
      {
        test: /\.html$/,
        use: [
          {
            loader: "html-loader",
            options: { minimize: true }
          }
        ]
      }
    ]
  },
  plugins: [
    new HtmlWebPackPlugin({
			template: './public/index.html', // public/index.html 파일을 읽는다.
      filename: 'index.html' // output으로 출력할 파일은 index.html 이다.
    })
  ]
};
```

- html-webpack-plugin은 웹팩이 html 파일을 읽어서 html 파일을 빌드할수 있게 도와줌
- loader는 html 파일을 읽었을때 html-loader를 실행하여 웹팩이 이해할 수 있게 함
- options 은 minimize라는 코드 최적화 옵션을 추가함.(한줄로 나오게함)

```
//터미널
yarn build

//결과
build/index.html 파일 생성
```

### 웹팩으로 리엑트 빌드하기

```
//src/index.js

import React from "react";
import ReactDOM from "react-dom";
import Root from "./Root";

ReactDOM.render(<Root />, document.getElementById("root"));

//src/Root.js

import React from 'react';

const Root = () => {
  return (
    <h3>Hello, React</h3>
  );
};

export default Root;
```

- 그후 최상위 디렉토리에서 .babelrc 파일을 만들고 아래와 같은 내용 입력

```
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}

//바벨은 es6에서 es5로 자바스크립트를 변환해주는 역할을 한다.
```

```
//webpack.config.js 파일에 entry와 rules에 babel-loader를 추가해줌.

const path = require("path");
const HtmlWebPackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname + "/build")
  },
  mode: "none",
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: "/node_modules",
        use: ['babel-loader'],
      },
      {
        test: /\.html$/,
        use: [
          {
            loader: "html-loader",
            options: { minimize: true }
          }
        ]
      }
    ]
  },
  plugins: [
    new HtmlWebPackPlugin({
      template: './public/index.html', // public/index.html 파일을 읽는다.
      filename: 'index.html' // output으로 출력할 파일은 index.html 이다.
    })
  ]
};
```





출처 : [https://velog.io/@jeff0720/React-%EA%B0%9C%EB%B0%9C-%ED%99%98%EA%B2%BD%EC%9D%84-%EA%B5%AC%EC%B6%95%ED%95%98%EB%A9%B4%EC%84%9C-%EB%B0%B0%EC%9A%B0%EB%8A%94-Webpack-%EA%B8%B0%EC%B4%88](https://velog.io/@jeff0720/React-개발-환경을-구축하면서-배우는-Webpack-기초)



