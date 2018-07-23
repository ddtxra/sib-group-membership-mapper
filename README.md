# sib-group-membership-mapper

Attempts to create a keycloak mapper

First implemtentation was simply like this:



```bash
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
