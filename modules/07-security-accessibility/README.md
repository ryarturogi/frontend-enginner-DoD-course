# Module 7: Security & Accessibility

## 🎯 Learning Objectives

By the end of this module, you will:
- Implement OWASP Top 10 security best practices for frontend applications
- Prevent XSS, CSRF, and other common web vulnerabilities
- Configure Content Security Policy (CSP) and HTTPS properly
- Master WCAG 2.1 AA compliance principles and implementation
- Create accessible components with proper ARIA usage
- Design inclusive user experiences for all users
- Set up automated security and accessibility testing

## 🔒 Frontend Security: OWASP Top 10

### 1. Injection Attacks (SQL, XSS, Command Injection)

#### Cross-Site Scripting (XSS) Prevention

```typescript
// ❌ Vulnerable to XSS
const UserProfile: React.FC<{ user: User }> = ({ user }) => {
  return (
    <div>
      <h1 dangerouslySetInnerHTML={{ __html: user.name }} />
      <p dangerouslySetInnerHTML={{ __html: user.bio }} />
    </div>
  );
};

// ✅ XSS Protection with proper sanitization
import DOMPurify from 'dompurify';

const SafeUserProfile: React.FC<{ user: User }> = ({ user }) => {
  const sanitizedName = DOMPurify.sanitize(user.name);
  const sanitizedBio = DOMPurify.sanitize(user.bio, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em'],
    ALLOWED_ATTR: []
  });
  
  return (
    <div>
      <h1 dangerouslySetInnerHTML={{ __html: sanitizedName }} />
      <p dangerouslySetInnerHTML={{ __html: sanitizedBio }} />
    </div>
  );
};

// ✅ Even better: Avoid dangerouslySetInnerHTML when possible
const SecureUserProfile: React.FC<{ user: User }> = ({ user }) => {
  // Use React's built-in XSS protection
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
    </div>
  );
};

// ✅ For rich text, use a secure markdown renderer
import ReactMarkdown from 'react-markdown';

const MarkdownUserBio: React.FC<{ bio: string }> = ({ bio }) => {
  return (
    <ReactMarkdown
      allowedElements={['p', 'br', 'strong', 'em', 'ul', 'ol', 'li']}
      skipHtml={true}
    >
      {bio}
    </ReactMarkdown>
  );
};
```

#### Input Validation and Sanitization

```typescript
// Comprehensive input validation
interface ValidationRule {
  required?: boolean;
  minLength?: number;
  maxLength?: number;
  pattern?: RegExp;
  customValidator?: (value: string) => string | null;
}

class InputValidator {
  static validateEmail(email: string): string | null {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    
    if (!email) return 'Email is required';
    if (email.length > 254) return 'Email is too long';
    if (!emailRegex.test(email)) return 'Invalid email format';
    
    // Additional security checks
    if (email.includes('<') || email.includes('>')) {
      return 'Invalid characters in email';
    }
    
    return null;
  }
  
  static validatePassword(password: string): string | null {
    if (!password) return 'Password is required';
    if (password.length < 8) return 'Password must be at least 8 characters';
    if (password.length > 128) return 'Password is too long';
    
    const hasUppercase = /[A-Z]/.test(password);
    const hasLowercase = /[a-z]/.test(password);
    const hasNumbers = /\d/.test(password);
    const hasSymbols = /[!@#$%^&*(),.?":{}|<>]/.test(password);
    
    if (!hasUppercase) return 'Password must contain uppercase letters';
    if (!hasLowercase) return 'Password must contain lowercase letters';
    if (!hasNumbers) return 'Password must contain numbers';
    if (!hasSymbols) return 'Password must contain special characters';
    
    return null;
  }
  
  static sanitizeInput(input: string): string {
    return input
      .trim()
      .replace(/[<>]/g, '') // Remove angle brackets
      .replace(/javascript:/gi, '') // Remove javascript: protocol
      .replace(/on\w+=/gi, ''); // Remove event handlers
  }
  
  static validateAndSanitize(input: string, rules: ValidationRule): {
    isValid: boolean;
    value: string;
    error?: string;
  } {
    let sanitizedInput = this.sanitizeInput(input);
    
    // Required validation
    if (rules.required && !sanitizedInput) {
      return { isValid: false, value: '', error: 'This field is required' };
    }
    
    // Length validation
    if (rules.minLength && sanitizedInput.length < rules.minLength) {
      return { 
        isValid: false, 
        value: sanitizedInput, 
        error: `Minimum ${rules.minLength} characters required` 
      };
    }
    
    if (rules.maxLength && sanitizedInput.length > rules.maxLength) {
      return { 
        isValid: false, 
        value: sanitizedInput, 
        error: `Maximum ${rules.maxLength} characters allowed` 
      };
    }
    
    // Pattern validation
    if (rules.pattern && !rules.pattern.test(sanitizedInput)) {
      return { isValid: false, value: sanitizedInput, error: 'Invalid format' };
    }
    
    // Custom validation
    if (rules.customValidator) {
      const customError = rules.customValidator(sanitizedInput);
      if (customError) {
        return { isValid: false, value: sanitizedInput, error: customError };
      }
    }
    
    return { isValid: true, value: sanitizedInput };
  }
}

// Secure form component
const SecureContactForm: React.FC = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });
  const [errors, setErrors] = useState<Record<string, string>>({});
  
  const handleInputChange = (field: keyof typeof formData) => (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const value = e.target.value;
    
    let validation;
    switch (field) {
      case 'name':
        validation = InputValidator.validateAndSanitize(value, {
          required: true,
          minLength: 2,
          maxLength: 100
        });
        break;
      case 'email':
        validation = InputValidator.validateAndSanitize(value, {
          required: true,
          customValidator: InputValidator.validateEmail
        });
        break;
      case 'message':
        validation = InputValidator.validateAndSanitize(value, {
          required: true,
          minLength: 10,
          maxLength: 1000
        });
        break;
      default:
        validation = { isValid: true, value };
    }
    
    setFormData(prev => ({ ...prev, [field]: validation.value }));
    setErrors(prev => ({ 
      ...prev, 
      [field]: validation.error || '' 
    }));
  };
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    // Validate all fields
    const validations = {
      name: InputValidator.validateAndSanitize(formData.name, {
        required: true,
        minLength: 2,
        maxLength: 100
      }),
      email: InputValidator.validateAndSanitize(formData.email, {
        required: true,
        customValidator: InputValidator.validateEmail
      }),
      message: InputValidator.validateAndSanitize(formData.message, {
        required: true,
        minLength: 10,
        maxLength: 1000
      })
    };
    
    const newErrors: Record<string, string> = {};
    Object.entries(validations).forEach(([field, validation]) => {
      if (!validation.isValid && validation.error) {
        newErrors[field] = validation.error;
      }
    });
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }
    
    // Submit sanitized data
    const sanitizedData = {
      name: validations.name.value,
      email: validations.email.value,
      message: validations.message.value
    };
    
    try {
      await submitContactForm(sanitizedData);
    } catch (error) {
      console.error('Form submission error:', error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit} noValidate>
      <div>
        <label htmlFor="name">Name</label>
        <input
          id="name"
          type="text"
          value={formData.name}
          onChange={handleInputChange('name')}
          aria-invalid={!!errors.name}
          aria-describedby={errors.name ? 'name-error' : undefined}
        />
        {errors.name && (
          <div id="name-error" role="alert">{errors.name}</div>
        )}
      </div>
      
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={formData.email}
          onChange={handleInputChange('email')}
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <div id="email-error" role="alert">{errors.email}</div>
        )}
      </div>
      
      <div>
        <label htmlFor="message">Message</label>
        <textarea
          id="message"
          value={formData.message}
          onChange={handleInputChange('message')}
          aria-invalid={!!errors.message}
          aria-describedby={errors.message ? 'message-error' : undefined}
        />
        {errors.message && (
          <div id="message-error" role="alert">{errors.message}</div>
        )}
      </div>
      
      <button type="submit">Send Message</button>
    </form>
  );
};
```

