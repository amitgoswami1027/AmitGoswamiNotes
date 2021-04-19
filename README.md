# SYSTEM DESIGN
## Software Architecture Tiers
Think of a tier as a logical separation of components in an application or a service. And when I say separation, I mean physical separation at the component level, not the code level. 
What do I mean by components?
* Database
* Backend application server
* User interface
* Messaging
* Caching

![image](https://user-images.githubusercontent.com/13011167/115166957-4d476400-a0d3-11eb-8968-0560c758ff75.png)

### Single Tier Applications: A single-tier application is an application where the user interface, backend business logic & the database all reside in the same machine.
Typical examples of single-tier applications are desktop applications like MS Office, PC Games or an image editing software like Gimp.

* Advantage: The main upside of single-tier applications is they have no network latency since every component is located on the same machine. This adds up to the performance of the software. In single-tier apps, the data is easily & quickly available since it is located in the same machine.Though it largely depends on how powerful the machine is & the hardware requirements of the software, to gauge the real performance of a single-tier app. The data of the user stays in his machine & doesn’t need to be transmitted over a network. This ensures data safety at the highest level.
* Disadvantage: One big downside of single-tier app is that the business has no control over the application. The code in single-tier applications is also vulnerable to being tweaked & reversed engineered. The security, for the business, is minimal. The applications’ performance & the look and feel can get inconsistent as it largely depends on the configuration of the user’s machine.

### Two Tier Applications: A Two-tier application involves a client and a server. The client would contain the user interface & the business logic in one machine. And the backend server would be the database running on a different machine. The database server is hosted by the business & has control over it.

* Need of Two Tier Application : Good example of this is the online browser & app-based games or  to-do list app or a similar planner or a productivity app. The game files are pretty heavy, they get downloaded on the client just once when the user uses the application for the first time. Moreover, they make the network calls only to keep the game state persistent. Also, fewer server calls mean less money to be spent on the servers which is naturally economical.

### Three Tier Applications: In a three-tier application, the user interface, application logic & the database all lie on different machines & thus have different tiers. They are physically separated. So, if we take the example of a simple blog, the user interface would be written using Html, JavaScript, CSS, the backend application logic would run on a server like Apache & the database would be MySQL. A three-tier architecture works best for simple use cases.

![image](https://user-images.githubusercontent.com/13011167/115167314-cabfa400-a0d4-11eb-87ab-dd02bba55d72.png)

### N Tier Applications: An N-tier application is an application which has more than three components involved. 
What are those components?
* Cache
* Message queues for asynchronous behaviour
* Load balancers
* Search servers for searching through massive amounts of data
* Components involved in processing massive amounts of data
* Components running heterogeneous tech commonly known as web services etc.

* Why The Need For So Many Tiers: Two software design principles that are key to explaining this are the Single Responsibility Principle & the Separation of Concerns.
Single Responsibility Principle simply means giving one, just one responsibility to a component & letting it execute it with perfection. Be it saving data, running the application logic or ensuring the delivery of the messages throughout the system.This approach gives us a lot of flexibility & makes management easier.
Separation of concerns kind of means the same thing, be concerned about your work only & stop worrying about the rest of the stuff. These principles act at all the levels of the service, be it at the tier level or the code level.

### Difference Between Layers & Tiers - on’t confuse tiers with the layers of the application. Some prefer to use them interchangeably. But in the industry layers of an application typically means the user interface layer, business layer, service layer, or the data access layer. The layers mentioned in the illustration are at the code level. The difference between layers and tiers is that the layers represent the organization of the code and breaking it into components. Whereas, tiers involve physical separation of components. All these layers together can be used in any tiered application. Be it single, two, three or N-tier.
## System Design Readings

* [MultiTenant Design] https://success.outsystems.com/Support/Enterprise_Customers/Maintenance_and_Operations/Designing_the_Architecture_of_Your_OutSystems_Applications/Designing_Scalable_Multi-Tenant_Applications
* [MultiTenant Design] https://docs.microsoft.com/en-us/azure/azure-sql/database/saas-tenancy-app-design-patterns
* [MultiTenant Design] https://github.com/uglide/azure-content/blob/master/articles/sql-database/sql-database-design-patterns-multi-tenancy-saas-applications.md
* [MultiTenant Design] https://engineering.opsgenie.com/how-to-design-a-modern-multi-tenant-saas-application-with-auth0-45c446e825b7
