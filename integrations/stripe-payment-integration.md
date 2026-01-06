# Stripe Payment Integration

## Objective
Implement a complete Stripe payment system with subscription management, one-time payments, webhooks, and customer portal integration.

## Technology Stack
- Backend: Node.js/Express with TypeScript
- Frontend: React with TypeScript
- Stripe Node.js SDK
- @stripe/react-stripe-js for frontend
- @stripe/stripe-js for Stripe Elements
- Database: PostgreSQL/MongoDB for storing payment records

## Requirements

### 1. Payment Features

**One-Time Payments:**
- Product checkout flow
- Payment intent creation
- Card payment processing
- 3D Secure (SCA) support
- Payment confirmation
- Receipt generation

**Subscriptions:**
- Create subscription plans
- Subscribe users to plans
- Upgrade/downgrade subscriptions
- Cancel subscriptions
- Trial periods
- Proration handling
- Usage-based billing

**Customer Management:**
- Create Stripe customers
- Save payment methods
- Update payment methods
- View payment history
- Customer portal access

### 2. Backend Implementation

**API Endpoints:**
```typescript
// One-time payments
POST /api/payments/create-payment-intent
POST /api/payments/confirm-payment
GET /api/payments/:id

// Subscriptions
POST /api/subscriptions/create
POST /api/subscriptions/:id/cancel
POST /api/subscriptions/:id/upgrade
POST /api/subscriptions/:id/pause
GET /api/subscriptions/plans
GET /api/subscriptions/user/:userId

// Customer
POST /api/customers/create
GET /api/customers/:id
POST /api/customers/:id/payment-methods
DELETE /api/customers/:id/payment-methods/:methodId
POST /api/customers/:id/portal-session

// Webhooks
POST /api/webhooks/stripe
```

**Webhook Events to Handle:**
```typescript
const WEBHOOK_EVENTS = {
  // Payment events
  'payment_intent.succeeded': handlePaymentSuccess,
  'payment_intent.payment_failed': handlePaymentFailed,

  // Subscription events
  'customer.subscription.created': handleSubscriptionCreated,
  'customer.subscription.updated': handleSubscriptionUpdated,
  'customer.subscription.deleted': handleSubscriptionCanceled,
  'customer.subscription.trial_will_end': handleTrialEnding,

  // Invoice events
  'invoice.paid': handleInvoicePaid,
  'invoice.payment_failed': handleInvoicePaymentFailed,
  'invoice.upcoming': handleUpcomingInvoice,

  // Customer events
  'customer.created': handleCustomerCreated,
  'customer.updated': handleCustomerUpdated,
  'customer.deleted': handleCustomerDeleted,
};
```

### 3. Frontend Components

**CheckoutForm Component:**
```typescript
interface CheckoutFormProps {
  amount: number;
  currency: string;
  productName: string;
  onSuccess: (paymentIntent: PaymentIntent) => void;
  onError: (error: Error) => void;
}
```

**SubscriptionPlans Component:**
- Display available subscription plans
- Highlight recommended plan
- Show pricing (monthly/yearly toggle)
- Feature comparison table
- CTA buttons for each plan

**PaymentMethodForm Component:**
- Stripe Card Element integration
- Save card for future use option
- Billing address collection
- Form validation
- Error handling

**SubscriptionManagement Component:**
- Current plan display
- Upgrade/downgrade options
- Cancel subscription
- View billing history
- Update payment method
- Access customer portal

**InvoiceHistory Component:**
- List of past invoices
- Download invoice PDFs
- Payment status indicators
- Retry failed payments

### 4. File Structure
```
backend/
├── controllers/
│   ├── payment.controller.ts
│   ├── subscription.controller.ts
│   ├── customer.controller.ts
│   └── webhook.controller.ts
├── services/
│   ├── stripe.service.ts
│   ├── payment.service.ts
│   ├── subscription.service.ts
│   └── customer.service.ts
├── models/
│   ├── Payment.model.ts
│   ├── Subscription.model.ts
│   └── Customer.model.ts
├── routes/
│   ├── payment.routes.ts
│   ├── subscription.routes.ts
│   └── webhook.routes.ts
├── webhooks/
│   └── stripe.webhook.ts
└── utils/
    └── stripe.utils.ts

frontend/
├── components/
│   ├── payments/
│   │   ├── CheckoutForm.tsx
│   │   ├── PaymentMethodForm.tsx
│   │   ├── SubscriptionPlans.tsx
│   │   ├── SubscriptionManagement.tsx
│   │   └── InvoiceHistory.tsx
│   └── stripe/
│       └── StripeWrapper.tsx
├── hooks/
│   ├── useStripe.ts
│   ├── usePayment.ts
│   └── useSubscription.ts
└── utils/
    └── stripe.utils.ts
```

### 5. Database Schema

**Customers Table:**
```typescript
{
  id: UUID,
  userId: UUID (FK),
  stripeCustomerId: string (unique),
  email: string,
  metadata: JSON,
  createdAt: DateTime,
  updatedAt: DateTime
}
```

