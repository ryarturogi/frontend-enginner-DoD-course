# Module 5: Production-Grade Code Practices

## 🎯 Learning Objectives

By the end of this module, you will:
- Apply SOLID principles effectively in React and frontend development
- Implement DRY, KISS, and YAGNI principles in modern JavaScript/TypeScript
- Master error boundaries and graceful degradation patterns
- Optimize applications with code splitting and lazy loading
- Use TypeScript strict mode for maximum type safety
- Choose and implement appropriate state management patterns
- Design robust API communication patterns

## 🏗️ SOLID Principles in Frontend Development

### 1. Single Responsibility Principle (SRP)

**Definition:** A component or function should have only one reason to change.

#### ❌ Violating SRP
```tsx
// UserProfileCard doing too many things
const UserProfileCard = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [avatar, setAvatar] = useState(null);
  const [isEditing, setIsEditing] = useState(false);
  const [notifications, setNotifications] = useState([]);
  
  // Data fetching responsibility
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
      
    fetch(`/api/users/${userId}/avatar`)
      .then(res => res.blob())
      .then(setAvatar);
      
    fetch(`/api/users/${userId}/notifications`)
      .then(res => res.json())
      .then(setNotifications);
  }, [userId]);
  
  // Validation responsibility
  const validateUserData = (userData) => {
    const errors = {};
    if (!userData.name) errors.name = 'Name is required';
    if (!userData.email) errors.email = 'Email is required';
    return errors;
  };
  
  // Update responsibility
  const handleSave = async (userData) => {
    const errors = validateUserData(userData);
    if (Object.keys(errors).length > 0) return;
    
    await fetch(`/api/users/${userId}`, {
      method: 'PUT',
      body: JSON.stringify(userData)
    });
  };
  
  // Rendering responsibility (mixed with business logic)
  return (
    <div className="user-profile">
      {/* Complex rendering logic mixed with data handling */}
    </div>
  );
};
```

#### ✅ Following SRP
```tsx
// Separate concerns into focused components and hooks

// Data fetching responsibility
const useUser = (userId: string) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]);
  
  return { user, loading, error };
};

// Validation responsibility
const useUserValidation = () => {
  const validateUser = (userData: Partial<User>): ValidationErrors => {
    const errors: ValidationErrors = {};
    
    if (!userData.name?.trim()) {
      errors.name = 'Name is required';
    }
    
    if (!userData.email?.trim()) {
      errors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(userData.email)) {
      errors.email = 'Email format is invalid';
    }
    
    return errors;
  };
  
  return { validateUser };
};

// Update responsibility
const useUserUpdate = (userId: string) => {
  const [updating, setUpdating] = useState(false);
  
  const updateUser = async (userData: Partial<User>) => {
    setUpdating(true);
    try {
      const response = await fetch(`/api/users/${userId}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
      });
      
      if (!response.ok) throw new Error('Update failed');
      
      return await response.json();
    } finally {
      setUpdating(false);
    }
  };
  
  return { updateUser, updating };
};

// Pure presentation responsibility
const UserProfileCard: React.FC<{ userId: string }> = ({ userId }) => {
  const { user, loading, error } = useUser(userId);
  
  if (loading) return <UserProfileSkeleton />;
  if (error) return <UserProfileError error={error} />;
  if (!user) return <UserNotFound />;
  
  return (
    <div className="user-profile">
      <UserAvatar user={user} />
      <UserDetails user={user} />
      <UserActions userId={userId} />
    </div>
  );
};
```

### 2. Open/Closed Principle (OCP)

**Definition:** Software entities should be open for extension but closed for modification.

#### ✅ Extensible Button Component
```tsx
// Base button interface
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
  className?: string;
}

// Core button implementation (closed for modification)
const BaseButton: React.FC<ButtonProps> = ({ 
  children, 
  onClick, 
  disabled, 
  className 
}) => (
  <button 
    onClick={onClick}
    disabled={disabled}
    className={`btn ${className || ''}`}
  >
    {children}
  </button>
);