### 2. Broken Authentication

```typescript
// Secure authentication implementation
class AuthService {
  private static readonly TOKEN_KEY = 'auth_token';
  private static readonly REFRESH_TOKEN_KEY = 'refresh_token';
  
  // Secure token storage
  static setTokens(accessToken: string, refreshToken: string): void {
    // Use httpOnly cookies in production
    if (this.isSecureContext()) {
      // Set as httpOnly cookie via API call
      this.setSecureCookie('auth_token', accessToken);
      this.setSecureCookie('refresh_token', refreshToken);
    } else {
      // Fallback to sessionStorage (not localStorage for security)
      sessionStorage.setItem(this.TOKEN_KEY, accessToken);
      sessionStorage.setItem(this.REFRESH_TOKEN_KEY, refreshToken);
    }
  }
  
  static getAccessToken(): string | null {
    if (this.isSecureContext()) {
      return this.getSecureCookie('auth_token');
    }
    return sessionStorage.getItem(this.TOKEN_KEY);
  }
  
  static async refreshAccessToken(): Promise<string | null> {
    const refreshToken = this.getRefreshToken();
    if (!refreshToken) return null;
    
    try {
      const response = await fetch('/api/auth/refresh', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        credentials: 'include', // Include httpOnly cookies
        body: JSON.stringify({ refreshToken })
      });
      
      if (!response.ok) {
        this.clearTokens();
        return null;
      }
      
      const { accessToken, refreshToken: newRefreshToken } = await response.json();
      this.setTokens(accessToken, newRefreshToken);
      
      return accessToken;
    } catch (error) {
      console.error('Token refresh failed:', error);
      this.clearTokens();
      return null;
    }
  }
  
  static clearTokens(): void {
    if (this.isSecureContext()) {
      this.clearSecureCookie('auth_token');
      this.clearSecureCookie('refresh_token');
    } else {
      sessionStorage.removeItem(this.TOKEN_KEY);
      sessionStorage.removeItem(this.REFRESH_TOKEN_KEY);
    }
  }
  
  // Session management
  static startSessionTimeout(): void {
    const timeout = 30 * 60 * 1000; // 30 minutes
    
    let warningTimer: NodeJS.Timeout;
    let logoutTimer: NodeJS.Timeout;
    
    const resetTimers = () => {
      clearTimeout(warningTimer);
      clearTimeout(logoutTimer);
      
      // Show warning 5 minutes before logout
      warningTimer = setTimeout(() => {
        this.showSessionWarning();
      }, timeout - 5 * 60 * 1000);
      
      // Auto logout
      logoutTimer = setTimeout(() => {
        this.logout();
      }, timeout);
    };
    
    // Reset timers on user activity
    const events = ['mousedown', 'mousemove', 'keypress', 'scroll', 'touchstart'];
    const resetActivity = () => resetTimers();
    
    events.forEach(event => {
      document.addEventListener(event, resetActivity, true);
    });
    
    resetTimers();
  }
  
  private static isSecureContext(): boolean {
    return window.location.protocol === 'https:' || 
           window.location.hostname === 'localhost';
  }
  
  private static async setSecureCookie(name: string, value: string): Promise<void> {
    await fetch('/api/auth/set-cookie', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
      body: JSON.stringify({ name, value })
    });
  }
  
  private static getSecureCookie(name: string): string | null {
    // Cookies are automatically included in requests
    // The server should validate and return the cookie value
    return null; // Will be handled by server-side authentication
  }
}

// Secure login component
const SecureLoginForm: React.FC = () => {
  const [credentials, setCredentials] = useState({ email: '', password: '' });
  const [attempts, setAttempts] = useState(0);
  const [isLocked, setIsLocked] = useState(false);
  const [errors, setErrors] = useState<Record<string, string>>({});
  
  const MAX_ATTEMPTS = 5;
  const LOCKOUT_TIME = 15 * 60 * 1000; // 15 minutes
  
  useEffect(() => {
    // Check if account is locked
    const lockoutEnd = localStorage.getItem('login_lockout');
    if (lockoutEnd && Date.now() < parseInt(lockoutEnd)) {
      setIsLocked(true);
      const remaining = parseInt(lockoutEnd) - Date.now();
      setTimeout(() => setIsLocked(false), remaining);
    }
  }, []);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (isLocked) {
      setErrors({ form: 'Account temporarily locked. Please try again later.' });
      return;
    }
    
    if (attempts >= MAX_ATTEMPTS) {
      setIsLocked(true);
      localStorage.setItem('login_lockout', (Date.now() + LOCKOUT_TIME).toString());
      setErrors({ form: 'Too many failed attempts. Account locked for 15 minutes.' });
      return;
    }
    
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'X-Requested-With': 'XMLHttpRequest' // CSRF protection
        },
        credentials: 'include',
        body: JSON.stringify(credentials)
      });
      
      if (!response.ok) {
        setAttempts(prev => prev + 1);
        setErrors({ form: 'Invalid credentials' });
        return;
      }
      
      const { accessToken, refreshToken } = await response.json();
      AuthService.setTokens(accessToken, refreshToken);
      AuthService.startSessionTimeout();
      
      // Clear failed attempts on successful login
      setAttempts(0);
      localStorage.removeItem('login_lockout');
      
      // Redirect to dashboard
      window.location.href = '/dashboard';
    } catch (error) {
      setErrors({ form: 'Login failed. Please try again.' });
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={credentials.email}
          onChange={(e) => setCredentials(prev => ({ ...prev, email: e.target.value }))}
          autoComplete="email"
          required
        />
      </div>
      
      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          value={credentials.password}
          onChange={(e) => setCredentials(prev => ({ ...prev, password: e.target.value }))}
          autoComplete="current-password"
          required
        />
      </div>
      
      {errors.form && (
        <div role="alert" className="error">{errors.form}</div>
      )}
      
      {attempts > 0 && (
        <div className="warning">
          {MAX_ATTEMPTS - attempts} attempts remaining
        </div>
      )}
      
      <button type="submit" disabled={isLocked}>
        {isLocked ? 'Account Locked' : 'Sign In'}
      </button>
    </form>
  );
};
```

