# AUTHENTICATION & AUTHORIZATION - INTERVIEW PREPARATION

## 1. Core Concepts

### Authentication vs Authorization

**Authentication (AuthN)**: "Who are you?"
- Process của **verifying identity** của user
- Xác minh credentials (username/password, token, biometrics)
- User proves họ là ai họ claim

**Authorization (AuthZ)**: "What can you do?"
- Process của **determining permissions** sau khi authenticated
- Kiểm tra user có quyền access specific resources không
- Based on roles, attributes, policies

```javascript
// Authentication: Verify user identity
const authenticate = async (username, password) => {
  const user = await User.findOne({ username });
  if (!user || !await bcrypt.compare(password, user.passwordHash)) {
    throw new Error('Invalid credentials');
  }
  return user; // User authenticated
};

// Authorization: Check permissions
const authorize = (requiredRole) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Not authenticated' });
    }
    
    if (!req.user.roles.includes(requiredRole)) {
      return res.status(403).json({ error: 'Not authorized' });
    }
    
    next();
  };
};
```

## 2. Authentication Methods

### Session-Based Authentication

**How it works:**
1. User submits credentials
2. Server verifies và tạo session
3. Session ID stored in cookie
4. Subsequent requests send cookie
5. Server validates session

```javascript
// Session-based authentication implementation
const express = require('express');
const session = require('express-session');
const bcrypt = require('bcrypt');

const app = express();

// Session configuration
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production', // HTTPS only in production
    httpOnly: true, // Prevent XSS
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  },
  store: new RedisStore({ client: redisClient }) // Use Redis for production
}));

// Login endpoint
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  
  try {
    const user = await User.findOne({ username });
    
    if (!user || !await bcrypt.compare(password, user.passwordHash)) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Create session
    req.session.userId = user.id;
    req.session.user = {
      id: user.id,
      username: user.username,
      roles: user.roles
    };
    
    res.json({ message: 'Login successful', user: req.session.user });
  } catch (error) {
    res.status(500).json({ error: 'Login failed' });
  }
});

// Protected route middleware
const requireAuth = (req, res, next) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  next();
};

// Logout
app.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ error: 'Logout failed' });
    }
    res.clearCookie('connect.sid');
    res.json({ message: 'Logout successful' });
  });
});

// Protected route
app.get('/profile', requireAuth, (req, res) => {
  res.json({ user: req.session.user });
});
```

### JWT (JSON Web Tokens)

**Structure:**
- **Header**: Algorithm và token type
- **Payload**: Claims (data)
- **Signature**: Verification signature

