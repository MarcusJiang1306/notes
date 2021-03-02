

首先会被AbstractHmilyTransactionAspect切面里的注解拦截器拦截下来

然后加载了一个LocalParameterLoader拿到context，但context是空的，仔细研究，其实发起者是空的，参与者就是有context的了，

HmilyTransactionAspectInvoker第一次执行![image-20210222234043488](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210222234043488.png)

```
@Override
public Object handler(final ProceedingJoinPoint point, final HmilyTransactionContext context)
        throws Throwable {
    Object returnValue;
    Supplier<Boolean> histogramSupplier = null;
    Optional<MetricsHandlerFacade> handlerFacade = MetricsHandlerFacadeEngine.load();//默认加载了个MetricsTrackerHandlerFacade
    try {
        if (handlerFacade.isPresent()) {
        //但监控没开 所以counterIncrement是跳过的
            handlerFacade.get().counterIncrement(MetricsLabelEnum.TRANSACTION_TOTAL.getName(), TransTypeEnum.TCC.name());
            histogramSupplier = handlerFacade.get().histogramStartTimer(MetricsLabelEnum.TRANSACTION_LATENCY.getName(), TransTypeEnum.TCC.name());//这个没看是不是跳过的 待会看看
        }
        //这个是个关键，preTry就是构建一个Transaction，注册参与者，保存Transaction和TransactionContext的地方
        HmilyTransaction hmilyTransaction = executor.preTry(point);
        try {
            //execute try
            returnValue = point.proceed();
            hmilyTransaction.setStatus(HmilyActionEnum.TRYING.getCode());
            //更新状态没注意看
            executor.updateStartStatus(hmilyTransaction);
        } catch (Throwable throwable) {
            //if exception ,execute cancel
            final HmilyTransaction currentTransaction = HmilyTransactionHolder.getInstance().getCurrentTransaction();
            disruptorProviderManage.getProvider().onData(() -> {
                handlerFacade.ifPresent(metricsHandlerFacade -> metricsHandlerFacade.counterIncrement(MetricsLabelEnum.TRANSACTION_STATUS.getName(),
                        TransTypeEnum.TCC.name(), HmilyRoleEnum.START.name(), HmilyActionEnum.CANCELING.name()));
                executor.globalCancel(currentTransaction);
            });
            throw throwable;
        }
        //execute confirm
        final HmilyTransaction currentTransaction = HmilyTransactionHolder.getInstance().getCurrentTransaction();
        disruptorProviderManage.getProvider().onData(() -> {
            handlerFacade.ifPresent(metricsHandlerFacade -> metricsHandlerFacade.counterIncrement(MetricsLabelEnum.TRANSACTION_STATUS.getName(),
                    TransTypeEnum.TCC.name(), HmilyRoleEnum.START.name(), HmilyActionEnum.CONFIRMING.name()));
            executor.globalConfirm(currentTransaction);
        });
    } finally {
        HmilyContextHolder.remove();
        executor.remove();
        if (null != histogramSupplier) {
            histogramSupplier.get();
        }
    }
    return returnValue;
}
```

![image-20210228202803057](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210228202803057.png)

这里有两个holder，一个是`hmilyTransaction`的，另外一个是`HmilyTransactionContext`的holder，最后把`hmilyTransaction`返回。

在整个执行过程中，DubboHmilyTransactionFilter是rpc调用的其中一环，每个filter都会被invoke，这个filter主要负责把context传到参与者，来完成跨进程的事务的协调



有个HmilyTransactionSelfRecoveryScheduled.selfTccRecovery负责tcc恢复 具体再研究

一个tcc整个的执行的顺序

![image-20210302234113488](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210302234113488.png)