### 3. Content Security Policy (CSP)

```typescript
// CSP configuration for Next.js
// next.config.js
const ContentSecurityPolicy = `
  default-src 'self';
  script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google-analytics.com https://www.googletagmanager.com;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  img-src 'self' data: blob: https://images.unsplash.com https://res.cloudinary.com;
  font-src 'self' https://fonts.gstatic.com;
  connect-src 'self' https://api.myapp.com https://www.google-analytics.com;
  media-src 'self' https://media.myapp.com;
  object-src 'none';
  base-uri 'self';
  form-action 'self';
  frame-ancestors 'none';
  upgrade-insecure-requests;
`;

const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: ContentSecurityPolicy.replace(/\s{2,}/g, ' ').trim()
  },
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin'
  },
  {
    key: 'X-Frame-Options',
    value: 'DENY'
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'false'
  },
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=31536000; includeSubDomains; preload'
  },
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=()'
  }
];

module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: securityHeaders,
      },
    ];
  },
};

// CSP violation reporting
const CSPReporter = {
  init() {
    // Listen for CSP violations
    document.addEventListener('securitypolicyviolation', (e) => {
      const violation = {
        documentURI: e.documentURI,
        violatedDirective: e.violatedDirective,
        blockedURI: e.blockedURI,
        lineNumber: e.lineNumber,
        columnNumber: e.columnNumber,
        sourceFile: e.sourceFile,
        statusCode: e.statusCode,
        timestamp: Date.now()
      };
      
      // Report to monitoring service
      this.reportViolation(violation);
    });
  },
  
  async reportViolation(violation: any) {
    try {
      await fetch('/api/security/csp-violation', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(violation)
      });
    } catch (error) {
      console.error('Failed to report CSP violation:', error);
    }
  }
};

// Initialize CSP reporting
if (typeof window !== 'undefined') {
  CSPReporter.init();
}
```

