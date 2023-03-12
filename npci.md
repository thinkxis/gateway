building payments gateway from scratch using npci stack 
# Author: Dipesh Bhoir
In our case, we’re going to build a simple API Gateway for the entry of our microservice. So, the API Gateway will implement all of these functions below:
- Basic Authentication
- Monitoring
- Response Header Modification
- Request forwarding

here is a thing...
i'm chosing stack based on belief that you own your servers insted of lean model. Though in todays world that is not the case with most companies.
I'd prefer complete GCP based stack in that case replacing MongoDB with Firestore & Express with Cloud Functions.
Otherwise its preety basic concept of in four stages.


```
"dependencies": {
    "axios": "^0.19.1", // Adapter for forwarding the request
    "bcrypt": "^3.0.7", // Used for hashing the basic auth for consumer
    "bluebird": "^3.7.2", // Optional, I'm using it to async the redis with Async syntax
    "body-parser": "^1.19.0",
    "cors": "^2.8.5", // For enabling the CORS
    "express": "^4.17.1",
    "jsonwebtoken": "^8.5.1", // Used as the JWT for the gateway signature
    "mongoose": "^5.8.7",
    "morgan": "^1.9.1", // Optional for development log
    "redis": "^2.8.0", // Save it for later as the cache db
    "yaml": "^1.7.2" // For our configuration gateway
}
```

// npm install express axios body-parser bluebird cors jsonwebtoken morgan redis yaml bcrypt --save


```
function __grabRequest(req) {
    // We need the consume IP Address for collecting a log for our Gateway
    const ipAddress = (req.headers['x-forwarded-for'] || '').split(',').pop() || 
    req.connection.remoteAddress || 
    req.socket.remoteAddress || 
    req.connection.socket.remoteAddress
    const apiSignatureKey = req.headers['basic_auth'] || '' // Basic Auth for consumer.
    return {
        ip_address: ipAddress,
        basic_auth: apiSignatureKey,
        host: req.headers['host'], // Optional
        user_agent: req.headers['user-agent'] || '', // Optional
        method: req.method,
        path: req.path,
        originalUrl: req.originalUrl,
        query: req.query, // We keep the query request to be forward to the service
        params: req.params, // We also keep the params to be forwarded
        app_id: req.headers['app_id'], // This header for specifying which service will be used
        body: req.body, 
    }
}
```

```
// services:
//     firstService:
//         port: 3422
//         base_url: http://localhost
//         endpoints:
//             get:
//                 - /user
//                 - /item
//                 - /testing
//             put:
//                 - /user/*/*/fdf
//                 - /item/*/*
//             delete:
//                 - /user/
//                 - /item/
//             post: 
//                 - /user/create
//                 - /item/create
//         secret_key: mainServiceIsRestrictedOnly
//     secondService:
//         port: 3400
//         base_url: http://localhost
//         endpoints:
//             get:
//                 - /hello
//                 - /user/*
//             put:
//                 - /user/hello
//             delete:
//                 - /user/delete/*
//             post: 
//                 - /user/create
//         secret_key: mainServiceIsRestrictedOnly
```


```
__getServiceInformation(request.app_id || '')
.then(service => {
    let flag = false

    const availableEndPoints = service.endpoints[request.method.toLowerCase()] || []
    const splittedRequestPath = request.path.replace(/^\/|\/$/g, '').split('/')
    for ( let i = 0; i < availableEndPoints.length; i++ ) {
        let splittedEndPointPath = availableEndPoints[i].replace(/^\/|\/$/g, '').split('/')
        if ( splittedRequestPath.length === splittedEndPointPath.length ) {
            let fractalCheckFlag = true
            for ( let j = 0; j < splittedEndPointPath.length; j++ ) {
                if ( splittedEndPointPath[j] !== splittedRequestPath[j] && splittedEndPointPath[j] !== '*' ) {
                    fractalCheckFlag = false
                    break
                }
            }
            if ( fractalCheckFlag ) {
                flag = true
                break
            }
        }
    }
    if ( flag ) { // If method found
        logModel.addLog(new logModel({
            path: request.path,
            service: request.app_id,
            ip_address: request.ip_address
        }))
        callback(request, service, null)
    } else {
        const err = {
            type: 'NOT_FOUND',
            module_source: 'request_resolver',
            message: 'Request method is not found.'
        }
        callback(null, null, err)
    }
})
.catch(err => {
    callback(null, null, err)
})
```