```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

// JWT Configuration
const JWT_CONFIG = {
  secret: process.env.JWT_SECRET,
  accessTokenExpiry: '15m',
  refreshTokenExpiry: '7d'
};

// JWT Authentication Service
class JWTAuthService {
  generateTokens(user) {
    const payload = {
      userId: user.id,
      username: user.username,
      roles: user.roles,
      permissions: user.permissions
    };
    
    const accessToken = jwt.sign(payload, JWT_CONFIG.secret, {
      expiresIn: JWT_CONFIG.accessTokenExpiry,
      issuer: 'your-app',
      audience: 'your-app-users'
    });
    
    const refreshToken = jwt.sign(
      { userId: user.id, tokenType: 'refresh' },
      JWT_CONFIG.secret,
      { expiresIn: JWT_CONFIG.refreshTokenExpiry }
    );
    
    return { accessToken, refreshToken };
  }
  
  verifyToken(token) {
    try {
      return jwt.verify(token, JWT_CONFIG.secret);
    } catch (error) {
      if (error.name === 'TokenExpiredError') {
        throw new Error('Token expired');
      } else if (error.name === 'JsonWebTokenError') {
        throw new Error('Invalid token');
      }
      throw error;
    }
  }
  
  async refreshAccessToken(refreshToken) {
    try {
      const decoded = this.verifyToken(refreshToken);
      
      if (decoded.tokenType !== 'refresh') {
        throw new Error('Invalid refresh token');
      }
      
      const user = await User.findById(decoded.userId);
      if (!user) {
        throw new Error('User not found');
      }
      
      // Generate new access token
      return this.generateTokens(user);
    } catch (error) {
      throw new Error('Token refresh failed');
    }
  }
}

// JWT Middleware
const jwtAuth = new JWTAuthService();

const verifyJWT = async (req, res, next) => {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  const token = authHeader.split(' ')[1];
  
  try {
    const decoded = jwtAuth.verifyToken(token);
    req.user = decoded;
    next();
  } catch (error) {
    if (error.message === 'Token expired') {
      return res.status(401).json({ 
        error: 'Token expired',
        code: 'TOKEN_EXPIRED'
      });
    }
    return res.status(403).json({ error: 'Invalid token' });
  }
};

// Login endpoint with JWT
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  
  try {
    const user = await User.findOne({ username });
    
    if (!user || !await bcrypt.compare(password, user.passwordHash)) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    const tokens = jwtAuth.generateTokens(user);
    
    // Set refresh token as httpOnly cookie
    res.cookie('refreshToken', tokens.refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
    });
    
    res.json({
      accessToken: tokens.accessToken,
      user: {
        id: user.id,
        username: user.username,
        roles: user.roles
      }
    });
  } catch (error) {
    res.status(500).json({ error: 'Login failed' });
  }
});

// Token refresh endpoint
app.post('/refresh', async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  
  if (!refreshToken) {
    return res.status(401).json({ error: 'No refresh token' });
  }
  
  try {
    const tokens = await jwtAuth.refreshAccessToken(refreshToken);
    
    // Update refresh token cookie
    res.cookie('refreshToken', tokens.refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000
    });
    
    res.json({ accessToken: tokens.accessToken });
  } catch (error) {
    res.status(401).json({ error: error.message });
  }
});
```

### OAuth 2.0 Implementation

**OAuth 2.0 Flows:**
1. **Authorization Code Flow** - most secure, for server-side apps
2. **PKCE Flow** - for SPAs and mobile apps
3. **Client Credentials Flow** - for machine-to-machine

