---
layout: post
title: 'Java后台耗时任务思路'
tags: [code]
---

## 前记
>我们在处理后台任务的时候，经常会遇到某个任务需要较长时间，没法及时给前台反馈结果的情况，此时我们希望前台查询或者刷新的时候给出当前任务的执行状态，而不是一直空耗阻塞着，由此诞生了以下思路，当然Spring已经提供了对应的框架实现，这里只是对这种思路的处理方式的一种探究

- 耗时任务
```java
class DefaultTimeConsumingTast implements Callable{
        private String id;
        DefaultTimeConsumingTast(String id){
            this.id=id;
        }
        @Override
        public User call() {
            User user=new User();
            try {
                Thread.currentThread().sleep(5000);
                user.setAge(Integer.parseInt(id));
                return user;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return null;
        }
    }
```
- 状态查询响应
```java
@RequestMapping(value = "/getAge",method = RequestMethod.GET)
    @ResponseBody
    public User getUser1(@RequestParam("id")String id){
        User u=new User();
        u.setMessage("未完成");
        if(futures.containsKey(id)){
            Future<User> userFuture=futures.get(id);
            if(userFuture.isDone()){
                try{
                    u=userFuture.get();
                    u.setMessage("已完成");
                }catch (InterruptedException | ExecutionException e){
                    e.printStackTrace();
                }
                futures.remove(id);
            }
        }else{
            Future<User> future=exec.submit(new DefaultTimeConsumingTast(id));
            futures.put(id,future);
        }
        return u;
    }
```
- 全局声明变量或对象
```java
ExecutorService exec= Executors.newFixedThreadPool(2);
private static Map<String,Future<User>> futures=new HashMap<>();
```

