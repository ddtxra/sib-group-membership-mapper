# sib-group-membership-mapper

Attempts to create a keycloak group membership mapper, where an array of object is created instead of an array of string

## Build the jar
```bash
mvn clean install
```

## Create the structures in keycloak instance and copy there the jar file

```bash
> mkdir -p modules/swiss/sib/keycloak/main/
> cp /tmp/sib-group-membership-mapper.jar modules/swiss/sib/keycloak/main/
> touch modules/swiss/sib/keycloak/main/module.xml
> tree modules/swiss/sib/keycloak/main/
modules/swiss/sib/keycloak/main/
├── module.xml
└── sib-group-membership-mapper.jar
```

## Content of module.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<module xmlns="urn:jboss:module:1.3" name="swiss.sib.keycloak.sib-group-membership-mapper">
    <resources>
        <resource-root path="sib-group-membership-mapper.jar"/>
    </resources>
    <dependencies>
        <module name="org.keycloak.keycloak-core"/>
        <module name="org.keycloak.keycloak-server-spi"/>
        <module name="org.keycloak.keycloak-server-spi-private"/>
    </dependencies>
</module>
```

## Enable the module
```shell
vi standalone/configuration/standalone.xml
```

```xml
<subsystem xmlns="urn:jboss:domain:keycloak-server:1.1">
    <web-context>auth</web-context>
    <providers>
        <provider>classpath:${jboss.home.dir}/providers/*</provider>
        <provider>module:swiss.sib.keycloak.sib-group-membership-mapper</provider>
    </providers>
    <master-realm-name>master</master-realm-name>

```

### First implemtentation was simply like this:
```java
public class SIBGroupMembershipMapper extends GroupMembershipMapper  {

    public static final String PROVIDER_ID = "sib-group-membership-mapper";

    @Override
    protected void setClaim(IDToken token, ProtocolMapperModel mappingModel, UserSessionModel userSession) {

        List<Map<String, String>> membership = new LinkedList<>();
        boolean fullPath = useFullPath(mappingModel);
        for (GroupModel group : userSession.getUser().getGroups()) {
            if (fullPath) {
                membership.add(new HashMap<String, String>(){{put("sib_group_name", ModelToRepresentation.buildGroupPath(group));}});
            } else {
                membership.add(new HashMap<String, String>(){{put("sib_group_name", group.getName());}});
            }
        }
        String protocolClaim = mappingModel.getConfig().get(OIDCAttributeMapperHelper.TOKEN_CLAIM_NAME);

        token.getOtherClaims().put(protocolClaim, membership);
    }


}

```

## Error produced
```bash

18:02:37,579 ERROR [org.jboss.msc.service.fail] (ServerService Thread Pool -- 48) MSC000001: Failed to start service jboss.undertow.deployment.default-server.default-host./auth: org.jboss.msc.service.StartException in service jboss.undertow.deployment.default-server.default-host./auth: java.lang.RuntimeException: RESTEASY003325: Failed to construct public org.keycloak.services.resources.KeycloakApplication(javax.servlet.ServletContext,org.jboss.resteasy.core.Dispatcher)
	at org.wildfly.extension.undertow.deployment.UndertowDeploymentService$1.run(UndertowDeploymentService.java:84)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
	at org.jboss.threads.JBossThread.run(JBossThread.java:320)
Caused by: java.lang.RuntimeException: RESTEASY003325: Failed to construct public org.keycloak.services.resources.KeycloakApplication(javax.servlet.ServletContext,org.jboss.resteasy.core.Dispatcher)
	at org.jboss.resteasy.core.ConstructorInjectorImpl.construct(ConstructorInjectorImpl.java:162)
	at org.jboss.resteasy.spi.ResteasyProviderFactory.createProviderInstance(ResteasyProviderFactory.java:2298)
	at org.jboss.resteasy.spi.ResteasyDeployment.createApplication(ResteasyDeployment.java:340)
	at org.jboss.resteasy.spi.ResteasyDeployment.start(ResteasyDeployment.java:253)
	at org.jboss.resteasy.plugins.server.servlet.ServletContainerDispatcher.init(ServletContainerDispatcher.java:120)
	at org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher.init(HttpServletDispatcher.java:36)
	at io.undertow.servlet.core.LifecyleInterceptorInvocation.proceed(LifecyleInterceptorInvocation.java:117)
	at org.wildfly.extension.undertow.security.RunAsLifecycleInterceptor.init(RunAsLifecycleInterceptor.java:78)
	at io.undertow.servlet.core.LifecyleInterceptorInvocation.proceed(LifecyleInterceptorInvocation.java:103)
	at io.undertow.servlet.core.ManagedServlet$DefaultInstanceStrategy.start(ManagedServlet.java:250)
	at io.undertow.servlet.core.ManagedServlet.createServlet(ManagedServlet.java:133)
	at io.undertow.servlet.core.DeploymentManagerImpl$2.call(DeploymentManagerImpl.java:565)
	at io.undertow.servlet.core.DeploymentManagerImpl$2.call(DeploymentManagerImpl.java:536)
	at io.undertow.servlet.core.ServletRequestContextThreadSetupAction$1.call(ServletRequestContextThreadSetupAction.java:42)
	at io.undertow.servlet.core.ContextClassLoaderSetupAction$1.call(ContextClassLoaderSetupAction.java:43)
	at org.wildfly.extension.undertow.security.SecurityContextThreadSetupAction.lambda$create$0(SecurityContextThreadSetupAction.java:105)
	at org.wildfly.extension.undertow.deployment.UndertowDeploymentInfoService$UndertowThreadSetupAction.lambda$create$0(UndertowDeploymentInfoService.java:1508)
	at org.wildfly.extension.undertow.deployment.UndertowDeploymentInfoService$UndertowThreadSetupAction.lambda$create$0(UndertowDeploymentInfoService.java:1508)
	at org.wildfly.extension.undertow.deployment.UndertowDeploymentInfoService$UndertowThreadSetupAction.lambda$create$0(UndertowDeploymentInfoService.java:1508)
	at org.wildfly.extension.undertow.deployment.UndertowDeploymentInfoService$UndertowThreadSetupAction.lambda$create$0(UndertowDeploymentInfoService.java:1508)
	at io.undertow.servlet.core.DeploymentManagerImpl.start(DeploymentManagerImpl.java:578)
	at org.wildfly.extension.undertow.deployment.UndertowDeploymentService.startContext(UndertowDeploymentService.java:100)
	at org.wildfly.extension.undertow.deployment.UndertowDeploymentService$1.run(UndertowDeploymentService.java:81)
	... 6 more
Caused by: java.lang.RuntimeException: org.jboss.modules.ModuleNotFoundException: swiss.sib.keycloak.sib-group-membership-mapper
	at org.keycloak.provider.wildfly.ModuleProviderLoaderFactory.create(ModuleProviderLoaderFactory.java:45)
	at org.keycloak.provider.ProviderManager.<init>(ProviderManager.java:62)
	at org.keycloak.services.DefaultKeycloakSessionFactory.init(DefaultKeycloakSessionFactory.java:76)
	at org.keycloak.services.resources.KeycloakApplication.createSessionFactory(KeycloakApplication.java:326)
	at org.keycloak.services.resources.KeycloakApplication.<init>(KeycloakApplication.java:117)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at org.jboss.resteasy.core.ConstructorInjectorImpl.construct(ConstructorInjectorImpl.java:150)
	... 28 more
Caused by: org.jboss.modules.ModuleNotFoundException: swiss.sib.keycloak.sib-group-membership-mapper
	at org.jboss.modules.ModuleLoader.loadModule(ModuleLoader.java:285)
	at org.jboss.modules.ModuleLoader.loadModule(ModuleLoader.java:271)
	at org.keycloak.provider.wildfly.ModuleProviderLoaderFactory.create(ModuleProviderLoaderFactory.java:41)
	... 37 more

18:02:37,586 INFO  [org.jboss.as.server] (Thread-2) WFLYSRV0220: Server shutdown has been requested via an OS signal
18:02:37,592 ERROR [org.jboss.as.controller.management-operation] (Controller Boot Thread) WFLYCTL0013: Operation ("add") failed - address: ([("deployment" => "keycloak-server.war")]) - failure description: {"WFLYCTL0080: Failed services" => {"jboss.undertow.deployment.default-server.default-host./auth" => "java.lang.RuntimeException: RESTEASY003325: Failed to construct public org.keycloak.services.resources.KeycloakApplication(javax.servlet.ServletContext,org.jboss.resteasy.core.Dispatcher)
    Caused by: java.lang.RuntimeException: RESTEASY003325: Failed to construct public org.keycloak.services.resources.KeycloakApplication(javax.servlet.ServletContext,org.jboss.resteasy.core.Dispatcher)
    Caused by: java.lang.RuntimeException: org.jboss.modules.ModuleNotFoundException: swiss.sib.keycloak.sib-group-membership-mapper
    Caused by: org.jboss.modules.ModuleNotFoundException: swiss.sib.keycloak.sib-group-membership-mapper"}}
```