// Extensions (open for extension)
const PrimaryButton: React.FC<ButtonProps> = (props) => (
  <BaseButton {...props} className={`btn-primary ${props.className || ''}`} />
);

const SecondaryButton: React.FC<ButtonProps> = (props) => (
  <BaseButton {...props} className={`btn-secondary ${props.className || ''}`} />
);

// Loading button extension
interface LoadingButtonProps extends ButtonProps {
  loading?: boolean;
}

const LoadingButton: React.FC<LoadingButtonProps> = ({ 
  loading, 
  children, 
  disabled,
  ...props 
}) => (
  <PrimaryButton 
    {...props} 
    disabled={disabled || loading}
  >
    {loading ? <Spinner size="sm" /> : children}
  </PrimaryButton>
);

// Icon button extension
interface IconButtonProps extends ButtonProps {
  icon: React.ReactNode;
  iconPosition?: 'left' | 'right';
}

const IconButton: React.FC<IconButtonProps> = ({ 
  icon, 
  iconPosition = 'left', 
  children, 
  ...props 
}) => (
  <BaseButton {...props}>
    {iconPosition === 'left' && icon}
    {children}
    {iconPosition === 'right' && icon}
  </BaseButton>
);
```

#### ✅ Extensible Form Validation
```tsx
// Core validation interface (closed for modification)
interface Validator<T> {
  validate(value: T): ValidationResult;
}

interface ValidationResult {
  isValid: boolean;
  message?: string;
}

// Base validators (closed for modification)
class RequiredValidator implements Validator<string> {
  validate(value: string): ValidationResult {
    const isValid = value != null && value.trim().length > 0;
    return {
      isValid,
      message: isValid ? undefined : 'This field is required'
    };
  }
}

class EmailValidator implements Validator<string> {
  validate(value: string): ValidationResult {
    const emailRegex = /\S+@\S+\.\S+/;
    const isValid = emailRegex.test(value);
    return {
      isValid,
      message: isValid ? undefined : 'Please enter a valid email address'
    };
  }
}

// Extensions (open for extension)
class MinLengthValidator implements Validator<string> {
  constructor(private minLength: number) {}
  
  validate(value: string): ValidationResult {
    const isValid = value.length >= this.minLength;
    return {
      isValid,
      message: isValid ? undefined : `Minimum ${this.minLength} characters required`
    };
  }
}

class PasswordStrengthValidator implements Validator<string> {
  validate(value: string): ValidationResult {
    const hasUpper = /[A-Z]/.test(value);
    const hasLower = /[a-z]/.test(value);
    const hasNumber = /\d/.test(value);
    const hasSpecial = /[!@#$%^&*]/.test(value);
    
    const isValid = hasUpper && hasLower && hasNumber && hasSpecial;
    return {
      isValid,
      message: isValid ? undefined : 'Password must contain uppercase, lowercase, number, and special character'
    };
  }
}

// Composite validator (open for extension)
class CompositeValidator<T> implements Validator<T> {
  constructor(private validators: Validator<T>[]) {}
  
  validate(value: T): ValidationResult {
    for (const validator of this.validators) {
      const result = validator.validate(value);
      if (!result.isValid) {
        return result;
      }
    }
    return { isValid: true };
  }
}

// Usage - easily extensible without modifying existing code
const emailValidators = new CompositeValidator([
  new RequiredValidator(),
  new EmailValidator()
]);

const passwordValidators = new CompositeValidator([
  new RequiredValidator(),
  new MinLengthValidator(8),
  new PasswordStrengthValidator()
]);
```

### 3. Liskov Substitution Principle (LSP)

**Definition:** Objects of a superclass should be replaceable with objects of a subclass without breaking functionality.

#### ✅ Proper LSP Implementation
```tsx
// Base interface that all storage implementations must follow
interface Storage {
  getItem(key: string): Promise<string | null>;
  setItem(key: string, value: string): Promise<void>;
  removeItem(key: string): Promise<void>;
  clear(): Promise<void>;
}

// LocalStorage implementation
class LocalStorageAdapter implements Storage {
  async getItem(key: string): Promise<string | null> {
    try {
      return localStorage.getItem(key);
    } catch {
      return null;
    }
  }
  
