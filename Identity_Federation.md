# AMAZON IAM, STS, FEDERATION AND SINGLE SIGN-ON

### AWS:IAM:An introduction
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
   







### Important Links
* https://hamzahabdulla1.medium.com/introduction-to-aws-iam-282bceaa533c
