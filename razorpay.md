# Razorpay gateway integration
integrating the Razorpay payment gateway with an Angular app and Node.js backend. 
Here's a step-by-step guide:

First, you need to create an account with Razorpay and obtain your API keys. You can find your API keys in the Razorpay Dashboard under the "API Keys" section.

Next, you need to install the Razorpay SDK in your Angular app. You can do this by running the following command in your Angular project directory:
```
npm install razorpay --save
```

After installing the SDK, you need to add the Razorpay script to your index.html file. You can do this by adding the following line in the head section of your index.html file:
```
<script src="https://checkout.razorpay.com/v1/checkout.js"></script>
```

Next, you need to create a Razorpay payment form in your Angular app. You can do this by creating a new component and adding the following code:
```
import { Component } from '@angular/core';
import * as Razorpay from 'razorpay';

@Component({
  selector: 'app-payment',
  templateUrl: './payment.component.html',
  styleUrls: ['./payment.component.css']
})
export class PaymentComponent {
  private razorpay: any;

  constructor() {
    this.razorpay = new Razorpay({
      key_id: 'YOUR_RAZORPAY_KEY_ID',
      key_secret: 'YOUR_RAZORPAY_KEY_SECRET'
    });
  }

  makePayment(amount: number) {
    const options = {
      amount: amount,
      currency: 'INR',
      description: 'Payment for Order No: #123',
      handler: (response: any) => {
        console.log(response);
        // Call your backend API to verify the payment
      },
      prefill: {
        name: 'John Doe',
        email: 'john.doe@example.com',
        contact: '+919876543210'
      },
      notes: {
        address: 'Razorpay Corporate Office'
      },
      theme: {
        color: '#F37254'
      }
    };

    this.razorpay.open(options);
  }
}
```

In this code, we have created a new component called "PaymentComponent" that initializes the Razorpay SDK with your API keys and creates a new payment form. The "makePayment" function opens the payment form with the specified options.
After creating the payment form, you need to create a Node.js backend API to verify the payment. You can do this by creating a new route in your Node.js app and adding the following code:

```
const express = require('express');
const Razorpay = require('razorpay');

const app = express();
const razorpay = new Razorpay({
  key_id: 'YOUR_RAZORPAY_KEY_ID',
  key_secret: 'YOUR_RAZORPAY_KEY_SECRET'
});

app.post('/api/verify-payment', (req, res) => {
  const { razorpay_payment_id, razorpay_order_id, razorpay_signature } = req.body;

  const signature = razorpay.utils.generateSignature(JSON.stringify({
    razorpay_payment_id,
    razorpay_order_id
  }), 'YOUR_RAZORPAY_KEY_SECRET');

  if (signature === razorpay_signature) {
    // Payment is successful, update your database and send a response
    res.send({ success: true });
  } else {
    // Payment verification failed, send a response with error message
    res.send({ success: false, message: 'Payment verification failed' });
}
});

app.listen(3000, () => console.log('Server started at http://localhost:3000'));
```

In this code, we have created a new route "/api/verify-payment" that receives the payment details from the frontend and verifies the payment signature using the Razorpay SDK. If the payment signature is valid, you can update your database and send a response with "success: true". If the payment signature is invalid, you can send a response with "success: false" and an error message.

6. Finally, you need to call the backend API to verify the payment after the payment is successful. You can do this by modifying the "handler" function in your payment form:
```
handler: (response: any) => {
console.log(response);
// Call your backend API to verify the payment
fetch('/api/verify-payment', {
method: 'POST',
headers: {
'Content-Type': 'application/json'
},
body: JSON.stringify(response)
}).then(res => {
if (res.success) {
// Payment is successful, redirect to success page
window.location.href = '/success';
} else {
// Payment verification failed, show error message
alert(res.message);
}
}).catch(err => {
// Payment verification failed, show error message
alert('Payment verification failed');
});
},
```

In this code, we have added a fetch call to your backend API to verify the payment after the payment is successful. If the payment verification is successful, you can redirect the user to a success page. If the payment verification fails, you can show an error message.
That's it! You have now successfully integrated the Razorpay payment gateway with your Angular app and Node.js backend.
