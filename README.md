# AutoSignMachine

**一个自动执行任务的工具，通过它可以实现账号自动签到，自动领取权益等功能，帮助我们轻松升级。**


## 联通APP签到任务
**实现现联通帐号的每日签到任务。**
详细功能目录如下:

* **每日签到积分**

```sh
node index.js unicom --user 131*******12 --password 11****11 --appid f7af****ebb
```

### docker部署
```sh
# 构建
docker build -t auto-sign-machine:latest  -f docker/Dockerfile .
# 运行(cookies和账号密码两种方式二选一)
docker run \
  --name auto-sign-machine \
  -d \
  --label traefik.enable=false \
  -e enable_unicom=true \
  -e user=131*******12 \
  -e password=11****11 \
  -e appid=f7af****ebb \
  auto-sign-machine:latest
```

### 注意
#### cron中`%`号需要转义`\%`

### 脚本运行机制
任务并非在一次命令执行时全部执行完毕，任务创建时会根据某个时间段，将所有任务分配到该时间段内的随机的某个时间点，然后使用定时任务定时运行脚本入口，内部子任务的运行时机依赖于任务配置项的运行时间及延迟时间，这种机制意味着，只有当脚本的运行时间在当前定时任务运行时间之前，脚本子任务才有可能有选择的被调度出来运行

### crontab 任务示例
在4-23小时之间每隔三十分钟尝试运行可运行的脚本子任务
```txt
*/30 4-23 * * * /bin/node /workspace/AutoSignMachine/index.js unicom --user 1******5 --password 7****** --appid 1************9
```

### 多用户配置
启用`--accountSn`表示账户序号，例如`1,2`, 则将提取`option-sn`选项的值，例如`user-1`,`user-2`

### 配置文件示例
启用`--config /path/to/mycfg.json`表示配置文件
```json
{
    "accountSn": "1,2",
    "user-1": "22******1",
    "password-1": "31******1",
    "appid-1": "41******1",
    "user-2": "25******1",
    "password-3": "72******1",
    "appid-2": "92******1"
}
```

### 运行测试
```sh
## 立即模式, 一次性执行所有任务，仅建议测试任务是否正常时运行，该方式无法重试周期任务
## 该模式不缓存cookie信息，频繁使用将可能导致账号安全警告
#增加 --tryrun

## 指定任务模式，可以指定仅需要运行的子任务，多用户使用规则参看`多用户配置`
#增加 --tasks taskName1,taskName2,taskName3
```

### GitHub Actions 运行问题

1.将本代码仓库fork到自己的github。  
2.点击Settings选项卡，点击左侧Secrets，点击New secret，创建对应参数，这些值不会被公开。  
3.点击Actions选项卡，自己修改。  