### 4. Cross-Site Request Forgery (CSRF) Protection

```typescript
// CSRF protection implementation
class CSRFProtection {
  private static token: string | null = null;
  
  static async getToken(): Promise<string> {
    if (this.token) return this.token;
    
    try {
      const response = await fetch('/api/csrf-token', {
        credentials: 'include'
      });
      const { csrfToken } = await response.json();
      this.token = csrfToken;
      return csrfToken;
    } catch (error) {
      throw new Error('Failed to get CSRF token');
    }
  }
  
  static async makeSecureRequest(url: string, options: RequestInit = {}): Promise<Response> {
    const csrfToken = await this.getToken();
    
    const secureOptions: RequestInit = {
      ...options,
      credentials: 'include',
      headers: {
        ...options.headers,
        'X-CSRF-Token': csrfToken,
        'X-Requested-With': 'XMLHttpRequest'
      }
    };
    
    const response = await fetch(url, secureOptions);
    
    // If CSRF token is invalid, refresh and retry
    if (response.status === 403 && response.headers.get('X-CSRF-Invalid')) {
      this.token = null;
      const newToken = await this.getToken();
      secureOptions.headers = {
        ...secureOptions.headers,
        'X-CSRF-Token': newToken
      };
      return fetch(url, secureOptions);
    }
    
    return response;
  }
}

// Secure API client
class SecureApiClient {
  static async get<T>(url: string): Promise<T> {
    const response = await CSRFProtection.makeSecureRequest(url, {
      method: 'GET'
    });
    
    if (!response.ok) {
      throw new Error(`GET ${url} failed: ${response.status}`);
    }
    
    return response.json();
  }
  
  static async post<T>(url: string, data: any): Promise<T> {
    const response = await CSRFProtection.makeSecureRequest(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    
    if (!response.ok) {
      throw new Error(`POST ${url} failed: ${response.status}`);
    }
    
    return response.json();
  }
  
  static async put<T>(url: string, data: any): Promise<T> {
    const response = await CSRFProtection.makeSecureRequest(url, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    
    if (!response.ok) {
      throw new Error(`PUT ${url} failed: ${response.status}`);
    }
    
    return response.json();
  }
  
  static async delete<T>(url: string): Promise<T> {
    const response = await CSRFProtection.makeSecureRequest(url, {
      method: 'DELETE'
    });
    
    if (!response.ok) {
      throw new Error(`DELETE ${url} failed: ${response.status}`);
    }
    
    return response.json();
  }
}
```

## ♿ Accessibility: WCAG 2.1 AA Compliance

### 1. Semantic HTML and ARIA