  async setItem(key: string, value: string): Promise<void> {
    try {
      localStorage.setItem(key, value);
    } catch (error) {
      throw new Error(`Failed to save to localStorage: ${error.message}`);
    }
  }
  
  async removeItem(key: string): Promise<void> {
    try {
      localStorage.removeItem(key);
    } catch (error) {
      throw new Error(`Failed to remove from localStorage: ${error.message}`);
    }
  }
  
  async clear(): Promise<void> {
    try {
      localStorage.clear();
    } catch (error) {
      throw new Error(`Failed to clear localStorage: ${error.message}`);
    }
  }
}

// IndexedDB implementation - can substitute LocalStorage
class IndexedDBAdapter implements Storage {
  private dbName = 'AppStorage';
  private storeName = 'keyValue';
  
  async getItem(key: string): Promise<string | null> {
    const db = await this.openDB();
    return new Promise((resolve, reject) => {
      const transaction = db.transaction([this.storeName], 'readonly');
      const store = transaction.objectStore(this.storeName);
      const request = store.get(key);
      
      request.onsuccess = () => resolve(request.result?.value || null);
      request.onerror = () => reject(request.error);
    });
  }
  
  async setItem(key: string, value: string): Promise<void> {
    const db = await this.openDB();
    return new Promise((resolve, reject) => {
      const transaction = db.transaction([this.storeName], 'readwrite');
      const store = transaction.objectStore(this.storeName);
      const request = store.put({ key, value });
      
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
  
  async removeItem(key: string): Promise<void> {
    const db = await this.openDB();
    return new Promise((resolve, reject) => {
      const transaction = db.transaction([this.storeName], 'readwrite');
      const store = transaction.objectStore(this.storeName);
      const request = store.delete(key);
      
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
  
  async clear(): Promise<void> {
    const db = await this.openDB();
    return new Promise((resolve, reject) => {
      const transaction = db.transaction([this.storeName], 'readwrite');
      const store = transaction.objectStore(this.storeName);
      const request = store.clear();
      
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
  
  private openDB(): Promise<IDBDatabase> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, 1);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve(request.result);
      
      request.onupgradeneeded = () => {
        const db = request.result;
        if (!db.objectStoreNames.contains(this.storeName)) {
          db.createObjectStore(this.storeName, { keyPath: 'key' });
        }
      };
    });
  }
}

// Client code works with any Storage implementation
class UserPreferences {
  constructor(private storage: Storage) {}
  
  async saveTheme(theme: string): Promise<void> {
    await this.storage.setItem('theme', theme);
  }
  
  async getTheme(): Promise<string> {
    return (await this.storage.getItem('theme')) || 'light';
  }
  
  async clearAll(): Promise<void> {
    await this.storage.clear();
  }
}

// Can substitute implementations without changing client code
const userPrefs1 = new UserPreferences(new LocalStorageAdapter());
const userPrefs2 = new UserPreferences(new IndexedDBAdapter());
```

### 4. Interface Segregation Principle (ISP)

**Definition:** No client should be forced to depend on methods it does not use.

#### ❌ Violating ISP
```tsx
// Fat interface - forces unnecessary dependencies
interface MediaPlayer {
  play(): void;
  pause(): void;
  stop(): void;
  record(): void;      // Not all players can record
  stream(): void;      // Not all players can stream
  download(): void;    // Not all players support downloads
  uploadToCloud(): void; // Not all players have cloud features
}

// VideoPlayer forced to implement features it doesn't support
class VideoPlayer implements MediaPlayer {
  play() { /* implementation */ }
  pause() { /* implementation */ }
  stop() { /* implementation */ }
  
  // Forced to implement methods it doesn't need
  record() { throw new Error('Recording not supported'); }
  stream() { throw new Error('Streaming not supported'); }
  download() { throw new Error('Download not supported'); }
  uploadToCloud() { throw new Error('Cloud upload not supported'); }
}
```

#### ✅ Following ISP
```tsx
// Segregated interfaces
interface BasicPlayer {
  play(): void;
  pause(): void;
  stop(): void;
}

interface Recordable {
  record(): void;
  stopRecording(): void;
}

interface Streamable {
  startStream(): void;
  stopStream(): void;
}

interface Downloadable {
  download(url: string): Promise<void>;
}

interface CloudIntegration {
  uploadToCloud(): Promise<void>;
  downloadFromCloud(id: string): Promise<void>;
}

// Implementations only implement what they need
class SimpleVideoPlayer implements BasicPlayer {
  play() { /* implementation */ }
  pause() { /* implementation */ }
  stop() { /* implementation */ }
}

class AdvancedVideoPlayer implements BasicPlayer, Recordable, Downloadable {
  // BasicPlayer methods
  play() { /* implementation */ }
  pause() { /* implementation */ }
  stop() { /* implementation */ }
  
  // Recordable methods
  record() { /* implementation */ }
  stopRecording() { /* implementation */ }
  
  // Downloadable methods
  download(url: string) { /* implementation */ }
}

class StreamingPlayer implements BasicPlayer, Streamable, CloudIntegration {
  // BasicPlayer methods
  play() { /* implementation */ }
  pause() { /* implementation */ }
  stop() { /* implementation */ }
  
  // Streamable methods
  startStream() { /* implementation */ }
  stopStream() { /* implementation */ }
  
  // CloudIntegration methods
  uploadToCloud() { /* implementation */ }
  downloadFromCloud(id: string) { /* implementation */ }
}

// React component example
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
}

interface LoadingCapable {
  loading?: boolean;
}

interface IconCapable {
  icon?: React.ReactNode;
  iconPosition?: 'left' | 'right';
}

// Components implement only needed interfaces
const Button: React.FC<ButtonProps> = ({ children, onClick }) => (
  <button onClick={onClick}>{children}</button>
);

const LoadingButton: React.FC<ButtonProps & LoadingCapable> = ({ 
  children, 
  onClick, 
  loading 
}) => (
  <button onClick={onClick} disabled={loading}>
    {loading ? 'Loading...' : children}
  </button>
);

const IconButton: React.FC<ButtonProps & IconCapable> = ({ 
  children, 
  onClick, 
  icon, 
  iconPosition = 'left' 
}) => (
  <button onClick={onClick}>
    {iconPosition === 'left' && icon}
    {children}
    {iconPosition === 'right' && icon}
  </button>
);
```

### 5. Dependency Inversion Principle (DIP)

**Definition:** High-level modules should not depend on low-level modules. Both should depend on abstractions.

#### ✅ Following DIP
```tsx
// Abstractions (interfaces)
interface Logger {
  log(message: string, level: 'info' | 'warn' | 'error'): void;
}

interface ApiClient {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: any): Promise<T>;
}

interface NotificationService {
  send(message: string, type: 'success' | 'error' | 'info'): void;
}

// Low-level modules (implementations)
class ConsoleLogger implements Logger {
  log(message: string, level: 'info' | 'warn' | 'error'): void {
    console[level](`[${level.toUpperCase()}] ${message}`);
  }
}

class RemoteLogger implements Logger {
  async log(message: string, level: 'info' | 'warn' | 'error'): Promise<void> {
    await fetch('/api/logs', {
      method: 'POST',
      body: JSON.stringify({ message, level, timestamp: new Date() })
    });
  }
}

class FetchApiClient implements ApiClient {
  async get<T>(url: string): Promise<T> {
    const response = await fetch(url);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  }
  
  async post<T>(url: string, data: any): Promise<T> {
    const response = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  }
}

class ToastNotificationService implements NotificationService {
  send(message: string, type: 'success' | 'error' | 'info'): void {
    // Implementation using toast library
    toast(message, { type });
  }
}

// High-level module depends on abstractions
class UserService {
  constructor(
    private apiClient: ApiClient,
    private logger: Logger,
    private notifications: NotificationService
  ) {}
  
  async createUser(userData: CreateUserRequest): Promise<User> {
    try {
      this.logger.log('Creating new user', 'info');
      
      const user = await this.apiClient.post<User>('/api/users', userData);
      
      this.logger.log(`User created successfully: ${user.id}`, 'info');
      this.notifications.send('User created successfully!', 'success');
      
      return user;
    } catch (error) {
      this.logger.log(`Failed to create user: ${error.message}`, 'error');
      this.notifications.send('Failed to create user', 'error');
      throw error;
    }
  }
  
  async getUser(id: string): Promise<User> {
    try {
      return await this.apiClient.get<User>(`/api/users/${id}`);
    } catch (error) {
      this.logger.log(`Failed to fetch user ${id}: ${error.message}`, 'error');
      throw error;
    }
  }
}

// React hook that provides the service with injected dependencies
const useUserService = (): UserService => {
  return useMemo(() => {
    // Dependencies can be easily swapped based on environment
    const logger = process.env.NODE_ENV === 'production' 
      ? new RemoteLogger() 
      : new ConsoleLogger();
      
    const apiClient = new FetchApiClient();
    const notifications = new ToastNotificationService();
    
    return new UserService(apiClient, logger, notifications);
  }, []);
};

// React component uses the service
const CreateUserForm: React.FC = () => {
  const userService = useUserService();
  const [formData, setFormData] = useState<CreateUserRequest>({
    name: '',
    email: ''
  });
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      await userService.createUser(formData);
      setFormData({ name: '', email: '' });
    } catch (error) {
      // Error already handled by service
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form implementation */}
    </form>
  );
};
```

## 🎯 DRY, KISS, and YAGNI Principles

### DRY (Don't Repeat Yourself)

#### ✅ Eliminating Repetition with Custom Hooks
```tsx
// ❌ Repeated logic
const UserProfile = () => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetch('/api/user')
      .then(res => res.json())
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user?.name}</div>;
};

