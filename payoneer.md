# payoneer gateway integration
Sure! Here is a basic script to integrate the Payoneer payment gateway with an Angular app and a Node.js backend:

Angular App

First, install the Payoneer SDK in your Angular app using npm:
```
npm install payoneer-sdk --save
```

Import the Payoneer SDK in your Angular component:
```
import PayoneerSDK from 'payoneer-sdk';
```

Create a Payoneer instance and initialize it with your Payoneer credentials:
```
const payoneer = new PayoneerSDK({
  partnerId: 'your-partner-id',
  apiPassword: 'your-api-password',
  username: 'your-username',
  password: 'your-password',
});
```
Create a function to handle the payment request. This function should call the pay method of the Payoneer instance, passing in the necessary payment details:
```
async function handlePayment() {
  const paymentDetails = {
    amount: 10,
    currency: 'USD',
    description: 'Test payment',
    payerEmail: 'test@example.com',
    vendorId: 'vendor123',
    vendorName: 'Test Vendor',
  };

  const paymentResult = await payoneer.pay(paymentDetails);

  console.log(paymentResult);
}
```
Call the handlePayment function when the user clicks the payment button.
Node.js Backend

Install the Payoneer SDK in your Node.js backend using npm:
```
npm install payoneer-sdk --save
```
Import the Payoneer SDK in your Node.js server:
```
const PayoneerSDK = require('payoneer-sdk');
```
Create a Payoneer instance and initialize it with your Payoneer credentials:
```
const payoneer = new PayoneerSDK({
  partnerId: 'your-partner-id',
  apiPassword: 'your-api-password',
  username: 'your-username',
  password: 'your-password',
});
```
Create an API endpoint to handle the payment request. This endpoint should call the pay method of the Payoneer instance, passing in the necessary payment details:
```
app.post('/pay', async (req, res) => {
  const paymentDetails = {
    amount: req.body.amount,
    currency: req.body.currency,
    description: req.body.description,
    payerEmail: req.body.payerEmail,
    vendorId: req.body.vendorId,
    vendorName: req.body.vendorName,
  };

  const paymentResult = await payoneer.pay(paymentDetails);

  res.send(paymentResult);
});
```
Start your Node.js server and test the payment endpoint using a tool like Postman or curl.
That's it! This is a basic implementation of the Payoneer payment gateway integration with Angular and Node.js. You can customize it according to your specific requirements.