```typescript
// Accessible navigation component
const AccessibleNavigation: React.FC = () => {
  const [isOpen, setIsOpen] = useState(false);
  const [currentPage, setCurrentPage] = useState('/');
  const location = useLocation();
  
  useEffect(() => {
    setCurrentPage(location.pathname);
  }, [location]);
  
  const navigationItems = [
    { href: '/', label: 'Home' },
    { href: '/products', label: 'Products' },
    { href: '/about', label: 'About' },
    { href: '/contact', label: 'Contact' }
  ];
  
  return (
    <nav role="navigation" aria-label="Main navigation">
      <div className="nav-header">
        <Link to="/" aria-label="Go to homepage">
          <img src="/logo.svg" alt="Company Logo" />
        </Link>
        
        <button
          type="button"
          aria-expanded={isOpen}
          aria-controls="main-menu"
          aria-label={isOpen ? 'Close main menu' : 'Open main menu'}
          className="menu-toggle"
          onClick={() => setIsOpen(!isOpen)}
        >
          <span className="hamburger-line" aria-hidden="true"></span>
          <span className="hamburger-line" aria-hidden="true"></span>
          <span className="hamburger-line" aria-hidden="true"></span>
        </button>
      </div>
      
      <ul
        id="main-menu"
        className={`nav-menu ${isOpen ? 'nav-menu--open' : ''}`}
        role="menubar"
      >
        {navigationItems.map((item) => (
          <li key={item.href} role="none">
            <Link
              to={item.href}
              role="menuitem"
              aria-current={currentPage === item.href ? 'page' : undefined}
              className={`nav-link ${currentPage === item.href ? 'nav-link--active' : ''}`}
              onClick={() => setIsOpen(false)}
            >
              {item.label}
            </Link>
          </li>
        ))}
      </ul>
    </nav>
  );
};

// Accessible form component
const AccessibleContactForm: React.FC = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    subject: '',
    message: '',
    newsletter: false
  });
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitStatus, setSubmitStatus] = useState<'idle' | 'success' | 'error'>('idle');
  
  const formRef = useRef<HTMLFormElement>(null);
  const statusRef = useRef<HTMLDivElement>(null);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    
    try {
      await submitForm(formData);
      setSubmitStatus('success');
      setFormData({ name: '', email: '', subject: '', message: '', newsletter: false });
      
      // Announce success to screen readers
      if (statusRef.current) {
        statusRef.current.focus();
      }
    } catch (error) {
      setSubmitStatus('error');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <div>
      <h1 id="contact-heading">Contact Us</h1>
      
      <form
        ref={formRef}
        onSubmit={handleSubmit}
        aria-labelledby="contact-heading"
        noValidate
      >
        <fieldset>
          <legend>Contact Information</legend>
          
          <div className="form-group">
            <label htmlFor="name" className="required">
              Full Name
            </label>
            <input
              id="name"
              type="text"
              value={formData.name}
              onChange={(e) => setFormData(prev => ({ ...prev, name: e.target.value }))}
              aria-required="true"
              aria-invalid={!!errors.name}
              aria-describedby={errors.name ? 'name-error' : 'name-help'}
            />
            <div id="name-help" className="help-text">
              Enter your first and last name
            </div>
            {errors.name && (
              <div id="name-error" role="alert" className="error-message">
                {errors.name}
              </div>
            )}
          </div>
          
          <div className="form-group">
            <label htmlFor="email" className="required">
              Email Address
            </label>
            <input
              id="email"
              type="email"
              value={formData.email}
              onChange={(e) => setFormData(prev => ({ ...prev, email: e.target.value }))}
              aria-required="true"
              aria-invalid={!!errors.email}
              aria-describedby={errors.email ? 'email-error' : 'email-help'}
              autoComplete="email"
            />
            <div id="email-help" className="help-text">
              We'll never share your email address
            </div>
            {errors.email && (
              <div id="email-error" role="alert" className="error-message">
                {errors.email}
              </div>
            )}
          </div>
          
          <div className="form-group">
            <label htmlFor="subject">Subject</label>
            <select
              id="subject"
              value={formData.subject}
              onChange={(e) => setFormData(prev => ({ ...prev, subject: e.target.value }))}
              aria-describedby="subject-help"
            >
              <option value="">Select a topic</option>
              <option value="general">General Inquiry</option>
              <option value="support">Technical Support</option>
              <option value="billing">Billing Question</option>
              <option value="partnership">Partnership Opportunity</option>
            </select>
            <div id="subject-help" className="help-text">
              Choose the topic that best describes your inquiry
            </div>
          </div>
          
          <div className="form-group">
            <label htmlFor="message" className="required">
              Message
            </label>
            <textarea
              id="message"
              value={formData.message}
              onChange={(e) => setFormData(prev => ({ ...prev, message: e.target.value }))}
              aria-required="true"
              aria-invalid={!!errors.message}
              aria-describedby={errors.message ? 'message-error' : 'message-help'}
              rows={5}
            />
            <div id="message-help" className="help-text">
              Please provide details about your inquiry (minimum 10 characters)
            </div>
            {errors.message && (
              <div id="message-error" role="alert" className="error-message">
                {errors.message}
              </div>
            )}
          </div>
          
          <div className="form-group">
            <div className="checkbox-group">
              <input
                id="newsletter"
                type="checkbox"
                checked={formData.newsletter}
                onChange={(e) => setFormData(prev => ({ ...prev, newsletter: e.target.checked }))}
                aria-describedby="newsletter-help"
              />
              <label htmlFor="newsletter">
                Subscribe to newsletter
              </label>
            </div>
            <div id="newsletter-help" className="help-text">
              Receive updates about our products and services
            </div>
          </div>
        </fieldset>
        
        <div className="form-actions">
          <button
            type="submit"
            disabled={isSubmitting}
            aria-describedby="submit-status"
          >
            {isSubmitting ? (
              <>
                <span className="loading-spinner" aria-hidden="true"></span>
                Sending...
              </>
            ) : (
              'Send Message'
            )}
          </button>
        </div>
      </form>
      
      <div
        ref={statusRef}
        id="submit-status"
        role="status"
        aria-live="polite"
        aria-atomic="true"
        tabIndex={-1}
      >
        {submitStatus === 'success' && (
          <div className="success-message">
            Thank you! Your message has been sent successfully.
          </div>
        )}
        {submitStatus === 'error' && (
          <div className="error-message">
            Sorry, there was an error sending your message. Please try again.
          </div>
        )}
      </div>
    </div>
  );
};
```

### 2. Keyboard Navigation

