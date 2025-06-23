1. Loading NGINX Golden Path into backstage portal catalog and Humanitec for Developer self-service

 I created a backstage folder which contained the things needed  by backstage for humanitec, this would feed developers using backstage as portal:
    content folder
    template.yaml

2. Registered it as entity in the backstage catalog . this can be verified by clicking create in backstage portal UI and we can see the list of all existing golden paths including out new golden path just added nginx

For now its not usable until we add the needed resources which are needed in our score.yaml to our enviroment. we can see that when we try to run it 

3. Next task as Platform Engineer: Register the following needed resources using resource definitions for the workload to be deployed and provisioned sucessfully when called upon by our developers:

volume, 
horizontal-pod-autoscaler 
workload with securityContext

So i created the res-def folder, with the following files in it:

volume-emptydir.yaml (template driver)
hpa.yaml (hpa driver)
custom-workload-with-security-context.yaml (template driver)

i like renaming my resource definitions , so i will add my name to each.

4. Next tasks: 
 - create the resource class for each resource from our terminal
 - create the resource definitions using humctl create (since its template drivers not terraform drivers)
 - create matching criteria to make sure we can use this resource definition inside our 5min-idp environment type and using the resource class we just created

 If no class on the score.yaml for a resource, just create the resource definition, no need for resource class and matching criteria ( u may need to match it to environment though)


a. resource class for volume
    volume-emptydir: injects an emptyDir volume  into a Workload for any request of a volume resource with the class ephemeral 

humctl api post "/orgs/${HUMANITEC_ORG}/resources/types/volume/classes" \
  -d '{
  "id": "ephemeral",
  "description": "Specifically to be used when using ephemeral IDP settings"
}'

or class default

b. resource class for hpa
    horizontal-pod-autoscaler: injects horizontal-pod-autoscaler  into a Workload for any request of a horizontal-pod-autoscaler resource with the class 5min

humctl api post "/orgs/${HUMANITEC_ORG}/resources/types/horizontal-pod-autoscaler/classes" \
  -d '{
  "id": "5min",
  "description": "Specifically to be used when using 5min IDP settings"
}'

c. resource class for workload
    
No resource class for workload resource types because they are implicit resource types and not direct resource types. to understand better read my blog post on humanitec resource types(direct, indirect, implicit)

5. next task: Create resource definitions for each of our resources.

before we start run this to see the list of existing resource definitions we have: 

humctl get resource-definitions

Now we create our resource definitions using:

humctl create -f resource_definition_file.yaml

cd res-defs

humctl create -f volume-emptydir.yaml

humctl create -f hpa.yaml

humctl create -f custom-workload-with-security-context.yaml

6. setting up matching criterias for resources to classes

a. volume
      humctl api PUT /orgs/${HUMANITEC_ORG}/resources/defs/ikeme-volume-emptydir/criteria --data '[{"env_type": "5min-local","class":"default"}]'

b. hpa
    humctl api PUT /orgs/${HUMANITEC_ORG}/resources/defs/ikeme-hpa/criteria --data '[{"env_type": "5min-local","class":"default"}]'

finally as developer

we can now head over to backstage portal, select templates in categories, or click create to launch  our newly resgistered entity to the catalog.

and test our workload deployment

summary

we start by observing what we need and creating a score file, from the resources section we see what resources would be requested for by the workload.

Now create the resource definitions for the workload resources and resource classes depending on use case

apply matching criteria if needed

create backstage template

then go register our it  on backstage

test deployment

view on humanitec and gitea and backstage to see if all successful

confirm deployment on terminal using: 

humctl