**Subscriptions Table:**
```typescript
{
  id: UUID,
  customerId: UUID (FK),
  stripeSubscriptionId: string (unique),
  planId: string,
  status: enum (active, canceled, past_due, trialing),
  currentPeriodStart: DateTime,
  currentPeriodEnd: DateTime,
  cancelAtPeriodEnd: boolean,
  trialEnd: DateTime?,
  metadata: JSON,
  createdAt: DateTime,
  updatedAt: DateTime
}
```

**Payments Table:**
```typescript
{
  id: UUID,
  customerId: UUID (FK),
  stripePaymentIntentId: string (unique),
  amount: number,
  currency: string,
  status: enum (succeeded, failed, pending, canceled),
  paymentMethod: string,
  description: string?,
  metadata: JSON,
  createdAt: DateTime,
  updatedAt: DateTime
}
```

### 6. Stripe Setup

**Create Products & Prices:**
```typescript
// Subscription plans
const plans = [
  {
    name: 'Starter',
    price: 9.99,
    interval: 'month',
    features: ['Feature 1', 'Feature 2'],
  },
  {
    name: 'Professional',
    price: 29.99,
    interval: 'month',
    features: ['All Starter features', 'Feature 3', 'Feature 4'],
  },
  {
    name: 'Enterprise',
    price: 99.99,
    interval: 'month',
    features: ['All Pro features', 'Feature 5', 'Priority support'],
  },
];
```

### 7. Security Best Practices

- Store API keys securely in environment variables
- Use Stripe webhook signatures to verify events
- Never expose secret key on frontend
- Implement idempotency keys for payments
- Validate amounts on server-side
- Use HTTPS for all requests
- Implement rate limiting
- Log all payment attempts
- PCI compliance (use Stripe Elements, never handle raw card data)

### 8. Webhook Implementation

```typescript
export const handleStripeWebhook = async (req: Request, res: Response) => {
  const sig = req.headers['stripe-signature'];
  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      req.body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET
    );
  } catch (err) {
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }

  // Handle the event
  switch (event.type) {
    case 'payment_intent.succeeded':
      await handlePaymentSuccess(event.data.object);
      break;
    case 'customer.subscription.updated':
      await handleSubscriptionUpdated(event.data.object);
      break;
    // ... other cases
  }

  res.json({ received: true });
};
```

### 9. Frontend Stripe Setup

```typescript
// App.tsx
import { Elements } from '@stripe/react-stripe-js';
import { loadStripe } from '@stripe/stripe-js';

const stripePromise = loadStripe(process.env.REACT_APP_STRIPE_PUBLIC_KEY);

function App() {
  return (
    <Elements stripe={stripePromise}>
      <YourComponents />
    </Elements>
  );
}
```

**Payment Form:**
```typescript
const CheckoutForm = () => {
  const stripe = useStripe();
  const elements = useElements();
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    if (!stripe || !elements) return;

    setLoading(true);

    const { error, paymentIntent } = await stripe.confirmCardPayment(
      clientSecret,
      {
        payment_method: {
          card: elements.getElement(CardElement),
          billing_details: { name, email },
        },
      }
    );

    if (error) {
      // Handle error
    } else if (paymentIntent.status === 'succeeded') {
      // Payment successful
    }

    setLoading(false);
  };

  return (
    <form onSubmit={handleSubmit}>
      <CardElement />
      <button disabled={!stripe || loading}>Pay</button>
    </form>
  );
};
```

### 10. Environment Variables
```
# Backend
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PUBLISHABLE_KEY=pk_test_...

# Frontend
REACT_APP_STRIPE_PUBLIC_KEY=pk_test_...
```

### 11. Advanced Features

**Subscription Trials:**
```typescript
const subscription = await stripe.subscriptions.create({
  customer: customerId,
  items: [{ price: priceId }],
  trial_period_days: 14,
});
```

**Proration:**
```typescript
// Handled automatically by Stripe when upgrading/downgrading
const subscription = await stripe.subscriptions.update(subscriptionId, {
  items: [{ id: itemId, price: newPriceId }],
  proration_behavior: 'create_prorations',
});
```

**Usage-Based Billing:**
```typescript
// Report usage
await stripe.subscriptionItems.createUsageRecord(itemId, {
  quantity: 100,
  timestamp: Math.floor(Date.now() / 1000),
});
```

**Customer Portal:**
```typescript
const session = await stripe.billingPortal.sessions.create({
  customer: customerId,
  return_url: 'https://your-domain.com/account',
});
// Redirect user to session.url
```

## Deliverables
- Complete payment and subscription system
- Webhook handling for all critical events
- Frontend components for checkout and subscription management
- Database models for tracking payments
- Comprehensive error handling
- Testing suite for payment flows
- Documentation for setup and usage
- Security best practices implemented

## Testing Checklist
- [ ] One-time payment succeeds
- [ ] Payment fails gracefully with card decline
- [ ] 3D Secure authentication works
- [ ] Subscription creation succeeds
- [ ] Subscription upgrade/downgrade works
- [ ] Subscription cancellation works
- [ ] Trial periods function correctly
- [ ] Webhooks are verified and processed
- [ ] Customer portal redirects correctly
- [ ] Payment methods can be added/removed
- [ ] Invoice history displays correctly
- [ ] Refunds process successfully
- [ ] Proration calculates correctly
- [ ] Usage-based billing reports usage
- [ ] Error handling provides clear messages