const UserSettings = () => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetch('/api/user')
      .then(res => res.json())
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user?.email}</div>;
};

// ✅ DRY with custom hook
const useUser = () => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      setError(null);
      try {
        const response = await fetch('/api/user');
        if (!response.ok) throw new Error('Failed to fetch user');
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err as Error);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, []);
  
  return { user, loading, error };
};

// Reusable loading and error handling
const withAsyncState = <T,>(
  Component: React.ComponentType<{ data: T }>
) => {
  return ({ data, loading, error }: { data: T; loading: boolean; error: Error | null }) => {
    if (loading) return <LoadingSpinner />;
    if (error) return <ErrorMessage error={error} />;
    if (!data) return <NoDataMessage />;
    return <Component data={data} />;
  };
};

// Clean components
const UserProfile = () => {
  const { user, loading, error } = useUser();
  const UserComponent = withAsyncState(({ data }: { data: User }) => (
    <div>{data.name}</div>
  ));
  
  return <UserComponent data={user} loading={loading} error={error} />;
};
```

### KISS (Keep It Simple, Stupid)

#### ✅ Simple, Clear Code
```tsx
// ❌ Over-engineered solution
class ComplexStateManager {
  private state: Map<string, any> = new Map();
  private listeners: Map<string, Set<Function>> = new Map();
  private middleware: Function[] = [];
  