```javascript
// OAuth 2.0 Authorization Code Flow
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

// Google OAuth Strategy
passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: '/auth/google/callback'
}, async (accessToken, refreshToken, profile, done) => {
  try {
    // Check if user exists
    let user = await User.findOne({ googleId: profile.id });
    
    if (!user) {
      // Create new user
      user = await User.create({
        googleId: profile.id,
        username: profile.emails[0].value,
        email: profile.emails[0].value,
        name: profile.displayName,
        provider: 'google',
        avatar: profile.photos[0].value
      });
    }
    
    return done(null, user);
  } catch (error) {
    return done(error, null);
  }
}));

// OAuth routes
app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] })
);

app.get('/auth/google/callback',
  passport.authenticate('google', { session: false }),
  (req, res) => {
    // Generate JWT for authenticated user
    const tokens = jwtAuth.generateTokens(req.user);
    
    // Redirect to frontend with token
    res.redirect(`${process.env.FRONTEND_URL}/auth/callback?token=${tokens.accessToken}`);
  }
);

// Custom OAuth 2.0 Server Implementation
class OAuth2Server {
  // Authorization endpoint
  async authorize(req, res) {
    const {
      client_id,
      redirect_uri,
      response_type,
      scope,
      state,
      code_challenge,
      code_challenge_method
    } = req.query;
    
    // Validate client
    const client = await OAuthClient.findOne({ clientId: client_id });
    if (!client) {
      return res.status(400).json({ error: 'invalid_client' });
    }
    
    // Validate redirect URI
    if (!client.redirectUris.includes(redirect_uri)) {
      return res.status(400).json({ error: 'invalid_redirect_uri' });
    }
    
    // Generate authorization code
    const authCode = {
      code: generateRandomString(32),
      clientId: client_id,
      userId: req.user.id,
      redirectUri: redirect_uri,
      scope: scope,
      expiresAt: Date.now() + 10 * 60 * 1000, // 10 minutes
      codeChallenge: code_challenge,
      codeChallengeMethod: code_challenge_method
    };
    
    await AuthCode.create(authCode);
    
    // Redirect with code
    const redirectUrl = new URL(redirect_uri);
    redirectUrl.searchParams.append('code', authCode.code);
    if (state) redirectUrl.searchParams.append('state', state);
    
    res.redirect(redirectUrl.toString());
  }
  
  // Token endpoint
  async token(req, res) {
    const {
      grant_type,
      code,
      client_id,
      client_secret,
      redirect_uri,
      code_verifier
    } = req.body;
    
    if (grant_type !== 'authorization_code') {
      return res.status(400).json({ error: 'unsupported_grant_type' });
    }
    
    // Validate client
    const client = await OAuthClient.findOne({
      clientId: client_id,
      clientSecret: client_secret
    });
    
    if (!client) {
      return res.status(401).json({ error: 'invalid_client' });
    }
    
    // Validate authorization code
    const authCode = await AuthCode.findOne({ code });
    
    if (!authCode || authCode.expiresAt < Date.now()) {
      return res.status(400).json({ error: 'invalid_grant' });
    }
    
    // PKCE validation
    if (authCode.codeChallenge) {
      const challenge = base64url(crypto.createHash('sha256').update(code_verifier).digest());
      if (challenge !== authCode.codeChallenge) {
        return res.status(400).json({ error: 'invalid_grant' });
      }
    }
    
    // Generate tokens
    const user = await User.findById(authCode.userId);
    const accessToken = jwt.sign(
      {
        userId: user.id,
        clientId: client_id,
        scope: authCode.scope
      },
      JWT_CONFIG.secret,
      { expiresIn: '1h' }
    );
    
    const refreshToken = jwt.sign(
      {
        userId: user.id,
        clientId: client_id,
        tokenType: 'refresh'
      },
      JWT_CONFIG.secret,
      { expiresIn: '30d' }
    );
    
    // Delete used authorization code
    await AuthCode.deleteOne({ code });
    
    res.json({
      access_token: accessToken,
      refresh_token: refreshToken,
      token_type: 'Bearer',
      expires_in: 3600,
      scope: authCode.scope
    });
  }
}
```

### Multi-Factor Authentication (MFA)

```javascript
const speakeasy = require('speakeasy');
const qrcode = require('qrcode');

class MFAService {
  // Generate TOTP secret for user
  async generateTOTPSecret(userId) {
    const secret = speakeasy.generateSecret({
      name: `YourApp (${userId})`,
      issuer: 'YourApp',
      length: 20
    });
    
    // Save secret to user
    await User.findByIdAndUpdate(userId, {
      mfaSecret: secret.base32,
      mfaEnabled: false
    });
    
    // Generate QR code
    const qrCodeUrl = await qrcode.toDataURL(secret.otpauth_url);
    
    return {
      secret: secret.base32,
      qrCode: qrCodeUrl,
      manualEntryKey: secret.base32
    };
  }
  
  // Verify TOTP token
  verifyTOTP(secret, token) {
    return speakeasy.totp.verify({
      secret: secret,
      encoding: 'base32',
      token: token,
      window: 2 // Allow 2-step window for clock drift
    });
  }
  
  // Enable MFA
  async enableMFA(userId, token) {
    const user = await User.findById(userId);
    
    if (!user.mfaSecret) {
      throw new Error('MFA secret not generated');
    }
    
    const isValid = this.verifyTOTP(user.mfaSecret, token);
    if (!isValid) {
      throw new Error('Invalid MFA token');
    }
    
    await User.findByIdAndUpdate(userId, { mfaEnabled: true });
    return true;
  }
  
  // Generate backup codes
  async generateBackupCodes(userId) {
    const codes = Array.from({ length: 10 }, () => 
      Math.random().toString(36).substring(2, 8).toUpperCase()
    );
    
    const hashedCodes = await Promise.all(
      codes.map(code => bcrypt.hash(code, 10))
    );
    
    await User.findByIdAndUpdate(userId, {
      backupCodes: hashedCodes
    });
    
    return codes; // Return plain codes to user (one time only)
  }
}

// MFA-enabled login
app.post('/login', async (req, res) => {
  const { username, password, mfaToken, backupCode } = req.body;
  
  try {
    const user = await User.findOne({ username });
    
    if (!user || !await bcrypt.compare(password, user.passwordHash)) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Check if MFA is enabled
    if (user.mfaEnabled) {
      let mfaValid = false;
      
      // Check TOTP token
      if (mfaToken) {
        mfaValid = mfaService.verifyTOTP(user.mfaSecret, mfaToken);
      }
      
      // Check backup code
      if (!mfaValid && backupCode) {
        for (let i = 0; i < user.backupCodes.length; i++) {
          if (await bcrypt.compare(backupCode, user.backupCodes[i])) {
            mfaValid = true;
            // Remove used backup code
            user.backupCodes.splice(i, 1);
            await user.save();
            break;
          }
        }
      }
      
      if (!mfaValid) {
        return res.status(401).json({ 
          error: 'MFA required',
          requiresMFA: true 
        });
      }
    }
    
    // Generate JWT tokens
    const tokens = jwtAuth.generateTokens(user);
    
    res.json({
      accessToken: tokens.accessToken,
      user: {
        id: user.id,
        username: user.username,
        mfaEnabled: user.mfaEnabled
      }
    });
  } catch (error) {
    res.status(500).json({ error: 'Login failed' });
  }
});
```