```
function __generateGatewaySignature(serviceSecretKey, callback) {
    jwt.sign({
        gateway: 'ORION_GATEWAY',
        gateway_secret: SECRET_KEY,
    }, serviceSecretKey, { expiresIn: 1800000 }, callback)
}
```

```
axios({
    method: method,
    baseURL: service.base_url + ':' + service.port,
    url: request.path,
    responseType: 'json',
    data: request.body,
    params: request.params,
    headers: { 
        gateway_signature: token,
        authorization: request.authorization
    }
})
.then(response => { resolve(response) })
.catch(err => { reject(err) })
```

```
function __addSecureAndCacheHeaders(res) {
    // OWASP Secure Headers
    res.set('X-Content-Type-Options', 'nosniff')
    res.set('X-XSS-Protection', '1; mode=block')
    res.set('X-Frame-Options', 'DENY')
    res.set('Strict-Transport-Security', 'max-age=63072000; includeSubDomains')

    // Avoid Caching Tokens
    res.set('Cache-Control', 'no-cache, no-store, must-revalidate')
    res.set('Pragma', 'no-cache')
    res.set('Expires', '0')
}
```


we are done here with building a basic payment gateway.
We’re done… ( Not really :) )
Finally, the simple step to build your own API Gateway is done. There are so many things you could scale up like the authentication, caching or anything that really fits your needs.
Even then thats not marketable in countries like India specially where you have UPI like infrastructure


![alt text](https://nfinite.in/catalogue/how_it_works/2021/2/Upi_1613147183.png)


I'm listing NPCI API's
I dont think they are public
you wont be able to access them without proper license


# List of APIs

```
/ReqChkTxn/{version}/urn:txnid:{transaction}
bank ReqChkTxn

This API allows the PSPs to request for the status of the transaction. The PSPs must request for status only after the specified timeout period.
/ReqHbt/{version}/urn:txnid:{transaction}
bank ReqHbt

This API is a mechanism for UPI system monitoring i.e. testing connection status with PSPs and sending responding to PSPs when initiated by it.)
/ReqBalEnq/{version}/urn:txnid:{transaction}
bank ReqBalEnq

This API Allows PSP to enquirebalance of a user.
/ReqListAccount/{version}/urn:txnid:{transaction}
bank ReqListAccount

API allows PSPs to find the list of accounts linked to the mobile by an account provider.
/ReqListAccPvd/{version}/urn:txnid:{transaction}
bank ReqListAccPvd

/ReqListKeys/{version}/urn:txnid:{transaction}
bank ReqListKeys

/ReqListPsp/{version}/urn:txnid:{transaction}
bank ReqListPsp

/ReqListVae/{version}/urn:txnid:{transaction}
bank ReqListVae

/ReqManageVae/{version}/urn:txnid:{transaction}
bank ReqManageVae

/ReqOtp/{version}/urn:txnid:{transaction}
bank ReqOtp

/ReqPendingMsg/{version}/urn:txnid:{transaction}
bank ReqPendingMsg

/ReqRegMob/{version}/urn:txnid:{transaction}
bank ReqRegMob

/ReqSetCre/{version}/urn:txnid:{transaction}
bank ReqSetCre

/ReqTxnConfirmation/{version}/urn:txnid:{transaction}
bank ReqTxnConfirmation

/ReqValAdd/{version}/urn:txnid:{transaction}
bank ReqValAdd

/ReqMandate/{version}/urn:txnid:{transaction}
bank ReqMandate

/ReqPay/{version}/urn:txnid:{transaction}
bank ReqPay Info
```

This platform is only for testing purpose. APIs listed on this portal can be tested only in a stimulated environment and not in real-time. To fully use the APIs, you will have to contact the provider.


some usedful goverment links

# NPCI
https://www.npci.org.in/

# INDIA STACK
https://nfinite.in/product-detail/8cce7b57-b64e-458d-915f-b07e115bb1e3
https://upigateway.com/

All rights privileged! Ask before using my work anywhere. It might be out of date in most cases.
I do not share bleading-edge optimised versions due to NDA & legal challanges.
# Dipesh Bhoir