  register(key: string, initialValue: any, validator?: Function) {
    // Complex registration logic...
  }
  
  update(key: string, value: any) {
    // Complex update with middleware pipeline...
  }
  
  subscribe(key: string, listener: Function) {
    // Complex subscription logic...
  }
}

// ✅ Simple solution using React's built-in capabilities
const useSimpleState = <T>(initialValue: T) => {
  const [value, setValue] = useState<T>(initialValue);
  
  const updateValue = useCallback((newValue: T | ((prev: T) => T)) => {
    setValue(newValue);
  }, []);
  
  return [value, updateValue] as const;
};

// ✅ Simple form handling
const ContactForm: React.FC = () => {
  const [formData, setFormData] = useSimpleState({
    name: '',
    email: '',
    message: ''
  });
  
  const handleChange = (field: keyof typeof formData) => (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    setFormData(prev => ({ ...prev, [field]: e.target.value }));
  };
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    await fetch('/api/contact', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData)
    });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        value={formData.name}
        onChange={handleChange('name')}
        placeholder="Name"
      />
      <input 
        value={formData.email}
        onChange={handleChange('email')}
        placeholder="Email"
        type="email"
      />
      <textarea 
        value={formData.message}
        onChange={handleChange('message')}
        placeholder="Message"
      />
      <button type="submit">Send</button>
    </form>
  );
};
```

### YAGNI (You Aren't Gonna Need It)

#### ✅ Build What You Need Now
```tsx
// ❌ Over-engineering for future requirements
interface UltraConfigurableButton {
  text?: string;
  icon?: string;
  iconPosition?: 'left' | 'right' | 'top' | 'bottom';
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl' | 'custom';
  variant?: 'primary' | 'secondary' | 'tertiary' | 'ghost' | 'link' | 'danger' | 'warning' | 'success';
  shape?: 'rectangle' | 'rounded' | 'circle' | 'pill';
  animation?: 'none' | 'pulse' | 'bounce' | 'shake' | 'glow' | 'custom';
  tooltip?: string;
  badge?: string | number;
  gradient?: boolean;
  shadow?: 'none' | 'sm' | 'md' | 'lg' | 'xl';
  responsive?: {
    mobile?: Partial<UltraConfigurableButton>;
    tablet?: Partial<UltraConfigurableButton>;
    desktop?: Partial<UltraConfigurableButton>;
  };
  // ... 20 more properties you might never use
}

