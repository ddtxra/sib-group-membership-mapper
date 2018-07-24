# sib-group-membership-mapper

Custom Keycloak Protocol Mapper for group membership.

Changes the default keycloak implementation where an array of object is created instead of an array of strings for groups.

## Build the jar
```bash
mvn clean install
scp target/sib-group-membership-mapper.jar username@remote-keycloak-server:/tmp
```

### Install the jar as a module using the jboss script
 
```bash
./bin/jboss-cli.sh --command="module add --name=swiss.sib.keycloak.sib-group-membership-mapper --resources=/tmp/sib-group-membership-mapper.jar --dependencies=org.keycloak.keycloak-core,org.keycloak.keycloak-server-spi,org.keycloak.keycloak-server-spi-private,org.keycloak.keycloak-services"
```

### Or create the structure manually

```bash
> mkdir -p modules/swiss/sib/keycloak/main/
> cp /tmp/sib-group-membership-mapper.jar modules/swiss/sib/keycloak/main/
> touch modules/swiss/sib/keycloak/main/module.xml
> tree modules/swiss/sib/
  modules/swiss/sib/
  └── keycloak
      └── sib-group-membership-mapper
          └── main
              ├── module.xml
              └── sib-group-membership-mapper.jar

```

### Content of module.xml
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
        <module name="org.keycloak.keycloak-services"/>
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

## TODO 
Attempt to extend the default GroupMembershipMapper and check why it does not work
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
