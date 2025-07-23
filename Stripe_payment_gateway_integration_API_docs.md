# Stripe Payment Integration API Documentation
## Overview
This documentation provides a comprehensive guide to integrating Stripe as a payment
gateway for web applications. It covers backend and frontend integration, API calls, and
error handling, following best practices for secure and seamless transactions.
---
## Prerequisites
- A [Stripe](https://stripe.com/) account
- Secret API keys from the Stripe Dashboard
- Backend server using Node.js, Python, Java, or any supported language
- HTTPS endpoint (Stripe requires secure connections)
- Frontend framework (e.g., React, Angular, HTML/JS)
---
## API Keys
> ⚠️ **Do not expose your secret keys on the frontend.**
- **Publishable Key:** Used on the client side
- **Secret Key:** Used on the server side for creating payment intents and handling
secure operation
```bash
# Example (Set via ENV)
STRIPE_PUBLISHABLE_KEY=pk_test_123456789
STRIPE_SECRET_KEY=sk_test_123456789
```
---
## Flow Diagram
1. Client requests payment
2. Backend creates a PaymentIntent
3. Backend returns `client_secret` to frontend
4. Frontend confirms payment using Stripe.js
5. Payment is processed and response is handled
---
## Backend Integration (Node.js Example)
### 1. Install Stripe SDK
```bash
npm install stripe
```
### 2. Create a PaymentIntent
```javascript
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
app.post('/create-payment-intent', async (req, res) => {
 const { amount, currency } = req.body;
 try {
 const paymentIntent = await stripe.paymentIntents.create({
 amount: amount, // in smallest currency unit (e.g., cents)
 currency: currency,
 });
 res.send({
 clientSecret: paymentIntent.client_secret,
 });
 } catch (error) {
 res.status(500).json({ error: error.message });
 }
});
```
---
## Frontend Integration (Using Stripe.js)
### 1. Include Stripe.js
```html
<script src="https://js.stripe.com/v3/"></script>
```
### 2. Confirm the Payment
```javascript
const stripe = Stripe('pk_test_123456789');
const elements = stripe.elements();
const cardElement = elements.create('card');
cardElement.mount('#card-element');
const form = document.getElementById('payment-form');
form.addEventListener('submit', async (e) => {
 e.preventDefault();
 const { clientSecret } = await fetch('/create-payment-intent', {
 method: 'POST',
 headers: { 'Content-Type': 'application/json' },
 body: JSON.stringify({ amount: 5000, currency: 'usd' }),
 }).then(r => r.json());
const { error, paymentIntent } = await stripe.confirmCardPayment(clientSecret, {
 payment_method: {
 card: cardElement,
 }
 });
 if (error) {
 console.error('Payment failed:', error.message);
 } else if (paymentIntent.status === 'succeeded') {
 console.log('Payment successful!');
 }
});
```
---
## Common Errors
| Error Code | Description | Suggested Fix
 |
|----------------------|---------------------------------------------|------------------
--------------------|
| `card_declined` | The card was declined | Try another card
 |
| `incorrect_cvc` | Incorrect security code | Double-check CVC
 |
| `expired_card` | Card has expired | Use a valid,
unexpired card |
| `api_connection_error` | Network issues with Stripe API | Retry with
exponential backoff |
---
## Webhooks (Optional but Recommended)
Use webhooks to listen for events like `payment_intent.succeeded`.
```javascript
stripe.webhooks.constructEvent(...)
```
For local testing:
```bash
stripe listen --forward-to localhost:3000/webhook
```
---
## Best Practices
- Always validate amounts on the server
- Use HTTPS for all endpoints
- Tokenize card data using Stripe Elements
- Use `metadata` to link transactions with internal order IDs
- Store only what’s necessary—never save card numbers
---
## Useful Resources
- [Stripe API Reference](https://stripe.com/docs/api)
- [Stripe Node.js SDK](https://github.com/stripe/stripe-node)
- [Stripe Testing Cards](https://stripe.com/docs/testing)
---
## Changelog
### v1.0.0 – Initial Draft
- Added backend and frontend setup
- Included error handling and webhook notes
- Stripe.js and PaymentIntent API covered
---
## Author
*Aishwarya Pandey, Technical Writer 

