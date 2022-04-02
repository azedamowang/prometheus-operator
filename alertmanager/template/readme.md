命令测试钉钉接口 加密使用IP地址
curl 'https://oapi.dingtalk.com/robot/send?access_token=XXXX'    -H 'Content-Type: application/json'    -d '{"msgtype": "text","text": {"content": "新人内容测试"}}'
