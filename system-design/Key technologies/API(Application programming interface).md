
---

An interface is an contract between the engineers who implement the system and client applications who can use the system. The interface which is called by other applications is called and application programming interface or an API. 

API's can be classfied into three groups-

1. Public apis - exposed to general public
2. Private apis - Internally within any organization.
3. Partner apis - Only to partner organization.

API's can range from in process (eg java libraries) to network or web apis. 

API
 ├── Local API
 ├── Library API
 ├── OS API
 └── Web API
       ├── REST
       ├── GraphQL
       ├── gRPC
       └── SOAP

Best practices - 

1. Complete encapsulation of the internal design of the system. 
2. API should be completely decoupled from the internal design/algorithm meaning any change in the algorthim must not change the api. 
3. Keeping the operations idempotent as far as possible- Meaning the operation should not any additional / unintended effect if used more than once. 
4. Versioning the api- /api/v1/ - Using versioning we can use two versions at the same time and hence can work with incompatable changes which are introduced with future versions. 

### RPCs

The goal of RPC is deceptively simple: “I want to call a function that lives in another process or machine, but I want it to feel like a normal function call.”

So when your order service calls `paymentService.chargeCard(orderId, amount)`, it looks like a local method invocation. Under the hood, however, something very different happens. The arguments you pass to that function are converted into bytes, packaged into a message, sent across the network, received by the payment service, unpacked, and then executed as a real function on the remote side. The result of that function is then packaged again, sent back across the network, unpacked, and finally returned to your original caller as if nothing unusual happened.

To make this work, both sides must agree on a **strict contract**. This contract defines what functions exist, what parameters they take, and what they return. Without this agreement, the caller and the receiver would not understand each other. In early RPC systems this contract was defined in special interface definition languages. In modern systems like **gRPC**, the contract is written in a `.proto` file. That file becomes the single source of truth, from which client and server code are generated automatically.

A remote procedure call can be slow, can time out, can partially fail, or can succeed on the server but fail on the way back.

## Rest apis

REST stands for the representational state transfer is a set of architectural constraints and best practices for defining API's on the web. It is not a standard or a protocol. It is just an architectural style of building apis.

Rest apis abstracts everything into resources.A resource is a thing in your system that can be named and addressed a user, an order, a payment, a product. Each resource has a unique URL, and the client interacts with it using standard HTTP operations.

Two points-

1. Rest apis take resource orineted approach. The main abstraction from the user is the named resource. The resource encapsulate different entities of our system.
2. REST API allows the user to manipulate these resources through small number of methods. 

In REST api client requests one of the resources and server responds with the representation of the current state of resource. The protocol used is `http`. Note that we are not returning the exact resource rather a representation of that resource. The representation can be anything thing from an `html home` page or a `json` response. 

Two imporatant pillars of REST are - 

1. Statelessness - In a REST system, **each request must contain everything the server needs to process it**. The server does not store client-specific session state between requests. If the server receives a request, it does not assume any context from earlier requests.
2. Uniform interface for everyone. REST avoids this chaos by enforcing a **uniform interface**. This means all resources are accessed in a consistent, predictable way, using a small set of well-defined operations.

Note that REST is not just http+json its a architectural style. For example following are not REST apis

- `POST /doPayment`
- `POST /getUserData`
- `POST /updateProfile`

You’re sending commands, not manipulating resources. That is not REST — even if the response is JSON.

A RESTful system:
- Models everything as resources
- Uses standard HTTP methods consistently
- Relies on stateless requests
- Communicates meaning through status codes and headers
- Avoids hiding behavior inside action names

#### Hypermedia as the engine of Application state

HATEOAS stands for _Hypermedia As The Engine Of Application State_, which sounds terrifying, but the idea is simple:  
**the server tells the client what it can do next, at runtime, using links.**

For example say submitting an order-

```
GET /orders/123
```

Instead of returing only the orders system returns 

```json
{
  "orderId": 123,
  "status": "CREATED",
  "total": 999,
  "links": {
    "self": "/orders/123",
    "pay": "/orders/123/payment",
    "cancel": "/orders/123/cancel"
  }
}
```

So links have the next steps we can do. 
The server is saying:  
“This order exists. These are the actions that make sense _right now_.”

If the order is already paid, the response might look different:

```json
{
  "orderId": 123,
  "status": "PAID",
  "total": 999,
  "links": {
    "self": "/orders/123",
    "invoice": "/orders/123/invoice"
  }
}
```

HATEOAS solves a quiet but real problem: **clients knowing too much**.

In most “REST APIs”, clients secretly depend on undocumented rules:
- “If status is CREATED, call /pay”
- “If status is SHIPPED, don’t call /cancel”

That logic lives in client code. If the backend rules change, every client must be updated.

With HATEOAS, **the rules live on the server**, where they belong. The client becomes simpler: it renders what it sees and follows links that exist. If a link isn’t present, the action isn’t allowed.

This is exactly how the web works.  
Your browser doesn’t know every page on a website. It just follows links.

**HATEOAS means the server leads the client, not the other way around.**

### Resources

**A resource is a stable, nameable concept in your domain, not a function and not a data structure and is also called named resource**

Each resource is named and addressed using an URI for example `/orders`.Resources are organised in hierarchy using `/` meaning a certain resource can come under other for example profile and orders can come under user resource. `/user/profile`.

Each resource is either a simple resource or is a collection of resources. Note that a simple resource for example an user can have multiple subresources again profile and orders 

Collection resources on the other hand contains the list of same resource type. For example movie streaming service can have a collection resource called /movies. Again each movie can have a collection subresoure of directors etc. 

`/my-site/movies/123/directors`

Note how we are naming resources any id after the collection resource is consider a resource which can again be either simple or collection subresource. 

Resprestation of each resource can be expressed in multiple ways 
- Image
- Link to movie stream
- Object
- Html page
- Binary blob
- Executable code. 

#### Best practice on naming of resources

1. Name the resource as nouns only
2. Action to be done on any noun is named as a verb and provides a clear distinction about the resource being used or getting hurt. 
3. One must make distinction between collection resources(plural) and simple resources(singular names). 
4. No use of generic names and one must clear names for correct identification of resources. 

Now in REST once the resource is defined we can only perform predefined actions on each resource-

- Create a resource - POST
- Update the resource - PUT
- Deleting an existing resource - DELETE
- Getting the current state of resources(list of sub resources in case of collection resource) - GET

GET method is considered safe in the sense that applying it to the resource would not change the state of the resource. POST , PUT and DELETE are idempotent. To send additonal information as part of POST or PUT we can use

- json
- xml formats

Examples - 

`/movies/{movie_id}/reviews/{review_id}`
`/movies` -- collection movie resource
`/movies/{movie_id}` -- single movie resouce

Now sometimes we think about pure action basis for example running the code. But in these cases  two we can model it using the REST principles. For an instance we can model the code running as a submission resource. Now while submission we can create new submission resource `POST /submissions` and to get the status we can poll the submission as `GET /submissions/123/status` since `status` is a subresource. 

In general its usually worth answering the question- “What thing exists because of this action?”

- result - model the result as resource
- process - model the process as resource
- nothing - REST is not the good abstraction. 

Another example can be image compression where since new compressed image will be created we can go with
`/compressions/123`. Finally something which might be felt like pure action is actually the resource for example
`/movies/123/download` or `/movies/123/stream` etc. 

**REST allows verbs in URLs if they represent representations or sub-resources, not commands.**

At this time it is important to know that - 
```
/runCode
- Triggers computation
- Produces transient results
- Has no stable identity unless you create one
That difference is crucial.
```





