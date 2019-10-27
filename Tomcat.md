## 1 Tomcat的系统架构

https://www.cnblogs.com/softidea/p/5384395.html


### 1.1 Server

首先Server是管理Service接口的，Server是Tomcat的顶级容器，它是一个接口。它的标准实现类是StandardServer类。
```

/**
 * Standard implementation of the <b>Server</b> interface, available for use
 * (but not required) when deploying and starting Catalina.
 *
 * @author Craig R. McClanahan
 */
public final class StandardServer extends LifecycleMBeanBase implements Server {

    private static final Log log = LogFactory.getLog(StandardServer.class);


    // ------------------------------------------------------------ Constructor


    /**
     * Construct a default instance of this class.
     */
    public StandardServer() {

        super();

        globalNamingResources = new NamingResourcesImpl();
        globalNamingResources.setContainer(this);

        if (isUseNaming()) {
            namingContextListener = new NamingContextListener();
            addLifecycleListener(namingContextListener);
        } else {
            namingContextListener = null;
        }

    }
// --------------------------------------------------------- Server Methods


    /**
     * Add a new Service to the set of defined Services.
     *
     * @param service The Service to be added
     */
    @Override
    public void addService(Service service) {

        service.setServer(this);

        synchronized (servicesLock) {
            Service results[] = new Service[services.length + 1];
            System.arraycopy(services, 0, results, 0, services.length);
            results[services.length] = service;
            services = results;

            if (getState().isAvailable()) {
                try {
                    service.start();
                } catch (LifecycleException e) {
                    // Ignore
                }
            }

            // Report this property change to interested listeners
            support.firePropertyChange("service", null, service);
        }

    }
    
    /**
     * @return the specified Service (if it exists); otherwise return
     * <code>null</code>.
     *
     * @param name Name of the Service to be returned
     */
    @Override
    public Service findService(String name) {
        if (name == null) {
            return null;
        }
        synchronized (servicesLock) {
            for (int i = 0; i < services.length; i++) {
                if (name.equals(services[i].getName())) {
                    return services[i];
                }
            }
        }
        return null;
    }

```
我们重点关注Service的两个方法addService和findService。 从addService方法中可以看出Service通过*数组*来管理的，每新增一个service的话，就把原来的
serivce 拷贝到一个新数组，再把新的service放入数组中。而Server和Service是关联的 ，当判断getState().isAvailable()状态是否有效，来决定是否执行service方法。这里说的状态，就是Tomcat管理各组件生命周期的lifecycle接口。

Lifecycle接口

