#### [鼎智](https://www.dingsmotion.cn/)面试题

1. 触发器
```
 select TRANS_ID as ID,CSENDID ,CSOURCECLSID,CSOURCEID,  
                        CSYMBOL 图号, CNAME 名字, CSENDER 用户, CVERSION 版本,  
                        case when CSTATE = '1' then '进行中'  
                        when CSTATE = '0' then '集成运行完成'  
                        when CSTATE = '-1' then '失败'  
                        when CSTATE = '2' then '成功' end CSTATE,   
                        CRESULT 状态, CSENDDATE, CENDDATE,'明细' BTNTEXT,CJSON,'查看发送JSON' AS SENDJSON  
                        from INTERFACE_SEND_LOG   
                        where CSENDDATE >= '2024-10-01'  
                        and CSENDDATE < '2024-10-31'  
                        order by  CSENDDATE desc  
```

此表是一张接口日志表，当传递错误时CSTATE=’-1’,点重新发送会新生成一行数据；当新的数据发送成功，需要原数据状态字段增加成功字段；点重发时CSOURCEID是唯一值；写出实现方法；  