## 3. Authorization Patterns

### Role-Based Access Control (RBAC)

```javascript
// RBAC Implementation
class RBACService {
  constructor() {
    this.roles = new Map();
    this.permissions = new Map();
    
    this.initializeRoles();
  }
  
  initializeRoles() {
    // Define permissions
    const permissions = [
      'users:read', 'users:write', 'users:delete',
      'posts:read', 'posts:write', 'posts:delete',
      'admin:dashboard', 'admin:settings'
    ];
    
    // Define roles with permissions
    this.roles.set('admin', [
      'users:read', 'users:write', 'users:delete',
      'posts:read', 'posts:write', 'posts:delete',
      'admin:dashboard', 'admin:settings'
    ]);
    
    this.roles.set('moderator', [
      'users:read', 'posts:read', 'posts:write', 'posts:delete'
    ]);
    
    this.roles.set('user', [
      'posts:read'
    ]);
  }
  
  hasPermission(userRoles, requiredPermission) {
    for (const role of userRoles) {
      const rolePermissions = this.roles.get(role) || [];
      if (rolePermissions.includes(requiredPermission)) {
        return true;
      }
    }
    return false;
  }
  
  hasRole(userRoles, requiredRole) {
    return userRoles.includes(requiredRole);
  }
}

const rbac = new RBACService();

// Authorization middleware
const requirePermission = (permission) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    
    if (!rbac.hasPermission(req.user.roles, permission)) {
      return res.status(403).json({ 
        error: 'Insufficient permissions',
        required: permission
      });
    }
    
    next();
  };
};

const requireRole = (role) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    
    if (!rbac.hasRole(req.user.roles, role)) {
      return res.status(403).json({ 
        error: 'Insufficient role',
        required: role
      });
    }
    
    next();
  };
};

// Usage examples
app.get('/users', verifyJWT, requirePermission('users:read'), getUsersController);
app.post('/users', verifyJWT, requirePermission('users:write'), createUserController);
app.delete('/users/:id', verifyJWT, requirePermission('users:delete'), deleteUserController);
app.get('/admin/dashboard', verifyJWT, requireRole('admin'), adminDashboardController);
```

### Attribute-Based Access Control (ABAC)

