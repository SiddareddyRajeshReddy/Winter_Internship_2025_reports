# Django Razorpay Payment Gateway

A complete Django-based payment gateway integration with Razorpay, supporting payment creation, verification, status tracking, and refund processing.

## Features

- 💳 **Payment Processing**: Create and capture payments through Razorpay
- ✅ **Payment Verification**: Secure signature verification for payment callbacks
- 📊 **Payment Status Tracking**: Real-time payment status monitoring
- 💰 **Refund Management**: Full and partial refund support
- 📝 **Payment History**: List and filter payment records
- 🔒 **Secure Integration**: Signature verification and secure API handling

## Tech Stack

- **Backend**: Django 4.2.7
- **API**: Django REST Framework
- **Payment Gateway**: Razorpay Python SDK
- **Database**: SQLite (development)
- **Environment Management**: python-decouple

## Prerequisites

- Python 3.8+
- Django 4.2+
- Razorpay Account (for API credentials)
- pip (Python package manager)

## Installation

### 1. Clone the Repository

```bash
git clone <repository-url>
cd payment_gateway
```

### 2. Create Virtual Environment

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install django
pip install djangorestframework
pip install razorpay
pip install python-decouple
```

### 4. Environment Configuration

Create a `.env` file in the project root:

```env
RAZORPAY_KEY_ID=your_razorpay_key_id
RAZORPAY_KEY_SECRET=your_razorpay_key_secret
```

**Note**: Get your Razorpay credentials from [Razorpay Dashboard](https://dashboard.razorpay.com/app/keys)

### 5. Database Setup

```bash
python manage.py makemigrations
python manage.py migrate
```

### 6. Run Development Server

```bash
python manage.py runserver
```

The application will be available at `http://127.0.0.1:8000/`

## Database Schema

### Payment Model

| Field | Type | Description |
|-------|------|-------------|
| `razorpay_order_id` | CharField | Unique Razorpay order identifier |
| `razorpay_payment_id` | CharField | Razorpay payment identifier |
| `razorpay_signature` | CharField | Payment signature for verification |
| `amount` | DecimalField | Payment amount |
| `currency` | CharField | Currency code (default: INR) |
| `status` | CharField | Payment status (created, authorized, captured, refunded, failed, partial_refund) |
| `customer_name` | CharField | Customer name |
| `customer_email` | EmailField | Customer email |
| `customer_contact` | CharField | Customer contact number |
| `refund_amount` | DecimalField | Total refunded amount |
| `razorpay_refund_id` | CharField | Razorpay refund identifier |
| `created_at` | DateTimeField | Record creation timestamp |
| `updated_at` | DateTimeField | Record update timestamp |
| `notes` | TextField | Additional notes |

## API Endpoints

### 1. Create Payment Order

**Endpoint**: `POST /create_order/`

**Request Body**:
```json
{
  "amount": 500.00,
  "name": "John Doe",
  "email": "john@example.com",
  "contact": "9876543210"
}
```

**Response**:
```json
{
  "amount": 50000,
  "currency": "INR",
  "razorpay_key": "rzp_test_xxxxx",
  "razorpay_order_id": "order_xxxxx"
}
```

### 2. Verify Payment

**Endpoint**: `POST /payment_handler/`

**Form Data** (from Razorpay callback):
- `razorpay_payment_id`
- `razorpay_order_id`
- `razorpay_signature`

**Response**: HTML page with payment status

### 3. Get Payment Status

**Endpoint**: `GET /payment_status/<order_id>/`

**Response**:
```json
{
  "order_id": "order_xxxxx",
  "payment_id": "pay_xxxxx",
  "amount": 500.00,
  "currency": "INR",
  "status": "captured",
  "refund_amount": 0.00,
  "customer_name": "John Doe",
  "customer_email": "john@example.com",
  "created_at": "2024-01-01T10:00:00Z",
  "updated_at": "2024-01-01T10:05:00Z"
}
```

### 4. List Payments

**Endpoint**: `GET /list_payments/`

**Query Parameters**:
- `status` (optional): Filter by payment status
- `limit` (optional): Number of records (default: 50)

**Response**:
```json
{
  "count": 10,
  "payments": [
    {
      "order_id": "order_xxxxx",
      "payment_id": "pay_xxxxx",
      "amount": 500.00,
      "status": "captured",
      "refund_amount": 0.00,
      "customer_name": "John Doe",
      "created_at": "2024-01-01T10:00:00Z"
    }
  ]
}
```

### 5. Initiate Refund

**Endpoint**: `POST /initiate_refund/`

**Request Body**:
```json
{
  "payment_id": "pay_xxxxx",
  "amount": 250.00  // Optional: for partial refund, omit for full refund
}
```

**Response**:
```json
{
  "message": "Refund initiated successfully",
  "refund_id": "rfnd_xxxxx",
  "amount_refunded": 250.00,
  "total_refunded": 250.00,
  "status": "partial_refund"
}
```

### 6. Fetch Refund Details

**Endpoint**: `GET /refund_details/<refund_id>/`

**Response**:
```json
{
  "refund_id": "rfnd_xxxxx",
  "payment_id": "pay_xxxxx",
  "amount": 250.00,
  "currency": "INR",
  "status": "processed",
  "created_at": 1234567890
}
```

## Payment Flow

### Standard Payment Flow

