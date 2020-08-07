# Zabbix配置文档（二）—— 报警


## 设置发送邮件报警

### 定义发件人

通过管理 - 报警媒介类型，选择 Email 来连接，先录入自己的邮件的 SMTP 信息

![6_sendEmail.png](https://img.zephyrl.co/images/2020/08/05/6_sendEmail.png)

### 添加收件人

我们在个人页面可以找到接受报警媒介的地方，添加个人信息即可

![7_recvEmail.png](https://img.zephyrl.co/images/2020/08/05/7_recvEmail.png)

### 开启动作

我们设置完收发件人之后，当有报警还是不会发送到邮箱。是因为没有设置动作。

我们可以通过以下步骤找到动作选项。

![8_findAction.gif](https://img.zephyrl.co/images/2020/08/06/8_findAction.gif)

点击一下右面的“已禁用”，让他启动。之后点击名字进入选项。

首先我们修改一下发通知的条件

![9_setAction.png](https://img.zephyrl.co/images/2020/08/06/9_setAction.png)

接着点击操作，按照图示进行操作

![10_setAction2.png](https://img.zephyrl.co/images/2020/08/06/10_setAction2.png)

之后我打开压力测试，让机器满载，测试一下是否会发送邮件

![11_fullStressTest.png](https://img.zephyrl.co/images/2020/08/06/11_fullStressTest.png)

此时我们可以发现已经发送成功

![12_sendSuccess.png](https://img.zephyrl.co/images/2020/08/06/12_sendSuccess.png)

打开报表日志也可以查看详细信息

![13_sendSuccess2.png](https://img.zephyrl.co/images/2020/08/06/13_sendSuccess2.png)

打开邮箱，查看详细信息

![14_sendSuccess3.png](https://img.zephyrl.co/images/2020/08/06/14_sendSuccess3.png)

## 钉钉群聊报警机器人

### 添加脚本

首先，我们要找到 Zabbix 默认存放脚本的地方，可以使用 `grep` 筛选

```shell
[root@zep0 ~]# grep -Ev '^$|#' /etc/zabbix/zabbix_server.conf
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/var/run/zabbix/zabbix_server.pid
SocketDir=/var/run/zabbix
DBHost=127.0.0.1
DBName=zabbix
DBUser=zabbix
DBPassword=Lza0423.
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
Timeout=4
AlertScriptsPath=/usr/lib/zabbix/alertscripts
ExternalScripts=/usr/lib/zabbix/externalscripts
LogSlowQueries=3000
StatsAllowedIP=127.0.0.1
```

在倒数第四行就是我们要找的 `alertscripts` 文件夹，我们进入文件夹，创建 `dingding.py`

```shell
vim /usr/lib/zabbix/alertscripts/dingding.py
```

插入以下脚本内容

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-

import requests
import json
import sys
import os

headers = {'Content-Type': 'application/json;charset=utf-8'}
api_url = "https://oapi.dingtalk.com/robot/send?access_token=youraccesstoken"

def msg(text):
    json_text= {
     "msgtype": "text",
        "at": {
            "atMobiles": [
                "电话号码"
            ],
            "isAtAll": False
        },
        "text": {
            "content": text
        }
    }
    print requests.post(api_url,json.dumps(json_text),headers=headers).content

if __name__ == '__main__':
    text = sys.argv[1]
    msg(text)
```

### 添加自定义机器人并复制 Webhook

点击 智能群管理，添加自定义机器人

![18_addOtherRobot.png](https://img.zephyrl.co/images/2020/08/07/18_addOtherRobot.png)

之后下拉寻找，创建好后复制自己的 `webhook` 代码，并且将脚本的 api_url 改为自己的 `webhook` 代码

![17_addRobotFindWebhook.png](https://img.zephyrl.co/images/2020/08/07/17_addRobotFindWebhook.png)

然后添加一些你要发送的内容中的关键词

![19_addRobotKeyword.png](https://img.zephyrl.co/images/2020/08/07/19_addRobotKeyword.png)

### 在 Zabbix Web 面板中设置

脚本写完后，我们进入 Zabbix web 面板，选择管理——报警媒介类型——创建媒介类型

![15_addAlertMedia.png](https://img.zephyrl.co/images/2020/08/06/15_addAlertMedia.png)

暂时可以先按照这里写，脚本可以根据自己微调后修改

接着，添加用户报警媒介,收件人随便写写就行：

![16_addAlertUserMedia.png](https://img.zephyrl.co/images/2020/08/06/16_addAlertUserMedia.png)

最后，在配置 —— 动作中添加动作“发送到钉钉”：

![16_addAction.png](https://img.zephyrl.co/images/2020/08/07/16_addAction.png)

添加故障发生的 Custom message

```templates
标题：故障发生: {TRIGGER.STATUS} 服务器: {HOSTNAME1}
消息：
告警主机：{HOSTNAME1}
告警事件：{EVENT.DATE} {EVENT.TIME}
告警等级：{TRIGGER.SEVERITY}
告警信息：{TRIGGER.NAME}
告警项目：{TRIGGER.KEY1}
问题详情：{ITEM.NAME}:{ITEM.VALUE}
当前状态：{TRIGGER.STATUS}:{ITEM.VALUE1}
```

添加故障解决的 Custom message

```templates
标题：问题：{ITEM.NAME}:{ITEM.VALUE} 已解决
消息：
问题已解决！
问题：{ITEM.NAME}:{ITEM.VALUE}
主机：{HOST.NAME}
解决时间：{EVENT.DATE} {EVENT.TIME}
持续时间：{EVENT.DURATION}
问题等级：{EVENT.SEVERITY}
问题ID：{EVENT.ID}
{TRIGGER.URL}
```

以上内容添加到 `钉钉告警` 的 Templates 里面也可以 

之后我们可以进行压力测试，看一看能不能正常报警，或者干脆关掉机器

我在这里使用 `stress` 将 `swap` 沾满，此时报警最快最省力

很快就收到了报警：

![20_dingdingWarningAlert.png](https://img.zephyrl.co/images/2020/08/07/20_dingdingWarningAlert.png)

当问题被解决，也可以收到提示：

![21_dingdingSolvedAlert.png](https://img.zephyrl.co/images/2020/08/07/21_dingdingSolvedAlert.png)

此时钉钉的报警已经设置成功啦