```javascript
// ABAC Implementation
class ABACService {
  constructor() {
    this.policies = [];
    this.initializePolicies();
  }
  
  initializePolicies() {
    this.policies = [
      // Users can read their own profile
      {
        subject: { roles: ['user'] },
        resource: { type: 'profile' },
        action: 'read',
        condition: (context) => context.user.id === context.resource.userId
      },
      
      // Users can edit their own posts
      {
        subject: { roles: ['user'] },
        resource: { type: 'post' },
        action: 'write',
        condition: (context) => context.user.id === context.resource.authorId
      },
      
      // Moderators can edit posts in their assigned categories
      {
        subject: { roles: ['moderator'] },
        resource: { type: 'post' },
        action: 'write',
        condition: (context) => 
          context.user.assignedCategories?.includes(context.resource.categoryId)
      },
      
      // Admins can do anything
      {
        subject: { roles: ['admin'] },
        resource: { type: '*' },
        action: '*',
        condition: () => true
      },
      
      // Time-based access
      {
        subject: { roles: ['employee'] },
        resource: { type: 'system' },
        action: 'access',
        condition: (context) => {
          const hour = new Date().getHours();
          return hour >= 9 && hour <= 17; // 9 AM to 5 PM
        }
      }
    ];
  }
  
  async evaluate(user, resource, action) {
    const context = { user, resource, action };
    
    for (const policy of this.policies) {
      if (this.matchesPolicy(policy, context)) {
        try {
          const allowed = await policy.condition(context);
          if (allowed) {
            return { allowed: true, policy };
          }
        } catch (error) {
          console.error('Policy evaluation error:', error);
        }
      }
    }
    
    return { allowed: false };
  }
  
  matchesPolicy(policy, context) {
    // Check subject (user) attributes
    if (policy.subject.roles) {
      const hasRole = policy.subject.roles.some(role => 
        context.user.roles?.includes(role)
      );
      if (!hasRole) return false;
    }
    
    // Check resource attributes
    if (policy.resource.type !== '*' && 
        policy.resource.type !== context.resource.type) {
      return false;
    }
    
    // Check action
    if (policy.action !== '*' && policy.action !== context.action) {
      return false;
    }
    
    return true;
  }
}

const abac = new ABACService();

// ABAC middleware
const checkAccess = (resourceType, action) => {
  return async (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    
    // Build resource context
    const resource = {
      type: resourceType,
      id: req.params.id,
      ...req.resourceData // Additional resource attributes
    };
    
    const result = await abac.evaluate(req.user, resource, action);
    
    if (!result.allowed) {
      return res.status(403).json({ 
        error: 'Access denied',
        resource: resourceType,
        action: action
      });
    }
    
    next();
  };
};

// Middleware to load resource data
const loadResource = (model) => {
  return async (req, res, next) => {
    try {
      const resource = await model.findById(req.params.id);
      if (!resource) {
        return res.status(404).json({ error: 'Resource not found' });
      }
      req.resourceData = resource.toJSON();
      next();
    } catch (error) {
      res.status(500).json({ error: 'Failed to load resource' });
    }
  };
};

// Usage examples
app.get('/posts/:id', 
  verifyJWT, 
  loadResource(Post),
  checkAccess('post', 'read'),
  getPostController
);

app.put('/posts/:id',
  verifyJWT,
  loadResource(Post),
  checkAccess('post', 'write'),
  updatePostController
);
```

## 4. Security Best Practices

### Token Storage & Security

```javascript
// Secure token handling
class SecureTokenService {
  // Store sensitive data in httpOnly cookies
  setSecureTokenCookie(res, name, token, maxAge = 24 * 60 * 60 * 1000) {
    res.cookie(name, token, {
      httpOnly: true,        // Prevent XSS
      secure: process.env.NODE_ENV === 'production', // HTTPS only in prod
      sameSite: 'strict',    // CSRF protection
      maxAge: maxAge,
      signed: true           // Cookie signing
    });
  }
  
  // Generate CSRF token
  generateCSRFToken() {
    return crypto.randomBytes(32).toString('hex');
  }
  
  // CSRF protection middleware
  csrfProtection() {
    return (req, res, next) => {
      if (['POST', 'PUT', 'DELETE', 'PATCH'].includes(req.method)) {
        const csrfToken = req.headers['x-csrf-token'] || req.body._csrf;
        const sessionCSRF = req.session.csrfToken;
        
        if (!csrfToken || !sessionCSRF || csrfToken !== sessionCSRF) {
          return res.status(403).json({ error: 'Invalid CSRF token' });
        }
      }
      next();
    };
  }
}
```

