## 🧩 1. User Authentication Module
### Functional Requirements

* Users can register as either Guest or Host.

* Users can log in using email and password.

* The system must authenticate using JWT tokens.

* Passwords must be securely hashed (bcrypt or Argon2) before storage.

* Implement role-based access control (RBAC) (Admin, Host, Guest).

### API Endpoints
Method	Endpoint	Description
* POST	/api/auth/register  -	Register a new user (Guest or Host)
* POST	/api/auth/login	- Authenticate user credentials
* GET	/api/auth/profile	- Fetch user profile (JWT required)
* PUT	/api/auth/profile	- Update user profile
* Input / Output - Specification
* POST /api/auth/register

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
