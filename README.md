thrift-client-pool-java
=======================

A Thrift Client pool for Java

* [x] raw and TypeSafe TServiceClient pool
* [x] Multi Backend Servers support
* [x] Backend Servers replace on the fly
* [ ] Backend route by hash or any other algorithm
* [x] java.io.Closeable resources (for try with resources)
* [x] Ease of use
* [x] jdk 1.8 only (1.7 is not okay without code modification)

## Usage

Install to your maven repo by

```Shell
mvn clean package install
```

or

```Shell
mvn clean package deploy
```

Add to your pom.xml (1.0 formal version will release when we running long enough in real production environment.)

```xml
<dependency>
    <groupId>com.wealoha</groupId>
    <artifactId>thrift-client-pool-java</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

```Java
// get a pool
PoolConfig config = new PoolConfig();
config.setFailover(true); // optional
config.setTimeout(1000); // optional
// PoolConfig is a instance of GenericObjectPoolConfig
config.setMinIdle(3);
config.setMaxTotal(30);
ThriftClientPool<TestThriftService.Client> pool = new ThriftClientPool<>(
    serverList,
    e -> new YourThriftService.Client(new TFramedTransport(new TBinaryProtocol(e))),  // ❶ 
    config);

// or pre jdk1.8
ThriftClientPool<TestThriftService.Client> pool = new ThriftClientPool<>(serverList,
    new ThriftClientFactory() { // ❶
        
        @Override
        public TServiceClient createClient(TTransport transport) {
            return new YourThriftService.Client(new TFramedTransport(new TBinaryProtocol(transport)));
        }
    }, config);

// just call thrift
try {
    Iface iFace = pool.iface(); // ❷
    String response = iFace.echo("Hello!"); // ❸
    logger.info("get response: {}", response);
} catch (Throwable e) {
    logger.error("call echo fail", e);
}

// or call like this
try (ThriftClient<Client> thriftClient = pool.getClient()) { // ❹
    // get Iface generated by Thrift
    Iface iFace = thriftClient.iFace(); // ❺
    
    String response = iFace.echo("Hello!");
    logger.info("get response: {}", response);
    response = iFace.echo("Hello again!");
    logger.info("get response: {}", response);
    
    // finish must be called at last
    thriftClient.finish(); // ❻ 
} catch (TException e) {
    logger.error("call echo fail", e);
}
```

* ❶ return your service Client(an IFace impl and TServiceClient subclass) generated by Thrift
* ❷ obtain it from pool
* ❸ call your service
* ❹ another way is getting a wrapped client using try with resources
* ❺ get Iface from wrapped client
* ❻ when using getClient(), finish must be called if non Exception throw in interacting 

## ThriftClientFactory

Only one interface need to impl with few lines, return one YourThriftService.Client in createClient(TTransport)
generated by thrift tools. Pool will handle all the rest.

## ThriftClientPool

* void setServices(List<ServiceInfo>);

Dynamically change backend services, all new client get from getClient() will using new services.

## Know issues

If you encourage this exception, please check that Iface match Pool's Client type.

```Java
java.lang.ClassCastException: com.sun.proxy.$Proxy3 cannot be cast to xx.xx.Iface
```

## See also

[A better pure connection pool with shard](https://github.com/PhantomThief/thrift-pool-client)


