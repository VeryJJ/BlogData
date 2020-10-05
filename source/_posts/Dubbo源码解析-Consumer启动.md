---
title: Dubbo源码解析-Consumer启动
tags: [Dubbo, RPC]
date: 2018-06-02 15:06:57
categories: 技术
---



- Dubbo Consumer的启动过程和Provider一样，以DubboNamespaceHandler为起点，去解析代码配置中的ReferenceBean。

```java
public class ReferenceBean<T> extends ReferenceConfig<T> implements FactoryBean, ApplicationContextAware, InitializingBean, DisposableBean {
```

<!--more-->

- ReferenceBean同样既继承了ReferenceConfig，又实现了InitializingBean。也是在afterProperitesSet()中去执行服务引用
  - ReferenceBean
    - afterPropertiesSet
      - ReferenceConfig
        - init()
          - 完成service interface的class， methods解析
          - 获取Service 注册中心registeries配置信息，用于向注册中西订阅service
          - 检测是否配置有Dubbo Mock， Dubbo Stub
          - createProxy()完成ReferenceConfig + Registeries ——》 Dubbo Service Invoker的转化。createProxy()返回时，返回的是被Proxy后的Invoker，即外层加了Dubbo Filter Chain。
            - DubboProtocol.refer(...)

### DubboProtocol



DubboProtocol.class 作为Dubbo RPC层的具体实现协议，尤其完成Consumer中向注册中心真正订阅的动作。



```java
public class DubboProtocol extends AbstractProtocol {
    ...
    public <T> Invoker<T> refer(Class<T> serviceType, URL url) throws RpcException {
        optimizeSerialization(url);
        // create rpc invoker with url.
        DubboInvoker<T> invoker = new DubboInvoker<T>(serviceType, url, getClients(url), invokers);
        invokers.add(invoker);
        return invoker;
    }
    
    private ExchangeClient[] getClients(URL url) {
        // whether to share connection
        boolean service_share_connect = false;
        int connections = url.getParameter(Constants.CONNECTIONS_KEY, 0);
        // if not configured, connection is shared, otherwise, one connection for one service
        if (connections == 0) {
            service_share_connect = true;
            connections = 1;
        }

        ExchangeClient[] clients = new ExchangeClient[connections];
        for (int i = 0; i < clients.length; i++) {
            if (service_share_connect) {
                clients[i] = getSharedClient(url);
            } else {
                clients[i] = initClient(url);
            }
        }
        return clients;
    }
    ...
}
```



### Invoker

```java
public interface Invoker<T> extends Node {

    /**
     * get service interface.
     *
     * @return service interface.
     */
    Class<T> getInterface();

    /**
     * invoke.
     *
     * @param invocation
     * @return result
     * @throws RpcException
     */
    Result invoke(Invocation invocation) throws RpcException;

}
```



```java
public class DubboInvoker<T> extends AbstractInvoker<T> {

    //与Service Provider端的连接
    private final ExchangeClient[] clients;

    //同一service的invokers集合，集群时用到。
    private final Set<Invoker<?>> invokers;

    ...

    public DubboInvoker(Class<T> serviceType, URL url, ExchangeClient[] clients, Set<Invoker<?>> invokers) {
        super(serviceType, url, new String[]{Constants.INTERFACE_KEY, Constants.GROUP_KEY, Constants.TOKEN_KEY, Constants.TIMEOUT_KEY});
        this.clients = clients;
        // get version.
        this.version = url.getParameter(Constants.VERSION_KEY, "0.0.0");
        this.invokers = invokers;
    }

    @Override
    protected Result doInvoke(final Invocation invocation) throws Throwable {
        RpcInvocation inv = (RpcInvocation) invocation;
        final String methodName = RpcUtils.getMethodName(invocation);
        inv.setAttachment(Constants.PATH_KEY, getUrl().getPath());
        inv.setAttachment(Constants.VERSION_KEY, version);

        ExchangeClient currentClient;
        if (clients.length == 1) {
            currentClient = clients[0];
        } else {
            currentClient = clients[index.getAndIncrement() % clients.length];
        }
        try {
            boolean isAsync = RpcUtils.isAsync(getUrl(), invocation);
            boolean isOneway = RpcUtils.isOneway(getUrl(), invocation);
            int timeout = getUrl().getMethodParameter(methodName, Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
            if (isOneway) {
                boolean isSent = getUrl().getMethodParameter(methodName, Constants.SENT_KEY, false);
                currentClient.send(inv, isSent);
                RpcContext.getContext().setFuture(null);
                return new RpcResult();
            } else if (isAsync) {
                ResponseFuture future = currentClient.request(inv, timeout);
                RpcContext.getContext().setFuture(new FutureAdapter<Object>(future));
                return new RpcResult();
            } else {
                RpcContext.getContext().setFuture(null);
                return (Result) currentClient.request(inv, timeout).get();
            }
        } catch (TimeoutException e) {
            throw new RpcException(RpcException.TIMEOUT_EXCEPTION, "Invoke remote method timeout. method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
        } catch (RemotingException e) {
            throw new RpcException(RpcException.NETWORK_EXCEPTION, "Failed to invoke remote method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
        }
    }

    ...
}
```



### ReferenceConfig 核心数据

```java
public class ReferenceConfig<T> extends AbstractReferenceConfig {
    //核心是DubboProtocol
    private static final Protocol refprotocol = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();

    //集群模式下使用，此处不解释
    private static final Cluster cluster = ExtensionLoader.getExtensionLoader(Cluster.class).getAdaptiveExtension();

    //注册中心地址
    private final List<URL> urls = new ArrayList<URL>();
    // interface name
    private String interfaceName;
    private Class<?> interfaceClass;
    // client type
    private String client;
    // url for peer-to-peer invocation
    private String url;
    // Service的方法列表
    private List<MethodConfig> methods;
    // default config
    private ConsumerConfig consumer;
    private String protocol;
    // invoker 的代理类
    private transient volatile T ref;
    //原生的service invoker
    private transient volatile Invoker<?> invoker;
    private transient volatile boolean initialized;
    private transient volatile boolean destroyed;
}
```



### ConsumerModel

经过ReferenceConfig一番处理后，最终会得到：Reference Dubbo Service Name, InvokerRef, Service Methods, ReferenceConfig Instance。



这些信息会封装成ConsumerModel，放到ApplicationModel.class中去全局统一记录Consumer的情况。

```java
public class ConsumerModel {
    private ReferenceConfig metadata;
    private Object proxyObject;
    private String serviceName;
	private final Map<Method, ConsumerMethodModel> methodModels = new IdentityHashMap<Method, ConsumerMethodModel>();
    ...
}
```

