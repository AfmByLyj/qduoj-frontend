# OnlineJudge Front End
<details>
  <summary>增加首页周榜</summary>
  <ul><li>界面优化</li></ul>
</details>
<details>
  <summary>问题界面更新</summary>
  <ul>
    <li>
      <details>
        <summary>增加</summary>
        <ul>
          <li>提交反馈</li>
          <li>在线IDE括号自动匹配，主题更新</li>
        </ul>
      </details>
    </li>
    <li>
      <details>
        <summary>更改</summary>
        <ul>
          <li>问题界面格式化</li>
          <li>时空限制和IO模式位置调整</li>
          <li>竞赛中的难度和标签隐藏</li>
        </ul>
      </details>
    </li>
    <li>
      <details>
        <summary>删除</summary>
        <ul>
          <li>题目信息中的ID、出题人</li>
        </ul>
      </details>
    </li>
  </ul>
</details>
<details>
  <summary>提交信息</summary>
  <ul>
    <li>详细页代码中制表符大小更改</li>
    <li>提交状态列表界面更新，删除ID</li>
  </ul>
</details>
<details>
  <summary>管理员界面</summary>
  <ul>
    <li>管理员竞赛列表增加删除按钮</li>
  </ul>
</details>

## Get Started

Install nodejs **v8.12.0** first.

### Linux

```bash
npm install
# we use webpack DllReference to decrease the build time,
# this command only needs execute once unless you upgrade the package in build/webpack.dll.conf.js
export NODE_ENV=development 
npm run build:dll

# the dev-server will set proxy table to your backend
export TARGET=http://Your-backend

# serve with hot reload at localhost:8080
npm run dev
```
### Windows

```bash
npm install
# we use webpack DllReference to decrease the build time,
# this command only needs execute once unless you upgrade the package in build/webpack.dll.conf.js
set NODE_ENV=development 
npm run build:dll

# the dev-server will set proxy table to your backend
set TARGET=http://Your-backend

# serve with hot reload at localhost:8080
npm run dev
```