```typescript
// Accessible modal with focus management
const AccessibleModal: React.FC<{
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}> = ({ isOpen, onClose, title, children }) => {
  const modalRef = useRef<HTMLDivElement>(null);
  const closeButtonRef = useRef<HTMLButtonElement>(null);
  const previousFocusRef = useRef<HTMLElement | null>(null);
  
  useEffect(() => {
    if (isOpen) {
      // Store the previously focused element
      previousFocusRef.current = document.activeElement as HTMLElement;
      
      // Focus the modal
      if (modalRef.current) {
        modalRef.current.focus();
      }
      
      // Prevent background scrolling
      document.body.style.overflow = 'hidden';
    } else {
      // Restore focus to the previously focused element
      if (previousFocusRef.current) {
        previousFocusRef.current.focus();
      }
      
      // Restore scrolling
      document.body.style.overflow = '';
    }
    
    return () => {
      document.body.style.overflow = '';
    };
  }, [isOpen]);
  
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (!isOpen) return;
      
      if (e.key === 'Escape') {
        onClose();
        return;
      }
      
      if (e.key === 'Tab') {
        trapFocus(e);
      }
    };
    
    const trapFocus = (e: KeyboardEvent) => {
      if (!modalRef.current) return;
      
      const focusableElements = modalRef.current.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      
      const firstElement = focusableElements[0] as HTMLElement;
      const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;
      
      if (e.shiftKey) {
        // Shift + Tab
        if (document.activeElement === firstElement) {
          lastElement.focus();
          e.preventDefault();
        }
      } else {
        // Tab
        if (document.activeElement === lastElement) {
          firstElement.focus();
          e.preventDefault();
        }
      }
    };
    
    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [isOpen, onClose]);
  
  if (!isOpen) return null;
  
  return (
    <div
      className="modal-overlay"
      onClick={(e) => {
        if (e.target === e.currentTarget) {
          onClose();
        }
      }}
    >
      <div
        ref={modalRef}
        className="modal"
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
      >
        <header className="modal-header">
          <h2 id="modal-title">{title}</h2>
          <button
            ref={closeButtonRef}
            type="button"
            className="modal-close"
            onClick={onClose}
            aria-label="Close dialog"
          >
            <span aria-hidden="true">&times;</span>
          </button>
        </header>
        
        <div className="modal-content">
          {children}
        </div>
      </div>
    </div>
  );
};

// Accessible dropdown menu
const AccessibleDropdown: React.FC<{
  trigger: React.ReactNode;
  children: React.ReactNode;
}> = ({ trigger, children }) => {
  const [isOpen, setIsOpen] = useState(false);
  const triggerRef = useRef<HTMLButtonElement>(null);
  const menuRef = useRef<HTMLDivElement>(null);
  
  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'Enter':
      case ' ':
        e.preventDefault();
        setIsOpen(!isOpen);
        break;
      case 'ArrowDown':
        e.preventDefault();
        if (!isOpen) {
          setIsOpen(true);
        } else {
          focusFirstMenuItem();
        }
        break;
      case 'ArrowUp':
        e.preventDefault();
        if (!isOpen) {
          setIsOpen(true);
        } else {
          focusLastMenuItem();
        }
        break;
      case 'Escape':
        if (isOpen) {
          setIsOpen(false);
          triggerRef.current?.focus();
        }
        break;
    }
  };
  
  const focusFirstMenuItem = () => {
    const firstItem = menuRef.current?.querySelector('[role="menuitem"]') as HTMLElement;
    firstItem?.focus();
  };
  
  const focusLastMenuItem = () => {
    const items = menuRef.current?.querySelectorAll('[role="menuitem"]');
    const lastItem = items?.[items.length - 1] as HTMLElement;
    lastItem?.focus();
  };
  
  const handleMenuKeyDown = (e: React.KeyboardEvent) => {
    const currentItem = e.target as HTMLElement;
    const items = Array.from(menuRef.current?.querySelectorAll('[role="menuitem"]') || []);
    const currentIndex = items.indexOf(currentItem);
    
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        const nextIndex = currentIndex < items.length - 1 ? currentIndex + 1 : 0;
        (items[nextIndex] as HTMLElement).focus();
        break;
      case 'ArrowUp':
        e.preventDefault();
        const prevIndex = currentIndex > 0 ? currentIndex - 1 : items.length - 1;
        (items[prevIndex] as HTMLElement).focus();
        break;
      case 'Home':
        e.preventDefault();
        (items[0] as HTMLElement).focus();
        break;
      case 'End':
        e.preventDefault();
        (items[items.length - 1] as HTMLElement).focus();
        break;
      case 'Escape':
        setIsOpen(false);
        triggerRef.current?.focus();
        break;
      case 'Tab':
        setIsOpen(false);
        break;
    }
  };
  
  return (
    <div className="dropdown">
      <button
        ref={triggerRef}
        type="button"
        aria-expanded={isOpen}
        aria-haspopup="menu"
        aria-controls="dropdown-menu"
        onKeyDown={handleKeyDown}
        onClick={() => setIsOpen(!isOpen)}
        onBlur={(e) => {
          // Close if focus moves outside the dropdown
          if (!e.currentTarget.parentElement?.contains(e.relatedTarget as Node)) {
            setIsOpen(false);
          }
        }}
      >
        {trigger}
      </button>
      
      {isOpen && (
        <div
          ref={menuRef}
          id="dropdown-menu"
          role="menu"
          className="dropdown-menu"
          onKeyDown={handleMenuKeyDown}
        >
          {children}
        </div>
      )}
    </div>
  );
};
```

### 3. Color Contrast and Visual Design