### Rate Limiting & Brute Force Protection

```javascript
const rateLimit = require('express-rate-limit');
const slowDown = require('express-slow-down');

// Rate limiting configuration
const createRateLimiter = (windowMs, max, message) => {
  return rateLimit({
    windowMs,
    max,
    message: { error: message },
    standardHeaders: true,
    legacyHeaders: false,
    keyGenerator: (req) => {
      // Use IP + user ID if authenticated
      return req.user ? `${req.ip}:${req.user.id}` : req.ip;
    }
  });
};

// Different rate limits for different endpoints
const authLimiter = createRateLimiter(
  15 * 60 * 1000, // 15 minutes
  5,              // 5 attempts
  'Too many authentication attempts'
);

const apiLimiter = createRateLimiter(
  15 * 60 * 1000, // 15 minutes
  100,            // 100 requests
  'Too many API requests'
);

// Progressive delay for suspicious activity
const speedLimiter = slowDown({
  windowMs: 15 * 60 * 1000, // 15 minutes
  delayAfter: 2,            // Allow 2 requests per windowMs without delay
  delayMs: 500             // Add 500ms delay per request after delayAfter
});

// Apply rate limiting
app.use('/api/', apiLimiter);
app.use('/auth/', authLimiter, speedLimiter);

// Brute force protection with Redis
class BruteForceProtection {
  constructor(redisClient) {
    this.redis = redisClient;
    this.maxAttempts = 5;
    this.lockoutDuration = 15 * 60; // 15 minutes
  }
  
  async recordFailedAttempt(identifier) {
    const key = `failed_attempts:${identifier}`;
    const attempts = await this.redis.incr(key);
    
    if (attempts === 1) {
      await this.redis.expire(key, this.lockoutDuration);
    }
    
    return attempts;
  }
  
  async isBlocked(identifier) {
    const key = `failed_attempts:${identifier}`;
    const attempts = await this.redis.get(key);
    return attempts && parseInt(attempts) >= this.maxAttempts;
  }
  
  async reset(identifier) {
    const key = `failed_attempts:${identifier}`;
    await this.redis.del(key);
  }
  
  middleware() {
    return async (req, res, next) => {
      const identifier = req.ip;
      
      if (await this.isBlocked(identifier)) {
        return res.status(429).json({
          error: 'Too many failed attempts. Try again later.'
        });
      }
      
      next();
    };
  }
}
```

### Input Validation & Sanitization

```javascript
const Joi = require('joi');
const xss = require('xss');
const validator = require('validator');

// Input validation schemas
const validationSchemas = {
  register: Joi.object({
    username: Joi.string().alphanum().min(3).max(30).required(),
    email: Joi.string().email().required(),
    password: Joi.string().min(8).pattern(
      new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]')
    ).required()
  }),
  
  login: Joi.object({
    username: Joi.string().required(),
    password: Joi.string().required(),
    mfaToken: Joi.string().length(6).pattern(/^\d+$/).optional()
  })
};

// Validation middleware
const validateInput = (schema) => {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body);
    
    if (error) {
      return res.status(400).json({
        error: 'Validation failed',
        details: error.details.map(detail => detail.message)
      });
    }
    
    // Sanitize input
    req.body = sanitizeInput(value);
    next();
  };
};

// Input sanitization
const sanitizeInput = (input) => {
  if (typeof input === 'string') {
    return xss(validator.escape(input));
  }
  
  if (Array.isArray(input)) {
    return input.map(sanitizeInput);
  }
  
  if (typeof input === 'object' && input !== null) {
    const sanitized = {};
    for (const [key, value] of Object.entries(input)) {
      sanitized[key] = sanitizeInput(value);
    }
    return sanitized;
  }
  
  return input;
};

// Usage
app.post('/register', 
  validateInput(validationSchemas.register),
  registerController
);
```

