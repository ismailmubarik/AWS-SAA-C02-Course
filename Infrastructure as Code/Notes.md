### CloudFormation Physical & Logical Resources

![image](https://user-images.githubusercontent.com/33827177/147842247-fa129f9c-2e59-4ef9-9196-01868325d486.png)

![image](https://user-images.githubusercontent.com/33827177/147842279-45e103a4-f11b-43cd-9db2-bafe8a6b4ecc.png)

![image](https://user-images.githubusercontent.com/33827177/147842284-90e5a681-1f8c-4e95-b273-5c62991d82f5.png)

### CloudFormation Template & Pseudo Parameters

![image](https://user-images.githubusercontent.com/33827177/147885243-15da30cf-896c-4b3c-8f6d-1381bf77be76.png)

![image](https://user-images.githubusercontent.com/33827177/147885278-5fdc7277-a018-47f4-ad0c-cc81318c1e46.png)

![image](https://user-images.githubusercontent.com/33827177/147885306-bc996909-fac9-45a9-a31a-e0b22db17f1b.png)

### CloudFormation Instrinsic Functions

CloudFormation Functions lets you get access to data at runtime. This is different then passing Logical resources statically or through parameters to CloudFormation Templates.
The template can take actions based on how things are when the template is used to create a stack.

![image](https://user-images.githubusercontent.com/33827177/147886187-96936f19-c12e-4611-91be-7dd93c0c6e70.png)

![image](https://user-images.githubusercontent.com/33827177/147886257-e94cb515-b9a0-4f57-863d-f3b11643d56f.png)

![image](https://user-images.githubusercontent.com/33827177/147896131-e6c760fa-13c1-456e-af45-c711526654f1.png)

![image](https://user-images.githubusercontent.com/33827177/147896142-afebfba0-1769-423d-9d56-9494be2cd154.png)

![image](https://user-images.githubusercontent.com/33827177/147896160-3a19beaa-3934-4583-b0e6-98d351b5457b.png)

![image](https://user-images.githubusercontent.com/33827177/147896177-bf40a408-4ff5-420f-84b6-2c450dcd6ab5.png)

### CloudFormation Mappings

![image](https://user-images.githubusercontent.com/33827177/147895624-8d0c9290-6117-4ef6-a2d7-4f35ead301b5.png)

![image](https://user-images.githubusercontent.com/33827177/147895779-100e322c-e487-4e91-9b68-bb6eb1f7cab7.png)

### CloudFormation Outputs

![image](https://user-images.githubusercontent.com/33827177/147895786-3dbbf2b9-05c0-4e1c-9fc4-e17e841de2c4.png)

![image](https://user-images.githubusercontent.com/33827177/147895806-87f9b2ff-61e4-4d45-b5ba-143cc9e9be13.png)

### CloudFormation Conditions

![image](https://user-images.githubusercontent.com/33827177/147895819-6c4e8b75-3c75-4678-a64a-adc4702b45dc.png)

![image](https://user-images.githubusercontent.com/33827177/147895829-714c2bde-2c41-476c-9c12-fd0ffd75be08.png)

### CloudFormation DependsON
![image](https://user-images.githubusercontent.com/33827177/147895843-4b27c57c-05a5-47d8-9526-62dc607f97ec.png)

![image](https://user-images.githubusercontent.com/33827177/147895853-fe76104f-755f-42c8-84d6-9818ef136f1b.png)

### CloudFormation Wait Conditions & CFN Signal

![image](https://user-images.githubusercontent.com/33827177/147896267-358cb471-a15c-41ee-8c18-7758c74808fd.png)

With simple provisioning when the EC2 instance tells CloudFormation that it is in create complete state it has no further information after that? For example, the EC2 instance has been provisioned but have the Bootstrapping steps been completed? CloudFormation has no way to find the success/failure of BootStrapping...So we can be in a situation where the EC2 instance is in Create_Complete but BootStrapping was a failure.

Creation Policy, Wait Policy and CFN signals provides a way around this to provide more details to CloudFormation.

![image](https://user-images.githubusercontent.com/33827177/147896658-ec8d866a-6959-4944-8cea-b8cc77aa2414.png)

AWS suggests that for provisiong EC2 or an AutoScaling group we use a CreationPolicy because it is tied to that specific resource...But we might have other requirements. For example, if we are integrating CloudFormation with an external IT System then we might want to use Wait Condition.

![image](https://user-images.githubusercontent.com/33827177/147896868-078212b7-acd6-4c53-b08a-064b50ec40e3.png)

![image](https://user-images.githubusercontent.com/33827177/147897083-67ae65d0-8b86-4757-9b49-7d82640c8999.png)

### CloudFormation Nested Stacks

![image](https://user-images.githubusercontent.com/33827177/147897362-64c4f2d6-533b-45e3-8217-88733adad7a5.png)

A Root Stack is the Stack that gets created first. It is the first that creates Manually through the Console UI or through the command line or using some form of automation.

A parent stack is a parent of any stack that it immediately creates.

You can also have a CloudFormation Stack as a logical resource within a CloudFormation template.

![image](https://user-images.githubusercontent.com/33827177/147897856-1767fc2a-822a-4dd8-aa35-45e634cece5f.png)

We have to provide parameters for the nested stack for all those parameters that are paramatized within the (nested) stack that we are using. One exception is if the stack is using default values in which we don't have to provide the parameters.

***Note***: The Root/parent stack can only reference the Outputs from the nested/child stack not the logical resources.

![image](https://user-images.githubusercontent.com/33827177/147897914-c066a542-0ec4-44e8-a797-cec2d3ab8712.png)

### CloudFormation Cross-Stack References
In nested stacks everytime you use/re-use a template you are creating a new stack and resources. The resources are not shared just a shared/common template is being used to create new and distinct/separated resources. To share resources say a VPC we need to use Cross-Refercing within thes same region...

![image](https://user-images.githubusercontent.com/33827177/147898411-5efe7c34-9fb6-4647-898d-7623f1aab0ea.png)

Because of the separation of Stacks, Stacks cannot see the Ouptputs of other stacks. They are only visible one the command line or UI. This means you cannot use the built-on REF functions to reference one stack or output in another. The exception though is the Root Stack...but that means its a nested stack we are talking about and thus the architecture is linked in terms of life-cycle

![image](https://user-images.githubusercontent.com/33827177/147898567-0973c64c-5446-4d4a-b967-0e074cc0ff92.png)

![image](https://user-images.githubusercontent.com/33827177/147898674-aa0bff93-3f51-46f9-aad6-ae807f3580eb.png)

The import of export of references to share resources like VPC has to be within the same region. Cross region import/export is not supported

### CloudFormation StackSets
A cloud formation feature that allows you to create, delete, update across multiple regions potentially across many AWS accounts.

Service-Managed Roles: When Cloud-Formation is used in conjunction with AWS Organizations so all of the roles get created on your behalf by the product behind the scene.
So in Service-managed everything is handled by the product for you. OR you can use

Self_Managed Roles: to grant permission so that CLoudFormation can create Stack Infrastructure across many different accounts in many Regions

![image](https://user-images.githubusercontent.com/33827177/147899460-b620299d-8cdc-4c13-8877-cc650ff631c4.png)

![image](https://user-images.githubusercontent.com/33827177/147899707-cac0c656-603b-47c1-8ce6-90f4b8b8dad7.png)

### CloudFormation Deletion Policy

![image](https://user-images.githubusercontent.com/33827177/147900017-f08ab124-d72f-4703-b3e9-555548026692.png)

![image](https://user-images.githubusercontent.com/33827177/147900023-84da9258-976d-41fc-a1bb-bc7616652721.png)

### CloudFormation Roles
![image](https://user-images.githubusercontent.com/33827177/147900175-afd7a8e6-0b91-42df-a66e-5b2229b57fa7.png)

![image](https://user-images.githubusercontent.com/33827177/147900258-a38f5326-0399-4210-ad17-f9c9b2362bfa.png)




