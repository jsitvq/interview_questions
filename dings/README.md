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

2. json API  

把查询语句转化成JSON API；不同系统之间接口对接案例；  
（json 脚本用postman提交）  

```
 select distinct      
		       convert(nvarchar(30),t0.belnr_id) + '-' + convert(nvarchar(30),t1.belpos_id)  as mo_code -- 工单号+序号  
			 ,t1.itemcode  as material_code                           -- 产品编码  
		      	 ,case when t1.whscode = '102' then '201' else t1.whscode end as warehouse -- 入库仓  
		          ,case when t1.WhsCode ='203' then 'OS' else 'NORMAL' end  as mo_type -- 工单类型  
		          ,t1.menge as mo_plan_qty                                 -- 计划数量  
		           ,cast(t1.anfzeit as date) as mo_plan_start_date          -- 预计开工日期  
                         ,cast(t1.endzeit as date) as mo_plan_end_date            -- 预计完工日期  
		  ,case when t1.abgkz = 'N' and isnull(t0.SPERRUNG,'') in ('N','') then '10' else '20' end as mo_status -- 工单状态   
		    ,'SYNC' as sync_status                                   -- 同步状态，YNC同步 DELETE删除  
		   ,case isnull(t0.PRIOR_ID,'Std') when 'Std' then 1 else 2 end as order_num -- 优先级  
		   ,t0.AUFTRAGINT as sales_order                            -- 销售订单  
		     ,t0.KND_ID as customer_code                              -- 客户代码  
		     ,'' as customer_name                                     -- 客户名称  
		     ,t1.din as u_makenum                                     -- 制造单  
		     ,left(t1.ZUSATZTEXT,10) as mo_memo                       -- 备注  
		    ,t0.belnr_id  as belnr_id                                -- 工作令  
                   ,t1.belpos_id  as belpos_id                              -- 工作令位置  
                    ,t1.ZU_BELPOS_ID  as ZU_BELPOS_ID                        -- 上级工作令位置  
             ,isnull(t1.ME_VERBRAUCH,'')  as ME_VERBRAUCH             -- 工作令单位（报工单位）  
               ,isnull(t1.ME_LAGER,'')  as ME_LAGER                     -- 库存单位  
                ,isnull(t1.ME_UMR,1)  as ME_UMR                          -- 单位转换利率（报工单位/库存单位）  
                 from  beas_fthaupt t0  
	             inner join beas_ftpos t1 on t0.belnr_id = t1.belnr_id  
 where 1=1  
                            and convert(nvarchar(30), t0.belnr_id) + '-' + convert(nvarchar(30), t1.belpos_id) = '49710-10'  
```

3. 跨不同数据源读写数据

不同系统不同的数据源读取，比如postgresql、mysql、sqlserver,  
   仓储系统数据源是postgresql，ERP数据源是sqlserver；  
   如果仓储系统、ERP系统的库存进行差异比较，如何操作？  

