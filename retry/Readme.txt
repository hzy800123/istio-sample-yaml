Istio - TrafficManagement - Retries
重试机制与实验步骤

-URL:
https://developer.aliyun.com/article/790507

-Flow:
Client ---> Nginx-Service ---> Httpd-Service

-Steps:
(1) Apply the YAML files to create the items below:
Deployment
Service
Virtual-Service

(2) 配置 nginx 反向代理 httpd 以完成上下游调用的效果
kubectl exec -it nginx-deployment-56c94b9957-xgw88 -- sh

tee /etc/nginx/conf.d/default.conf <<-'EOF'
server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://httpd-service;
        proxy_http_version 1.1;
    }
}
EOF

nginx -t
nginx -s reload

(3) 通过 kubectl exec -it client-deployment-56c94b9957-xgw88 -- sh 进入容器
执行 wget -q -O - http://nginx-service 测试

(4) 执行 kubectl logs -f nginx-deployment-86684f9cf6-lvxtj  -c istio-proxy -n demo 观测SideCar边车日志

____________________


#### 重试相关参数配置

##### retries 参数
可以定义请求失败时的策略，重试策略包括重试次数、超时、重试条件
attempts: 必选字段，定义重试的次数
perTryTimeout: 单次重试超时的时间，单位可以是ms、s、m和h
retryOn: 重试的条件，可以是多个条件，以逗号分隔

##### retryOn 参数
5xx：在上游服务返回5xx应答码，或者在没有返回时重试
gateway-error：类似于5xx异常，只对502、503和504应答码进行重试
connect-failure：在链接上游服务失败时重试 
retriable-4xx：在上游服务返回可重试的4xx应答码时执行重试
refused-stream：在上游服务使用REFUSED_STREAM错误码重置时执行重试
cancelled：gRPC应答的Header中状态码是cancelled时执行重试
deadline-exceeded：在gRPC应答的Header中状态码是deadline-exceeded时执行重试
internal：在gRPC应答的Header中状态码是internal时执行重试
resource-exhausted：在gRPC应答的Header中状态码是resource-exhausted时执行重试
unavailable：在gRPC应答的Header中状态码是unavailable时执行重试。