1. **Create Order**: Frontend calls `/create_order/` with amount and customer details
2. **Initialize Razorpay**: Use returned credentials to open Razorpay checkout
3. **Payment Callback**: Razorpay redirects to `/payment_handler/` with payment details
4. **Verification**: Server verifies payment signature
5. **Status Update**: Payment status updated to 'captured' or 'failed'

### Refund Flow

1. **Initiate Refund**: Call `/initiate_refund/` with payment_id
2. **Process**: System creates refund via Razorpay API
3. **Update Status**: Payment status updated to 'refunded' or 'partial_refund'
4. **Track**: Use `/refund_details/` to check refund status

## Payment Statuses

- `created`: Order created but payment not initiated
- `authorized`: Payment authorized but not captured
- `captured`: Payment successfully captured
- `refunded`: Full refund processed
- `partial_refund`: Partial refund processed
- `failed`: Payment failed or signature verification failed

## Security Considerations

### 1. Environment Variables
Never commit `.env` file or expose API credentials in code.

### 2. CSRF Protection
Most endpoints use `@csrf_exempt` for API access. For production:
- Implement proper CSRF token handling
- Use session-based authentication
- Consider JWT tokens for API endpoints

### 3. Signature Verification
All payment callbacks are verified using Razorpay's signature verification to prevent tampering.

### 4. Production Checklist
- [ ] Set `DEBUG = False`
- [ ] Configure `ALLOWED_HOSTS`
- [ ] Use environment variables for secrets
- [ ] Enable HTTPS
- [ ] Implement rate limiting
- [ ] Add logging and monitoring
- [ ] Use PostgreSQL or MySQL instead of SQLite
- [ ] Set up proper CORS headers
- [ ] Implement authentication for sensitive endpoints

## Frontend Integration Example

```html
<!DOCTYPE html>
<html>
<head>
    <script src="https://checkout.razorpay.com/v1/checkout.js"></script>
</head>
<body>
    <button onclick="makePayment()">Pay Now</button>
    
    <script>
        async function makePayment() {
            // Create order
            const response = await fetch('/create_order/', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    amount: 500,
                    name: 'John Doe',
                    email: 'john@example.com',
                    contact: '9876543210'
                })
            });
            
            const data = await response.json();
            
            // Open Razorpay checkout
            const options = {
                key: data.razorpay_key,
                amount: data.amount,
                currency: data.currency,
                order_id: data.razorpay_order_id,
                name: 'Your Company Name',
                description: 'Payment for services',
                handler: function(response) {
                    // Submit to verification endpoint
                    const form = document.createElement('form');
                    form.method = 'POST';
                    form.action = '/payment_handler/';
                    
                    const fields = {
                        razorpay_payment_id: response.razorpay_payment_id,
                        razorpay_order_id: response.razorpay_order_id,
                        razorpay_signature: response.razorpay_signature
                    };
                    
                    for (let key in fields) {
                        const input = document.createElement('input');
                        input.type = 'hidden';
                        input.name = key;
                        input.value = fields[key];
                        form.appendChild(input);
                    }
                    
                    document.body.appendChild(form);
                    form.submit();
                },
                prefill: {
                    name: 'John Doe',
                    email: 'john@example.com',
                    contact: '9876543210'
                }
            };
            
            const rzp = new Razorpay(options);
            rzp.open();
        }
    </script>
</body>
</html>
```

## Testing

### Manual Testing with cURL

**Create Payment:**
```bash
curl -X POST http://127.0.0.1:8000/create_order/ \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 500,
    "name": "Test User",
    "email": "test@example.com",
    "contact": "9876543210"
  }'
```

**Get Payment Status:**
```bash
curl http://127.0.0.1:8000/payment_status/order_xxxxx/
```

**List Payments:**
```bash
curl "http://127.0.0.1:8000/list_payments/?status=captured&limit=10"
```

**Initiate Refund:**
```bash
curl -X POST http://127.0.0.1:8000/initiate_refund/ \
  -H "Content-Type: application/json" \
  -d '{
    "payment_id": "pay_xxxxx",
    "amount": 250
  }'
```

## Troubleshooting

### Common Issues

**1. Signature Verification Failed**
- Ensure Razorpay credentials are correct
- Verify payment details are passed correctly from frontend
- Check if test/live mode keys match

**2. Payment Not Found**
- Verify order_id is correct
- Check if payment record was created in database

**3. Refund Failed**
- Ensure payment is in 'captured' status
- Verify payment_id is correct
- Check if payment was made more than 180 days ago

**4. ImportError: No module named 'decouple'**
```bash
pip install python-decouple
```

## Project Structure

```
payment_gateway/
├── core/                      # Main app
│   ├── models.py             # Payment model
│   ├── views.py              # API endpoints
│   ├── urls.py               # URL routing
│   └── tests.py              # Test cases
├── payment_gateway/          # Project settings
│   ├── settings.py           # Django settings
│   ├── urls.py               # Root URL config
│   └── wsgi.py              # WSGI config
├── .env                      # Environment variables (not in repo)
├── manage.py                 # Django management script
└── README.md                 # This file
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License.

## Support

For Razorpay API documentation: [https://razorpay.com/docs/api/](https://razorpay.com/docs/api/)

For issues and questions, please open an issue on GitHub.

## Acknowledgments

- [Razorpay](https://razorpay.com/) for payment gateway services
- [Django](https://www.djangoproject.com/) for the web framework
- [Django REST Framework](https://www.django-rest-framework.org/) for API development