## 5. Common Security Vulnerabilities

### Cross-Site Scripting (XSS) Protection

```javascript
// XSS Protection middleware
const xssProtection = () => {
  return (req, res, next) => {
    // Set security headers
    res.setHeader('X-XSS-Protection', '1; mode=block');
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'DENY');
    res.setHeader('Content-Security-Policy', 
      "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'"
    );
    
    next();
  };
};

app.use(xssProtection());
```

### SQL Injection Prevention

```javascript
// Always use parameterized queries
class SecureUserRepository {
  constructor(db) {
    this.db = db;
  }
  
  // ❌ Vulnerable to SQL injection
  async findUserUnsafe(username) {
    const query = `SELECT * FROM users WHERE username = '${username}'`;
    return await this.db.query(query);
  }
  
  // ✅ Safe parameterized query
  async findUserSafe(username) {
    const query = 'SELECT * FROM users WHERE username = $1';
    return await this.db.query(query, [username]);
  }
  
  // ✅ Using ORM (Sequelize example)
  async findUserORM(username) {
    return await User.findOne({
      where: { username } // Automatically escaped
    });
  }
}
```

### Session Hijacking Prevention

```javascript
// Session security configuration
const sessionConfig = {
  secret: process.env.SESSION_SECRET,
  name: 'sessionId', // Change default session name
  resave: false,
  saveUninitialized: false,
  rolling: true, // Reset expiry on each request
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 30 * 60 * 1000, // 30 minutes
    sameSite: 'strict'
  },
  genid: () => crypto.randomBytes(32).toString('hex') // Custom session ID generation
};

// Session hijacking protection
const sessionProtection = (req, res, next) => {
  if (req.session.userAgent && req.session.userAgent !== req.get('User-Agent')) {
    req.session.destroy();
    return res.status(401).json({ error: 'Session hijacking detected' });
  }
  
  if (req.session.ipAddress && req.session.ipAddress !== req.ip) {
    req.session.destroy();
    return res.status(401).json({ error: 'IP address changed' });
  }
  
  // Set session fingerprint
  if (req.session.userId) {
    req.session.userAgent = req.get('User-Agent');
    req.session.ipAddress = req.ip;
  }
  
  next();
};
```

## 6. Common Interview Questions

### 1. JWT vs Session-based authentication - Khi nào dùng gì?

**JWT:**
- **Pros**: Stateless, scalable, mobile-friendly, cross-domain
- **Cons**: Can't revoke immediately, larger size, security concerns if not handled properly
- **Use when**: Microservices, mobile apps, distributed systems

**Session-based:**
- **Pros**: Server-side control, can revoke immediately, smaller size
- **Cons**: Server state required, scaling challenges, not ideal for APIs
- **Use when**: Traditional web apps, need immediate revocation

### 2. Làm sao handle token refresh securely?

```javascript
// Secure token refresh strategy
class TokenRefreshService {
  async refreshTokens(refreshToken) {
    // 1. Verify refresh token
    const decoded = jwt.verify(refreshToken, JWT_SECRET);
    
    // 2. Check token family (detect token theft)
    const tokenFamily = await TokenFamily.findOne({ 
      userId: decoded.userId,
      family: decoded.family 
    });
    
    if (!tokenFamily || tokenFamily.isRevoked) {
      // Potential token theft - revoke all tokens
      await this.revokeAllUserTokens(decoded.userId);
      throw new Error('Token theft detected');
    }
    
    // 3. Generate new token pair with same family
    const newTokens = this.generateTokenPair(decoded.userId, decoded.family);
    
    // 4. Mark old refresh token as used
    await this.markTokenUsed(refreshToken);
    
    return newTokens;
  }
}
```

### 3. OAuth 2.0 vs OpenID Connect khác nhau như thế nào?

**OAuth 2.0:**
- **Authorization protocol** - "What can you access?"
- Focuses on granting access to resources
- Doesn't provide user identity information