// ✅ Simple solution for current needs
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary';
  type?: 'button' | 'submit';
}

const Button: React.FC<ButtonProps> = ({ 
  children, 
  onClick, 
  disabled, 
  variant = 'primary',
  type = 'button'
}) => (
  <button 
    type={type}
    onClick={onClick}
    disabled={disabled}
    className={`btn btn-${variant}`}
  >
    {children}
  </button>
);

// Add features only when needed
interface LoadingButtonProps extends ButtonProps {
  loading?: boolean;
}

const LoadingButton: React.FC<LoadingButtonProps> = ({ 
  loading, 
  children, 
  disabled,
  ...props 
}) => (
  <Button {...props} disabled={disabled || loading}>
    {loading ? 'Loading...' : children}
  </Button>
);
```

## ⚠️ Error Boundaries and Graceful Degradation

### Comprehensive Error Boundary Implementation

```tsx
interface ErrorInfo {
  componentStack: string;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
  errorInfo?: ErrorInfo;
  errorId?: string;
}

interface ErrorBoundaryProps {
  children: React.ReactNode;
  fallback?: React.ComponentType<{ error: Error; retry: () => void }>;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
  isolate?: boolean; // Prevent error propagation to parent
}

class ErrorBoundary extends React.Component<ErrorBoundaryProps, ErrorBoundaryState> {
  private retryTimeoutId: number | null = null;
  
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return {
      hasError: true,
      error,
      errorId: `error-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`
    };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    this.setState({ errorInfo });
    
    // Log error to monitoring service
    this.logError(error, errorInfo);
    
