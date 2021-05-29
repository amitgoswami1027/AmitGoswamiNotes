
# AWS SHARED RESPONSIBILITY MODEL
![image](https://user-images.githubusercontent.com/13011167/118427396-b4fcc900-b6ea-11eb-8959-4c623acbe2d2.png)

# AMAZON IAM, STS, FEDERATION AND SINGLE SIGN-ON
## AWS:IAM:An introduction
* The "identity" aspect of AWS Identity and Access Management (IAM) helps us by asking a question "Who is that user?", often referred to as authentication.

* If IAM is the bouncer then IAM Roles, Users and Groups are the different lists the bouncer checks to see who is allowed in, the A-list crowd. And if your name is on said list, ie you have permission then think of that as a valid Policy — are you cool enough to enter this particular party? OK, so far this analogy is holding up, great!
* Roles are a collection of one or more Policies (permissions) that combined together, list out what the principal can or can’t do to other resources.
* We have Policy, Action, Resource, Effect and Principal. Principal>Action>Resource.
what can access them (Resource based). Once you understand that, policies become super simple. Great!
* You can have up to 10 Policies attached to an IAM Role or an IAM User. This is a soft limit on AWS, however, you can ask for this to be increased and if you ask nicely it goes up to 20 per Role or User. Even though it is an auto-approved no hassle increase, you have to ask as 10 is the default
* The Policy breakdown:
  * An IAM identity is a User, Role or Group
  * Policies can be attached to IAM Identities
  * Policies are a list of permissions and can be found in JSON format
  * Policies use the Effect + PARC attributes
* Identity based policies don’t have a Principal attribute. That’s because they assume that the Resource the IAM Role is attached to IS the Principal, the actor. Makes sense, right? That they’re called an IDENTITY policy, you are therefore specifying the identity of the Resource. What the Resource can do.
* Note: A key point to remember is that policies attached to a resource define its permissions. Whether that’s specifying what they can access (Identity based) or 

  ![image](https://user-images.githubusercontent.com/13011167/106695198-1e574100-6600-11eb-8fda-5fc6e81485aa.png)
  ![image](https://user-images.githubusercontent.com/13011167/106695544-d4228f80-6600-11eb-82a9-d6e3f2226534.png)
  ![image](https://user-images.githubusercontent.com/13011167/106695619-f5837b80-6600-11eb-8137-c1d486a5385e.png)
  ![image](https://user-images.githubusercontent.com/13011167/106696074-e8b35780-6601-11eb-87d9-d76dd379f111.png)
  ![image](https://user-images.githubusercontent.com/13011167/106694042-c28bb880-65fd-11eb-9f05-4bdd53fb1e7d.png)
  ![image](https://user-images.githubusercontent.com/13011167/106696756-5d3ac600-6603-11eb-9b4f-9fb5ad1bdd2f.png)


# IDENTITY FEDERATION
* (Identity Users): Creation of the trusted relationship between External Identity Provider and AWS.

![image](https://user-images.githubusercontent.com/13011167/118425404-c7750380-b6e6-11eb-9317-e90ef82c80ed.png)

### STS (Security Token Service) ia a web service that enables you to request temporary, limited privilage credentials for AWS Identity and access management user or user that we want to autnenticate (Federated Users). Temporary credentials are useful in the scenarios that involve identity federation,delegation, cross account access and IAM roles.STS APIs : 
* Assumerole(IAM User) 
* AssumeRoleWithSAML(Any USer)/Known Identiy provider
* AssumeRoleWithWebIdentity(Any USer)/Known Identiy provider/
* GetSessionToken (IAM User) - Does not support passing the policies
* GetFederationsToken(IAM User)

### AssumeRoleWithWebIdentity API - Any User can call. Caller must have web identity token that indicates authentication from a known identity provider.
![image](https://user-images.githubusercontent.com/13011167/120058400-39930400-c068-11eb-908f-799cd85ad628.png)

# SAML 2.0 - Security Asseration Markup Language
* Open Standards for exchanging the authentication & Authorization informaiton data between the security domains.
* XML based protocol that uses security Tokens containing asserations to pass informations about a principal (using end user) between SAML Authority, name as identity provider ,SAML customer and a named service provider.

![image](https://user-images.githubusercontent.com/13011167/118426751-7581ad00-b6e9-11eb-924f-9583716e1af7.png)

# IDENTITY FEDERATION USING ENTERPRSE FEDERATION  SAML 2.0
![image](https://user-images.githubusercontent.com/13011167/120058674-47498900-c06a-11eb-9549-fa70b7e71547.png)

# SSO USING SAML & STS GetFederationToken APIs
* User Browser recieves the SAML asseration in the form of an authentication response from ADFS.
* Broeser post the SAML asseration to the AWS sing-in end point for SAML (https://signin.aws.amazon.com/saml/)
* Behind the scenes, sign-in uses the AssumeRoleWithSAML API to request temporary security credentials and then construct the sign-in URL for AWS Management console.
* Browser will recieve the sign in URL and is redirected to the console.
![image](https://user-images.githubusercontent.com/13011167/120059068-a7412f00-c06c-11eb-8d31-c5b111ae4227.png)















### Important Links
* https://hamzahabdulla1.medium.com/introduction-to-aws-iam-282bceaa533c
* [Federatation using MSAD]https://aws.amazon.com/blogs/security/aws-federated-authentication-with-active-directory-federation-services-ad-fs/
* [ SAML 2.0 FEDERATION] : https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_saml.html
