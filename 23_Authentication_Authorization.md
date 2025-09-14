# AUTHENTICATION & AUTHORIZATION

## Authentication Methods
- JWT (JSON Web Tokens)
- Session-based authentication  
- OAuth 2.0 / OpenID Connect
- Multi-factor Authentication (MFA)
- Passwordless authentication

## Authorization Patterns
- Role-based Access Control (RBAC)
- Attribute-based Access Control (ABAC)
- Resource-based permissions
- JWT claims and scopes

## Security Best Practices
- Token storage (HttpOnly cookies vs localStorage)
- Token refresh strategies
- CSRF protection
- Rate limiting
- Input validation & sanitization

## Code Examples
```javascript
// JWT middleware
const verifyToken = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid token' });
  }
};
```
