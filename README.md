# Devhub Meetup Notes: How to build your own data acquisition framework in Typescript

## Data pipeline:
Events layer (event production, between user and system, click on a button, etc...) ->  
Data streaming layer (kafka, pub-sub system, there is a producer, there are consumers, uses multiple topics) ->  
Data extraction layer (consumer of kafka, which later puts data in specified places) ->  
Storage layer (any storage system like hadoop, google cloud storage) ->  
Data delivery (presto db for distributed query system) ->  
UI layer (e.g. Mode, Jupyter, data scientist interactions start here)

## Philosophy of data collection
collect everything
- tons of data
- never miss out on anything

vs

collect intentionally
- only capture what you intend

## Data acquisition layer
shopify built a Monorail which ensures a contract between producer of events and consumer of data  
Monorail is doing that by using schemas  
Schema might define click on a button event which has information about a click as well as user_id  
Format might look as following (except in real life it is in yaml format, not json)  
```js
{
  name, doc: 'Click schema for shopify_admin', verson, format: 'json', 
  fields: [{
    name, type, optional: bool, doc, privacy_handler /*for user privacy policy, used to scrub personal data - for example for GDPR when user requests to delete all data*/
  }]
} 
```

Advantages
- data s attaches contexts, which makes it easy to understand data
- data consistency
- registry of all metrics
- out of the box privacy handling
- versioning of events
- bad data is caught early

Shopify built Monorail for multiple languages - ruby, php, js, python

Pipeline looks like this:  
Client -> api call to Monorail -> kafka layer -> other layers mentioned in data pipeline above

In order to make sure that data being sent has correct types Monorail is using Typescript

Data scientist specifies schemas, commits them to a repo, where they are automatically transformed into typescript interfaces.

Function that uses interfaces is looking like following Monorail.produce('shopify_admin_click/1.0', user_id, ...)

Monorail takes advantage of discriminated unions (type Shape = Circle | Rectangle), so for Monorail.produce interface looks like this type MonorailType = AdminClickTypeGeneratedFromSchema | SomeOtherTypeGeneratedFromSchema | ... all interfaces from schemas

So interfaces are 'united' into MonorailType custom type which later is used to verify that event data is in correct format at the time when code is written.
