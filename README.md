# OnlineJudge Front End
1. 增加首页周榜（随便塞的
2. 问题界面更新
  <br>
  **增加**
  - 提交反馈
  <br>
  **更改**
  - 问题界面格式化
  - 时空限制和IO模式位置调整
  - 竞赛中的难度和标签隐藏
  <br>
  **删除**
  - 题目信息中的ID、出题人
3. 提交状态列表界面更新，删除ID
4. 管理员竞赛列表增加删除按钮

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