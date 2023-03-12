# paytm gateway integration
Sure, I can provide you with a basic script to integrate Paytm gateway with Angular app and Node.js backend. However, please note that this script will only provide you with the basic integration and you will need to modify it according to your specific requirements.

Here are the steps to integrate Paytm gateway with your Angular app and Node.js backend:

Create an account on Paytm and obtain the Merchant ID and Merchant Key.

Install the Paytm SDK in your Angular app using the following command:
```
npm install paytm-pg-node-sdk
```

Create a form in your Angular app that will collect the necessary details for the payment transaction, such as the amount to be paid and the customer's details.

When the user submits the form, send the payment request to your Node.js backend.

In your Node.js backend, create a route that will receive the payment request from your Angular app. You will need to use the Paytm SDK to generate a unique transaction ID, calculate the checksum hash, and send the payment request to Paytm.

Once the payment request is sent to Paytm, you will receive a response containing a URL to the Paytm payment page. Redirect the user to this page so that they can complete the payment.

After the payment is complete, Paytm will redirect the user back to your Node.js backend. You will need to use the Paytm SDK to verify the response and update your database with the payment status.

Here's a basic script to help you get started with the integration:

Angular app:
```
import { Component } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-paytm-payment',
  templateUrl: './paytm-payment.component.html',
  styleUrls: ['./paytm-payment.component.css']
})
export class PaytmPaymentComponent {
  paytmParams: any = {};

  constructor(private http: HttpClient) {}

  ngOnInit() {
    // Set the Paytm payment parameters
    this.paytmParams.amount = '100';
    this.paytmParams.email = 'customer@email.com';
    this.paytmParams.mobileNumber = '9999999999';
  }

  // Submit the payment request to the Node.js backend
  initiatePayment() {
    this.http.post('/api/paytm', this.paytmParams).subscribe(response => {
      window.location.href = response['paymentUrl'];
    });
  }
}
```

Node.js backend:
```
const Paytm = require('paytm-pg-node-sdk');
const express = require('express');
const app = express();

app.use(express.json());

const PAYTM_MERCHANT_ID = '<your_merchant_id>';
const PAYTM_MERCHANT_KEY = '<your_merchant_key>';
const PAYTM_WEBSITE = 'WEBSTAGING';

// Create a Paytm instance
const paytm = new Paytm({
  merchantId: PAYTM_MERCHANT_ID,
  merchantKey: PAYTM_MERCHANT_KEY,
  website: PAYTM_WEBSITE,
  mode: 'TEST' // Replace with 'PROD' in production environment
});

// Route to handle payment requests
app.post('/api/paytm', async (req, res) => {
  try {
    // Generate a unique transaction ID
    const orderId = Date.now().toString();

    // Create a payment request
    const paymentDetails = {
      orderId: orderId,
      customerId: '123456',
      amount: req.body.amount,
      channel: 'WEB',
      industryTypeId: 'Retail',
      website: PAYTM_WEBSITE,
      callbackUrl: 'http://localhost:3000/api/paytm/callback',
      email: req.body.email,
      mobileNumber: req.body.mobileNumber
    };

    // Generate the checksum hash

const checksumHash = await paytm.generateChecksum(paymentDetails);

// Add the checksum hash to the payment request
paymentDetails.checksumHash = checksumHash;

// Send the payment request to Paytm
const paymentResponse = await paytm.initiateTransaction(paymentDetails);

// If the payment request was successful, return the Paytm payment URL to the Angular app
if (paymentResponse.status === 'SUCCESS') {
  res.send({ paymentUrl: paymentResponse.redirectUrl });
} else {
  res.status(400).send({ message: 'Payment request failed' });
}
} catch (err) {
console.error(err);
res.status(500).send({ message: 'Internal server error' });
}
});

// Route to handle Paytm callback
app.post('/api/paytm/callback', async (req, res) => {
try {
// Verify the Paytm response using the checksum hash
const checksumHash = await paytm.verifyChecksum(req.body);


// If the Paytm response is valid, update the database with the payment status
if (checksumHash.isValid) {
  // Check the payment status and update the database accordingly
  res.send({ message: 'Payment successful' });
} else {
  res.status(400).send({ message: 'Invalid Paytm response' });
}
} catch (err) {
console.error(err);
res.status(500).send({ message: 'Internal server error' });
}
});

app.listen(3000, () => {
console.log('Server started on port 3000');
});
```
This is a basic script to get you started with integrating Paytm gateway with your Angular app and Node.js backend. You will need to modify it according to your specific requirements.

