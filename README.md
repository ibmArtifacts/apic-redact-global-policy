# apic-redact-global-policy
Sample APIC redact global policy  

This global policy will be obfuscating values for ssn and creditcard.  
The final result will look like the following:  

![image](https://github.com/user-attachments/assets/70bbdfbc-5fad-4aee-bbf4-5b48034dc1ee)  

## Design the API logic  
First, build the logic out on an intermediary API to copy to the global policy file later.  
![image](https://github.com/user-attachments/assets/05a8c459-88f3-4b5a-b358-df61a141f8e7)  

The diagram above shows a redact API to test. 
The api can be downloaded [here](https://github.com/ibmArtifacts/apic-redact-global-policy/blob/main/redact-api.yaml).  
The logic is to  
1. evaluate the content-type header to detemine whether the payload is xml or json (the requests must have proper content-type headers),  
2. parse the body, and  
3. redact value of fields that contain ssn or creditcard.  

Once the logic is set, the assembly logic can be copied over to the global policy file.    

## Develop the global policy  
Take the code after assembly.execute from the intermediary redact API and copy it under the assembly.execute section of the global policy.  
![image](https://github.com/user-attachments/assets/97c1ea67-fab0-4c3b-a42c-6b8da74f66f3)  

The final global policy should like the following [redact-global-policy.yaml](https://github.com/ibmArtifacts/apic-redact-global-policy/blob/main/redact-global-policy.yaml) file.  

## Deploy to APIC  
The following are the APIC toolkit commmands to deploy the global policy.  
If you are on Windows, it's best to run on gitbash.  

### Set the environment variables  
```
export APIC=apic
export GLOBAL_POLICY_FILE=refact-global-policy.yaml
export NAME_OF_POLICY=redact
export VERSION_OF_POLICY=1.0.0
export APIC_SERVER=APIM_URL_HERE
export REALM=provider/default-idp-2
export USERNAME=USERNAME_HERE
export PASS=PASSWORD_HERE
export CATALOG=CATALOG_NAME_HERE
export GWY_NAME=GATEWAY_NAME_HERE
export PORG=PROVIDER_ORGANIZATION_NAME_HERE
export GWY_URL=INVOCATION_URL_HERE
```

### Login  
`${APIC} login -s ${APIC_SERVER} --realm ${REALM} --username ${USERNAME} --password ${PASS}`


### Set pre-flow global policy  
```
${APIC} global-policies:create --server ${APIC_SERVER} --configured-gateway-service ${GWY_NAME} --org ${PORG}  --scope catalog ${GLOBAL_POLICY_FILE}

${APIC} global-policies:get --configured-gateway-service ${GWY_NAME} --org ${PORG} --server ${APIC_SERVER} --scope catalog ${NAME_OF_POLICY}:${VERSION_OF_POLICY} --fields url

sed -i 's|url|global_policy_url|' GlobalPolicy.yaml

${APIC} global-policy-prehooks:create --configured-gateway-service ${GWY_NAME} --org ${PORG} --server ${APIC_SERVER} --scope catalog GlobalPolicy.yaml
```  
## Removing and cleaning up  
```
${APIC} global-policy-prehooks:delete --configured-gateway-service ${GWY_NAME} --org ${PORG} --server ${APIC_SERVER} --scope catalog

${APIC} global-policies:delete --configured-gateway-service ${GWY_NAME} --org ${PORG} --server ${APIC_SERVER} ${NAME_OF_POLICY}:${VERSION_OF_POLICY} --scope catalog

rm GlobalPolicy.yaml
```

## Testing
The [echo-api.yaml](https://github.com/ibmArtifacts/apic-redact-global-policy/blob/main/echo-api.yaml) can be used as a test.
Once published, you should be able to test it with the following and see the results:  
`$ curl -X POST -k https://${GWY_URL}/${PORG}/${CATALOG}/echo/ -H "content-type: application/json" -d '{"ssn":"123456789", "creditcard":"12324454655543", "Name":"Will"}'`

Response:  
`{"ssn":"*******","creditcard":"*******","Name":"Will"}`


# References  
All the steps taken were instructions from the IBM APIC documentation: [Working with Global Policies](https://www.ibm.com/docs/en/api-connect/10.0.8?topic=applications-working-global-policies) and [Customizing the preflow policy](https://www.ibm.com/docs/en/api-connect/10.0.8?topic=policies-customizing-preflow).





