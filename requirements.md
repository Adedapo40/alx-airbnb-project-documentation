## 🧩 1. User Authentication Module
### Functional Requirements

* Users can register as either Guest or Host.

* Users can log in using email and password.

* The system must authenticate using JWT tokens.

* Passwords must be securely hashed (bcrypt or Argon2) before storage.

* Implement role-based access control (RBAC) (Admin, Host, Guest).

### API Endpoints
* ``` POST	/api/auth/register ```  -	Register a new user (Guest or Host)
* ```POST	/api/auth/login```	- Authenticate user credentials
* ```GET	/api/auth/profile```	- Fetch user profile (JWT required)
* ```PUT	/api/auth/profile```	- Update user profile
* ```Input / Output - Specification```
* ```POST /api/auth/register```

#### Input (JSON):

```{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "StrongPass123",
  "role": "host"
}
```  

#### Output (Success 201):

```{
  "message": "User registered successfully",
  "user": {
    "id": "u123",
    "name": "John Doe",
    "role": "host"
  },
  "token": "jwt-token"
}
```
#### POST /api/auth/login

#### Input:
```
{
  "email": "john@example.com",
  "password": "StrongPass123"
}
```
#### Output (Success 200):
```
{
  "message": "Login successful",
  "token": "jwt-token"
}
```
#### Validation Rules

* Email must be unique and valid.

* Password must be ≥ 8 characters, contain uppercase, lowercase, and numbers.

* Role must be one of: ```guest```, ```host```, ```admin```.

* All fields required on registration.

#### Performance Criteria

* Authentication response time ≤ 1 second.

* Token validation time ≤ 500 ms.

* Support at least 100 concurrent login requests.



## 🏠 2. Property Management Module
## Functional Requirements

* Hosts can create, edit, and delete property listings.

* Each listing includes title, description, price, location, amenities, and images.

* Guests can view all available properties.

* Properties can be filtered by location, price, and amenities.

### API Endpoints
* ```POST	/api/properties```:	Add new property (Host only)
* ```GET	/api/properties```:	Retrieve all or filtered properties
* ```GET	/api/properties/```: id	Retrieve a single property by ID
* ```PUT	/api/properties/```: id	Update property (Host only)
* ```DELETE	/api/properties/```: id	Delete property (Host only)
* ```Input / Output Specification```
* ```POST /api/properties```

#### Input:

```{
  "title": "Cozy Apartment",
  "description": "2-bedroom apartment in city center",
  "location": "Lagos, Nigeria",
  "price_per_night": 150,
  "amenities": ["WiFi", "Air Conditioning"],
  "availability": true
}
```

#### Output:

```{
  "message": "Property created successfully",
  "property_id": "p123"
}
```
#### Validation Rules

*  Title, location, price_per_night, and description are required.

* Price must be a positive number.

* Only the property owner (host) can modify or delete listings.

* Image upload size ≤ 5MB per file.

#### Performance Criteria

* Listing retrieval time ≤ 2 seconds (with filters applied).

* API must handle at least 500 simultaneous property queries.

* Use Redis caching for popular searches to reduce database load.

## 📅 3. Booking System Module
### Functional Requirements

* Guests can book available properties.

* Prevent overlapping bookings (date validation).

* Support booking cancellation (with status update).

* Automatically update property availability after successful booking.

### API Endpoints

```POST	/api/bookings```	Create a new booking
```GET	/api/bookings```	Retrieve all bookings (Admin/Host/Guest)
```GET	/api/bookings/ ```: id	Retrieve booking details
```PUT	/api/bookings/```: id/cancel	Cancel a booking
```Input / Output Specification```
```POST /api/bookings```

#### Input:

```{
  "property_id": "p123",
  "user_id": "u456",
  "check_in": "2025-10-10",
  "check_out": "2025-10-15",
  "total_price": 750
}
````

#### Output (Success 201):

```{
  "message": "Booking successful",
  "booking": {
    "id": "b789",
    "status": "confirmed"
  }
}
```

### Validation Rules

* Property must exist and be available for selected dates.

* Check-out date must be after check-in.

* Total price = nights × price_per_night.

* Only the guest or host can cancel a booking.

### Performance Criteria

* Booking confirmation time ≤ 2 seconds (including payment verification).

* Booking API should process up to 200 concurrent requests without failure.

* Use database transactions to ensure data consistency.
  

## 💳 4. Payment Integration Module
### Functional Requirements

* Users can make secure payments for confirmed bookings.

*Payments are processed using a third-party gateway (e.g., Stripe, PayPal, or Flutterwave).

* System must verify payment status via webhooks.

* Once payment is successful:

* Booking status is updated to “paid”.

* Property availability is marked as unavailable for the booked dates.

* Generate payment receipts and store transaction history.

### API Endpoints

* ```POST	/api/payments/initiate```	Start a payment process for a booking
* ```POST	/api/payments/webhook```: Receive payment confirmation from payment provider
* ```GET	/api/payments/```: booking_id	Retrieve payment details for a booking
* ```GET	/api/payments/history/```: user_id	Retrieve user’s payment history
* ```Input / Output Specification```
* ```POST /api/payments/initiate```

#### Input:

```{
  "booking_id": "b789",
  "amount": 750,
  "payment_method": "stripe"
}
```

#### Output (Success 200):

```{
  "payment_url": "https://checkout.stripe.com/pay/cs_test_abc123",
  "status": "pending"
}
```
#### POST /api/payments/webhook

#### Input (from payment gateway):

```{
  "event": "payment_intent.succeeded",
  "data": {
    "booking_id": "b789",
    "amount": 750,
    "currency": "USD",
    "transaction_id": "txn_987654"
  }
}
```


#### Output (Success 200):
```
{
  "message": "Payment confirmed",
  "booking_status": "paid"
}
```

### Validation Rules

* Payment amount must match the total booking price.

* Booking must exist and have a status of “confirmed” before payment.

* Webhook requests must include valid signature for security verification.

* Handle failed or canceled payments with proper status updates.

### Performance Criteria

* Payment initiation response time ≤ 2 seconds.

* Webhook processing and booking update ≤ 1 second.

* System should support 100+ concurrent payments securely.

* Ensure 99.9% payment transaction accuracy through data integrity checks.

### Data Flow Summary

* Guest initiates payment for a confirmed booking.

* Backend calls the payment gateway API to generate a payment URL.

* Guest completes the payment on the gateway.

* Gateway sends webhook to the backend for confirmation.

* Backend updates booking and property availability.

* A payment receipt is generated and stored in the database.