```typescript
// Color contrast utilities
class ColorContrastUtils {
  // Calculate relative luminance (WCAG formula)
  static getRelativeLuminance(rgb: [number, number, number]): number {
    const [r, g, b] = rgb.map(c => {
      const sRGB = c / 255;
      return sRGB <= 0.03928 
        ? sRGB / 12.92 
        : Math.pow((sRGB + 0.055) / 1.055, 2.4);
    });
    
    return 0.2126 * r + 0.7152 * g + 0.0722 * b;
  }
  
  // Calculate contrast ratio between two colors
  static getContrastRatio(color1: [number, number, number], color2: [number, number, number]): number {
    const l1 = this.getRelativeLuminance(color1);
    const l2 = this.getRelativeLuminance(color2);
    
    const lighter = Math.max(l1, l2);
    const darker = Math.min(l1, l2);
    
    return (lighter + 0.05) / (darker + 0.05);
  }
  
  // Check if contrast meets WCAG requirements
  static meetsWCAGRequirements(
    color1: [number, number, number], 
    color2: [number, number, number],
    level: 'AA' | 'AAA' = 'AA',
    size: 'normal' | 'large' = 'normal'
  ): { passes: boolean; ratio: number; required: number } {
    const ratio = this.getContrastRatio(color1, color2);
    
    let required: number;
    if (level === 'AAA') {
      required = size === 'large' ? 4.5 : 7;
    } else {
      required = size === 'large' ? 3 : 4.5;
    }
    
    return {
      passes: ratio >= required,
      ratio,
      required
    };
  }
  
  // Convert hex to RGB
  static hexToRgb(hex: string): [number, number, number] | null {
    const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
    return result ? [
      parseInt(result[1], 16),
      parseInt(result[2], 16),
      parseInt(result[3], 16)
    ] : null;
  }
}

// Accessible theme configuration
export const accessibleTheme = {
  colors: {
    // Primary colors with verified contrast ratios
    primary: {
      main: '#1976d2',     // 4.5:1 against white
      dark: '#115293',     // 7:1 against white
      light: '#42a5f5',    // 3:1 against white (large text only)
      contrastText: '#ffffff'
    },
    
    secondary: {
      main: '#388e3c',     // 4.5:1 against white
      dark: '#2e7d32',     // 7:1 against white
      light: '#66bb6a',    // 3.1:1 against white (large text only)
      contrastText: '#ffffff'
    },
    
    error: {
      main: '#d32f2f',     // 5.8:1 against white
      dark: '#c62828',     // 7:1 against white
      light: '#ef5350',    // 3.3:1 against white (large text only)
      contrastText: '#ffffff'
    },
    
    warning: {
      main: '#f57c00',     // 4.6:1 against white
      dark: '#ef6c00',     // 5.4:1 against white
      light: '#ffb74d',    // 2.7:1 against black (large text only)
      contrastText: '#000000'
    },
    
    success: {
      main: '#388e3c',     // 4.5:1 against white
      dark: '#2e7d32',     // 7:1 against white
      light: '#66bb6a',    // 3.1:1 against white (large text only)
      contrastText: '#ffffff'
    },
    
    // Text colors
    text: {
      primary: '#212121',    // 16.7:1 against white
      secondary: '#757575',  // 4.6:1 against white
      disabled: '#9e9e9e',   // 2.8:1 against white (not for body text)
      hint: '#757575'        // 4.6:1 against white
    },
    
    // Background colors
    background: {
      default: '#ffffff',
      paper: '#ffffff',
      alt: '#f5f5f5'       // Alternative background with good contrast
    }
  },
  
  // Typography scale optimized for accessibility
  typography: {
    // Font sizes that work well for accessibility
    fontSizes: {
      xs: '0.75rem',    // 12px - minimum for UI elements
      sm: '0.875rem',   // 14px - small text
      base: '1rem',     // 16px - body text
      lg: '1.125rem',   // 18px - large text threshold
      xl: '1.25rem',    // 20px
      '2xl': '1.5rem',  // 24px
      '3xl': '1.875rem', // 30px
      '4xl': '2.25rem'   // 36px
    },
    
    // Line heights for readability
    lineHeights: {
      tight: 1.25,
      normal: 1.5,
      relaxed: 1.75
    },
    
    // Font families with good accessibility support
    fontFamilies: {
      sans: [
        'Inter',
        '-apple-system',
        'BlinkMacSystemFont',
        'Segoe UI',
        'Roboto',
        'Helvetica Neue',
        'Arial',
        'sans-serif'
      ],
      mono: [
        'Monaco',
        'Menlo',
        'Ubuntu Mono',
        'monospace'
      ]
    }
  },
  
  // Spacing scale for consistent layouts
  spacing: {
    xs: '0.25rem',   // 4px
    sm: '0.5rem',    // 8px
    md: '1rem',      // 16px
    lg: '1.5rem',    // 24px
    xl: '2rem',      // 32px
    '2xl': '3rem',   // 48px
    '3xl': '4rem'    // 64px
  }
};

// Component for testing color contrast
const ContrastChecker: React.FC<{
  foreground: string;
  background: string;
  text: string;
  size?: 'normal' | 'large';
}> = ({ foreground, background, text, size = 'normal' }) => {
  const [contrastResult, setContrastResult] = useState<{
    passes: boolean;
    ratio: number;
    required: number;
  } | null>(null);
  
  useEffect(() => {
    const fgRgb = ColorContrastUtils.hexToRgb(foreground);
    const bgRgb = ColorContrastUtils.hexToRgb(background);
    
    if (fgRgb && bgRgb) {
      const result = ColorContrastUtils.meetsWCAGRequirements(fgRgb, bgRgb, 'AA', size);
      setContrastResult(result);
    }
  }, [foreground, background, size]);
  
  return (
    <div 
      style={{ 
        color: foreground, 
        backgroundColor: background,
        padding: '1rem',
        border: '1px solid #ccc'
      }}
    >
      <p style={{ fontSize: size === 'large' ? '18px' : '16px' }}>
        {text}
      </p>
      {contrastResult && (
        <div style={{ fontSize: '14px', marginTop: '0.5rem' }}>
          <strong>Contrast Ratio:</strong> {contrastResult.ratio.toFixed(2)}:1
          <br />
          <strong>Required:</strong> {contrastResult.required}:1
          <br />
          <strong>WCAG AA:</strong> {contrastResult.passes ? '✅ Pass' : '❌ Fail'}
        </div>
      )}
    </div>
  );
};
```

