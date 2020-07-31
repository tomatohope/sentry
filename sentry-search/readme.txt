
Sentry是一个C/S架构，我们需要在自己应用中集成Sentry的SDK才能在应用发生错误是将错误信息发送给Sentry服务端

sentry client ----消息上报------>web-------> redis(消息处理)-------->worker-------->pgsql
                          |                                                      |
                          | <---- cron(服务探活)---------------------------------->


################# sentry deploy  by k8s
1## image prepare &&  you maybe prepare docker hub by reading dockerhub_prepare

2## prepare k8s yml for sentry roles
#https://kubernetes.io/docs/

#prepare SENTRY_SECRET_KEY for communication of sentry containers
docker run --rm 192.168.10.109:5000/basic/sentry:9.1.2 config generate-secret-key

redis.yml
pgsql.yml
sentry-init-db.yml
sentry-init-user.yml
sentry-web.yml
sentry-cron.yml
sentry-worker.yml

3## started one by one order by as follow:
redis
pgsql
#init-db for the first; then the data will be saved as local
#init-user for the first
web
cron
worker

4## config after login:
	url:  #please config nginx proxy
	https://sentry.xxx.com
	root email:
	xxx@xxx.com


5## create a prpject && Capture an Error test
# because no nginx proxy , use DSN  is without ssl and with port  DSN=http://8d13d4d5ba4c431fa3baa208a07c157e:1a0cbd2bb2944a22b44d6bc25eb13384@192.168.1.117:31772/3
#default DSN : https://8d13d4d5ba4c431fa3baa208a07c157e:1a0cbd2bb2944a22b44d6bc25eb13384@sentry.xxx.com/3

https://docs.sentry.io/error-reporting/quickstart/?platform=python#pick-a-client-integration
pip install --upgrade sentry-sdk==0.16.2

import sentry_sdk
sentry_sdk.init("http://8d13d4d5ba4c431fa3baa208a07c157e:1a0cbd2bb2944a22b44d6bc25eb13384@192.168.1.117:31772/3")

try:
    a_potentially_failing_function()
except Exception as e:
    # Alternatively the argument can be omitted
    sentry_sdk.capture_exception(e)

6## some commands:
kubectl get all -n namespace
kubectl logs -f   podname  -n namespace
kubectl describe pod podname  -n namespace
kubectl  exec -it podname env -n namespace
