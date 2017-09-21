# Distributed HTTP Gateway

The HTTP (v. 1.1/2.0) gateway translates requests from HTTP into 
internal Distributed Protocol Messages which then are encoded  
using messagepack and sent over the wire to the responsible service
using zeroMQ.

The gateway does its routing to hosts where the services are
running using DNS. Each application runs under a hostname 
under which each service can register using subdomains. Which 
service to route messages to is determined by the first URL part. 

For example, a simple application consisting of a user service 
may have the following URL & DNS configurations.

Server A, HTTP Gateway, Public DNS Name: `api.example.com`
Server B, User Service, Private DNS Name: `user-service.myapp.private`

An outside user can now request a user with the id `1` from the 
user service by sending a GET request to the URL 
`api.example.com/user-service/user/1` which is translated by the 
HTTP gateway to a service: `user`, a resource `user` and a 
resourceId: `1`.

The gateway now checks in the private DNS if there is an entry 
for the host `user-service.myapp.private`. If it finds the DNS name,
it creates a connection to that host and sends the message there.

Now the user service takes over and routes the message to the 
appropriate controller for the resource `user` which then 
processes the message and sends a response back to the gateway.


If you are looking for websocket support you should have a look at
the distributed-ws-gateway, which lets you directly send distributed
messages without the need to use HTTP.


## Request Format

Each resource may be filtered, ordered and paginated. Fields to 
return may be selected.


filter: eventData.venueFloor.name=not(null)
order: id asc, eventData.venueFloor.id desc
range: 0-10, eventData 0-100 

{
    resource: 'event'
    filter: [{
        type: 'or',
        nodes: [{
            type: 'resource',
            value: 'eventData',
            nodes: [{
                type: 'resource',
                value: 'venueFloor',
                nodes: [{
                    type: 'property',
                    value: 'name',
                    nodes: [{
                        type: 'comparison',
                        value: 'not',
                        nodes: [{
                            type: 'value',
                            value: null
                        }]
                    }]
                }]
            }]
        }]
    }]
    selection: []
    order: {property: 'id', direction: 'ascending'},
    offset: 0,
    limit: 100,
    include: [{
        resource: 'eventData'
    }]
}





```javascript

    const events = await Q.build().event('id', 'name', 
        Q.filter().id(Q.in(1,2,3).or.equal(78)), 
        Q.order(id).desc(),
        Q.offset(range).limit(100),
    ).eventData('id').find();
```


{
    data: [],
    range: 0-9,
    hasNext: true,
    hasPrevious: false,
}