    // Call optional error handler
    this.props.onError?.(error, errorInfo);
  }
  
  private logError = (error: Error, errorInfo: ErrorInfo) => {
    const errorData = {
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      url: window.location.href,
      userId: this.getCurrentUserId(),
      errorId: this.state.errorId
    };
    
    // Send to monitoring service (Sentry, LogRocket, etc.)
    if (process.env.NODE_ENV === 'production') {
      fetch('/api/errors', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(errorData)
      }).catch(() => {
        // Fallback to console if logging service fails
        console.error('Failed to log error to service:', errorData);
      });
    } else {
      console.error('Error caught by boundary:', errorData);
    }
  };
  
  private getCurrentUserId = (): string | null => {
    // Get user ID from context, localStorage, etc.
    try {
      return localStorage.getItem('userId');
    } catch {
      return null;
    }
  };
  
  private handleRetry = () => {
    // Clear error state to retry rendering
    this.setState({ hasError: false, error: undefined, errorInfo: undefined });
  };
  
  private handleReload = () => {
    window.location.reload();
  };
  
  render() {
    if (this.state.hasError) {
      const FallbackComponent = this.props.fallback || DefaultErrorFallback;
      
      return (
        <FallbackComponent 
          error={this.state.error!}
          retry={this.handleRetry}
          reload={this.handleReload}
          errorId={this.state.errorId}
        />
      );
    }
    
    return this.props.children;
  }
}

// Default error fallback component
const DefaultErrorFallback: React.FC<{
  error: Error;
  retry: () => void;
  reload: () => void;
  errorId?: string;
}> = ({ error, retry, reload, errorId }) => (
  <div className="error-boundary-fallback">
    <h2>Something went wrong</h2>
    <details style={{ whiteSpace: 'pre-wrap' }}>
      <summary>Error details</summary>
      {error.message}
      {process.env.NODE_ENV === 'development' && (
        <pre>{error.stack}</pre>
      )}
    </details>
    {errorId && (
      <p>Error ID: <code>{errorId}</code></p>
    )}
    <div className="error-actions">
      <button onClick={retry}>Try Again</button>
      <button onClick={reload}>Reload Page</button>
    </div>
  </div>
);

// Specialized error boundaries for different contexts
const AsyncErrorBoundary: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <ErrorBoundary
    fallback={({ error, retry }) => (
      <div className="async-error">
        <p>Failed to load data</p>
        <button onClick={retry}>Retry</button>
      </div>
    )}
    onError={(error) => {
      // Specific handling for async errors
      console.warn('Async operation failed:', error.message);
    }}
  >
    {children}
  </ErrorBoundary>
);

const RouteErrorBoundary: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <ErrorBoundary
    fallback={({ error, reload }) => (
      <div className="route-error">
        <h1>Page Error</h1>
        <p>This page encountered an error and cannot be displayed.</p>
        <button onClick={reload}>Reload Page</button>
        <Link to="/">Go Home</Link>
      </div>
    )}
  >
    {children}
  </ErrorBoundary>
);
```

### Graceful Degradation Patterns

```tsx
// Progressive enhancement with fallbacks
const AdvancedChart: React.FC<{ data: ChartData }> = ({ data }) => {
  const [hasWebGL, setHasWebGL] = useState<boolean | null>(null);
  const [chartLibLoaded, setChartLibLoaded] = useState(false);
  
  useEffect(() => {
    // Check WebGL support
    const canvas = document.createElement('canvas');
    const gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');
    setHasWebGL(!!gl);
    
    // Dynamically load chart library
    import('chart.js').then(() => {
      setChartLibLoaded(true);
    }).catch(() => {
      console.warn('Chart library failed to load');
    });
  }, []);
  
  // Loading state
  if (hasWebGL === null || !chartLibLoaded) {
    return <ChartSkeleton />;
  }
  
  // Progressive fallbacks
  if (hasWebGL && chartLibLoaded) {
    return <WebGLChart data={data} />;
  }
  
  if (chartLibLoaded) {
    return <CanvasChart data={data} />;
  }
  
  // Final fallback - always works
  return <DataTable data={data} />;
};

