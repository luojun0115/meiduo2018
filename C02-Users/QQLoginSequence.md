```sequence
Title: QQ登录时序图
浏览器->服务器: 请求用于QQ登录的URL网址（携带next参数，表示登录成功后进入美多的哪个页面）
服务器->浏览器: 返回QQ登录网址
浏览器-->QQ服务器: 用户在QQ网页进行登录
QQ服务器-->浏览器: 用户在QQ登录成功，QQ将用户重定向到服务器的callback网址，并携带授权code与state，其中state就是上面提到的next参数
浏览器->服务器: 访问callback?code=xxx&state=xxx
服务器->QQ服务器: 凭借code向QQ服务器请求access token
QQ服务器->服务器: 返回access token
服务器->QQ服务器: 凭借access token向QQ服务器请求用户的openid
QQ服务器->服务器: 返回用户的openid(用户的唯一身份标识)
服务器->服务器: 判断用户是否是第一次使用QQ登录
服务器-->浏览器: 如果用户不是第一次QQ登录，则登录成功，返回JWT token，用户跳转到state指明的页面
服务器-->浏览器: 如果用户是第一次使用QQ登录，则生成绑定用户身份的access token并返回
浏览器-->服务器: 携带手机号、密码、短信验证码、access token请求绑定QQ用户身份
服务器-->服务器: 如果服务器中存在用户数据，直接绑定；如果不存在，创建用户并绑定身份
服务器-->浏览器: 返回登录成功的JWT token，用户跳转到state指明的页面
```

```flow
st=>start: 开始
e=>end: 结束
io=>inputoutput: 接收手机号、密码、短信验证码、access_token
co=>condition: 判断access_token是否正确
op1=>operation: 错误返回
co2=>condition: 判断短信验证码是否正确
op2=>operation: 错误返回
co3=>condition: 判断用户是否存在
co4=>condition: 判断密码是否正确
op3=>operation: 创建并保存用户
op4=>operation: 设置用户与openid的关联信息
op5=>operation: 错误返回
st->io->co
co(no)->op1
co(yes)->co2
co2(no)->op2
co2(yes)->co3
co3(yes)->co4
co3(no)->op3
co4(no)->op5
co4(yes)->op4
op3->op4
```
