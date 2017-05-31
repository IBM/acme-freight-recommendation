Build smarter apps faster with serverless and APIs
---

Acme Freight is a sophisticated logistics company that is able to respond to weather fluctuations by rerouting shipping paths or choosing a different delivery truck to execute the shipment. They can do this because of the intelligent recommendations application built into their architecture using an event-based model called serverless.

### Brief overview

Serverless is centered around responding to events. The programming model is based on 4 concepts: packages, triggers, actions, and rules. Packages provide event feeds. Triggers associated with those feeds fire when an event occurs, and developers can map actions — or functions — to triggers using rules.

![OpenWhisk flow](ow-flow.gif)

There are a few serverless solutions on the market today, such as Google Functions and Amazon Lambda. Acme Freight uses OpenWhisk, which IBM created and open sourced through the Apache Foundation. IBM also added a native API experience to OpenWhisk on Bluemix that made integrating recommendations into our Acme Freight application easy and straightforward.

With Acme Freight, we chose to implement just the recommendations aspect using serverless, but there are companies that have moved their entire web architecture to serverless. The benefits are very easy to see and many organizations are embracing serverless. Serverless is also great for things beyond web applications, such as data processing, IoT, chatbots, and batch jobs.


### Acme Freight + serverless

Acme Freight uses a serverless approach to respond to weather events. An action is triggered by a storm event; retailers affected by the weather event are identified; their stock and shipments are retrieved; recommendations are made based on the storm, stock and shipments. While the guts of the code are abstracted out of this flow into their own function blocks, you can see the process in the following code from the [recommend action](https://github.com/IBM/acme-freight-recommendation/blob/dev/actions/recommend.js#L46):

**`actions/recommend.js`**

```javascript
    return new Promise((resolve, reject) => {
      async.waterfall([

        // delete existing recommendations for this demo
        function(callback) {
          self.cleanup(
            args['services.cloudant.url'],
            args['services.cloudant.database'],
            args.demoGuid,
            callback);
        },

        // retrieve list of retailers
        function(callback) {
          self.getRetailers(args['services.controller.url'],
            args.demoGuid, callback);
        },

        // identify retailers affected by the weather event
        function(retailers, callback) {
          self.filterRetailers(retailers, args.event, callback);
        },

        // retrieve their stock and make new shipments
        function(retailers, callback) {
          self.recommend(retailers, callback);
        },

        // persist the recommendations
        function(recommendations, callback) {
          self.persist(
            args['services.cloudant.url'],
            args['services.cloudant.database'],
            args.demoGuid,
            recommendations,
            callback);
        }
      ], (err, result) => {
        if (err) {
          reject({ ok: false });
        }
        else {
          resolve({
            demoGuid: args.demoGuid,
            event: args.event,
            recommendations: result,
          });
        }
      });
    });
```

The code above has comments inline to describe the flow of the action

1. delete existing recommendations for this demo
1. retrieve list of retailers
1. identify retailers affected by the weather event
1. retrieve their stock and make new shipments
1. persist the recommendations

And the last code block has an if/else that either rejects the promise **if** there is an error or **else** it resolves the promise with an object of recommendation information.

### OpenWhisk Native APIs

If you look at the [actions in the repo](https://github.com/IBM/acme-freight-recommendation/tree/dev/actions), you'll see the functional code is broken into logical separations of concern. With these actions deployed to Bluemix, we are able to take advantage of the new native API experience for OpenWhisk and make the recommendations interface available to our application via APIs. This is a new feature of Bluemix and is rolling out across a number of services and applications. Through the API pane of the OpenWhisk web UI, we can manage a number of aspects of our API layer on top of our OpenWhisk functions, including:

- provide a clear interface to our recommendations functionality
- simplify authorization details to just an API key from the calling code
- add API management, if desired, such as rate limiting, tokenization and more

Making our OpenWhisk interface available as an API keeps the seams between the Acme Freight application and the recommendations engine cleanly defined. Separating the recommendations logic from the Acme Freight application cuts our costs, removes deployment dependencies and frees up our developers to focus on the core application needs.

Add this video here? https://www.youtube.com/watch?v=vxk9OgNFfD0

Learn more about [serverless](https://developer.ibm.com/openwhisk/what-is-serverless-computing/)<br>
Learn more about [IBM OpenWhisk](https://developer.ibm.com/openwhisk/)<br>
Learn more about [Native APIs in Bluemix](https://developer.ibm.com/apiconnect/2017/04/28/introducing-bluemix-native-experience-openwhisk/)

Thanks to [Daniel Krook](https://twitter.com/DanielKrook) for the graphic above and his input on this article