// Network-aware components
const useNetworkStatus = () => {
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  const [connectionType, setConnectionType] = useState<string>('unknown');
  
  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    // Network Information API (when available)
    if ('connection' in navigator) {
      const connection = (navigator as any).connection;
      setConnectionType(connection.effectiveType || 'unknown');
      
      const handleConnectionChange = () => {
        setConnectionType(connection.effectiveType || 'unknown');
      };
      
      connection.addEventListener('change', handleConnectionChange);
      
      return () => {
        window.removeEventListener('online', handleOnline);
        window.removeEventListener('offline', handleOffline);
        connection.removeEventListener('change', handleConnectionChange);
      };
    }
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  
  return { isOnline, connectionType };
};

const AdaptiveImageGallery: React.FC<{ images: ImageData[] }> = ({ images }) => {
  const { isOnline, connectionType } = useNetworkStatus();
  
  // Adapt to network conditions
  const getImageQuality = () => {
    if (!isOnline) return 'thumbnail';
    if (connectionType === 'slow-2g' || connectionType === '2g') return 'low';
    if (connectionType === '3g') return 'medium';
    return 'high';
  };
  
  const quality = getImageQuality();
  
  if (!isOnline) {
    return (
      <div className="offline-gallery">
        <OfflineIcon />
        <p>Gallery unavailable offline</p>
        <p>Images will load when connection is restored</p>
      </div>
    );
  }
  
  return (
    <div className="image-gallery">
      {images.map(image => (
        <AdaptiveImage 
          key={image.id}
          src={image.urls[quality]}
          alt={image.alt}
          quality={quality}
        />
      ))}
    </div>
  );
};

// Feature detection with polyfills
const ModernFeatureComponent: React.FC = () => {
  const [supportsIntersectionObserver, setSupportsIntersectionObserver] = useState(false);
  const [supportsResizeObserver, setSupportsResizeObserver] = useState(false);
  
  useEffect(() => {
    // Feature detection
    setSupportsIntersectionObserver('IntersectionObserver' in window);
    setSupportsResizeObserver('ResizeObserver' in window);
    
    // Load polyfills if needed
    const loadPolyfills = async () => {
      if (!supportsIntersectionObserver) {
        await import('intersection-observer');
        setSupportsIntersectionObserver(true);
      }
      
      if (!supportsResizeObserver) {
        await import('@juggle/resize-observer');
        setSupportsResizeObserver(true);
      }
    };
    
    loadPolyfills();
  }, []);
  
  if (supportsIntersectionObserver && supportsResizeObserver) {
    return <ModernScrollComponent />;
  }
  
  if (supportsIntersectionObserver) {
    return <PartiallyModernComponent />;
  }
  
  return <FallbackScrollComponent />;
};
```

## 📝 Exercise: Refactor with SOLID Principles

Take a complex component from your codebase and refactor it following SOLID principles:

### Step 1: Identify Violations
- Does one component handle multiple responsibilities?
- Are there hardcoded dependencies?
- Can you easily extend functionality without modifying existing code?

### Step 2: Apply SOLID Principles
- **SRP:** Break down into single-purpose components/hooks
- **OCP:** Make components extensible without modification
- **LSP:** Ensure interface substitutability
- **ISP:** Create focused interfaces
- **DIP:** Depend on abstractions, not concretions

### Step 3: Add Error Handling
- Implement appropriate error boundaries
- Add graceful degradation for failure scenarios
- Include logging and monitoring

### Step 4: Optimize for Production
- Add performance optimizations
- Implement proper loading states
- Consider network conditions

**Deliverable:** A refactored component demonstrating all SOLID principles with comprehensive error handling and graceful degradation.

---

**Next:** [Module 6: Performance & Monitoring](../06-performance-monitoring/) - Master Core Web Vitals, performance budgets, caching strategies, and observability.