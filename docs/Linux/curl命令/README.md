# curl命令

## 普通请求

```
curl http://localhost:8080/payment/1
```



## 携带cookie

```
curl http://localhost:8080/payment/1 --cookie "username=xgc"
```



## 携带请求头

```
curl http://localhost:8080/payment/1 -H "X-Request-Id:123"
```

## post请求

```
curl -X POST "http://localhost:8080/payment"
```