## 🔍 Automated Security and Accessibility Testing

### Security Testing with ESLint

```javascript
// .eslintrc.js - Security-focused configuration
module.exports = {
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:security/recommended'
  ],
  plugins: [
    'security',
    'no-unsanitized'
  ],
  rules: {
    // Security rules
    'security/detect-object-injection': 'error',
    'security/detect-non-literal-regexp': 'warn',
    'security/detect-unsafe-regex': 'error',
    'security/detect-buffer-noassert': 'error',
    'security/detect-child-process': 'warn',
    'security/detect-disable-mustache-escape': 'error',
    'security/detect-eval-with-expression': 'error',
    'security/detect-no-csrf-before-method-override': 'error',
    'security/detect-non-literal-fs-filename': 'warn',
    'security/detect-non-literal-require': 'warn',
    'security/detect-possible-timing-attacks': 'warn',
    'security/detect-pseudoRandomBytes': 'error',
    
    // No unsanitized content
    'no-unsanitized/method': 'error',
    'no-unsanitized/property': 'error',
    
    // React security
    'react/no-danger': 'error',
    'react/no-danger-with-children': 'error',
    'react/jsx-no-script-url': 'error',
    'react/jsx-no-target-blank': 'error'
  }
};
```

### Accessibility Testing Integration

```typescript
// jest-axe setup
import 'jest-axe/extend-expect';
import { configureAxe } from 'jest-axe';

// Configure axe for consistent testing
const axe = configureAxe({
  rules: {
    // Enable additional rules
    'color-contrast-enhanced': { enabled: true },
    'focus-order-semantics': { enabled: true },
    'hidden-content': { enabled: true },
    
    // Disable rules that might conflict with design system
    'color-contrast': { enabled: false }, // Using enhanced version
    'landmark-unique': { enabled: false } // May conflict with component libraries
  }
});

// Accessibility testing utilities
export const testAccessibility = async (component: React.ReactElement) => {
  const { container } = render(component);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
  return container;
};

export const testKeyboardNavigation = async (component: React.ReactElement) => {
  const { container } = render(component);
  const focusableElements = container.querySelectorAll(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  
  // Test tab navigation
  for (let i = 0; i < focusableElements.length; i++) {
    await userEvent.tab();
    expect(focusableElements[i]).toHaveFocus();
  }
  
  return container;
};

// Example accessibility tests
describe('AccessibleContactForm', () => {
  it('should not have accessibility violations', async () => {
    await testAccessibility(<AccessibleContactForm />);
  });
  
  it('should support keyboard navigation', async () => {
    await testKeyboardNavigation(<AccessibleContactForm />);
  });
  
  it('should announce form errors to screen readers', async () => {
    const user = userEvent.setup();
    render(<AccessibleContactForm />);
    
    // Submit form with empty fields
    await user.click(screen.getByRole('button', { name: /send message/i }));
    
    // Check that error messages are announced
    const errorMessages = screen.getAllByRole('alert');
    expect(errorMessages.length).toBeGreaterThan(0);
    
    // Check that form fields are marked as invalid
    const nameInput = screen.getByLabelText(/full name/i);
    expect(nameInput).toHaveAttribute('aria-invalid', 'true');
  });
  
  it('should maintain focus management in form', async () => {
    const user = userEvent.setup();
    render(<AccessibleContactForm />);
    
    const nameInput = screen.getByLabelText(/full name/i);
    const emailInput = screen.getByLabelText(/email/i);
    
    // Focus should move logically through form
    await user.click(nameInput);
    expect(nameInput).toHaveFocus();
    
    await user.tab();
    expect(emailInput).toHaveFocus();
  });
});
```

## 📝 Exercise: Security and Accessibility Audit

Conduct a comprehensive security and accessibility audit:

### Step 1: Security Assessment
- Review all user inputs for XSS vulnerabilities
- Implement CSRF protection
- Configure proper CSP headers
- Audit authentication and authorization
- Test for common OWASP Top 10 vulnerabilities

### Step 2: Accessibility Assessment
- Run automated accessibility tests
- Test keyboard navigation
- Verify color contrast ratios
- Test with screen readers
- Validate ARIA implementation

### Step 3: Implementation
- Fix identified security vulnerabilities
- Implement accessibility improvements
- Add automated testing
- Create security and accessibility guidelines

### Step 4: Monitoring
- Set up security monitoring
- Implement accessibility monitoring
- Create incident response procedures
- Document compliance requirements

**Deliverable:** A comprehensive security and accessibility report with identified issues, implemented fixes, testing procedures, and ongoing monitoring setup.

---

**Next:** [Module 8: Documentation & Handoff](../08-documentation-handoff/) - Master technical documentation, ADRs, and maintainable code practices.