**OpenID Connect:**
- **Authentication layer** built on top of OAuth 2.0
- "Who are you?" + "What can you access?"
- Provides ID tokens with user information
- Includes standardized user info endpoint

### 4. Làm sao implement role-based permissions hiệu quả?

```javascript
// Hierarchical role system
class HierarchicalRBAC {
  constructor() {
    this.roleHierarchy = {
      'super-admin': ['admin', 'moderator', 'user'],
      'admin': ['moderator', 'user'],
      'moderator': ['user'],
      'user': []
    };
  }
  
  getAllRoles(userRole) {
    return [userRole, ...(this.roleHierarchy[userRole] || [])];
  }
  
  hasPermission(userRoles, requiredPermission) {
    const allRoles = userRoles.flatMap(role => this.getAllRoles(role));
    return this.checkPermission(allRoles, requiredPermission);
  }
}
```

### 5. Cách prevent common security attacks?

**CSRF Prevention:**
- Use SameSite cookies
- CSRF tokens for state-changing operations
- Validate Origin/Referer headers

**XSS Prevention:**
- Input sanitization
- Content Security Policy
- HttpOnly cookies
- Output encoding

**Brute Force Prevention:**
- Rate limiting
- Account lockouts
- Progressive delays
- CAPTCHA after multiple failures

### 6. Multi-factor authentication implementation considerations?

```javascript
// MFA implementation considerations
const mfaConsiderations = {
  backup_codes: "Always provide backup codes for account recovery",
  time_synchronization: "Handle clock drift with time windows",
  rate_limiting: "Prevent MFA token brute force",
  user_experience: "Smooth onboarding and recovery flows",
  security_vs_usability: "Balance security requirements with UX"
};
```

## 7. Advanced Topics

### Zero Trust Architecture

```javascript
// Zero Trust implementation
class ZeroTrustMiddleware {
  async validateRequest(req, res, next) {
    // 1. Verify identity (authentication)
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    
    // 2. Verify device trust
    const deviceTrust = await this.verifyDeviceTrust(req);
    if (!deviceTrust.trusted) {
      return res.status(403).json({ error: 'Untrusted device' });
    }
    
    // 3. Verify network location
    const locationTrust = await this.verifyNetworkLocation(req.ip);
    if (!locationTrust.trusted) {
      return res.status(403).json({ error: 'Suspicious location' });
    }
    
    // 4. Verify behavior patterns
    const behaviorTrust = await this.verifyBehaviorPatterns(req.user.id, req);
    if (!behaviorTrust.trusted) {
      return res.status(403).json({ error: 'Suspicious behavior' });
    }
    
    // 5. Contextual authorization
    const contextualAuth = await this.evaluateContextualAuth(req);
    if (!contextualAuth.authorized) {
      return res.status(403).json({ error: 'Context-based access denied' });
    }
    
    next();
  }
}
```

### Passwordless Authentication

```javascript
// WebAuthn implementation
const webauthn = require('@webauthn/server');

class PasswordlessAuth {
  async generateRegistrationOptions(userId) {
    const user = await User.findById(userId);
    
    return await webauthn.generateRegistrationOptions({
      rpName: 'Your App',
      rpID: 'yourdomain.com',
      userID: user.id,
      userName: user.email,
      userDisplayName: user.name,
      attestationType: 'direct',
      authenticatorSelection: {
        authenticatorAttachment: 'platform',
        userVerification: 'required'
      }
    });
  }
  
  async verifyRegistration(userId, credential) {
    const verification = await webauthn.verifyRegistrationResponse({
      credential,
      expectedChallenge: user.currentChallenge,
      expectedOrigin: 'https://yourdomain.com',
      expectedRPID: 'yourdomain.com'
    });
    
    if (verification.verified) {
      // Save authenticator to user
      await User.findByIdAndUpdate(userId, {
        $push: {
          authenticators: {
            id: verification.registrationInfo.credentialID,
            publicKey: verification.registrationInfo.credentialPublicKey,
            counter: verification.registrationInfo.counter
          }
        }
      });
    }
    
    return verification.verified;
  }
}
```
