# Module 5: Production-Grade Code Practices

## 🎯 Learning Objectives

By the end of this module, you will:
- **Master Modern SOLID Principles**: Apply SOLID principles with advanced TypeScript patterns and React 19 features
- **Advanced Architecture Patterns**: Implement Clean Architecture, Hexagonal Architecture, and modern design patterns
- **Enterprise Error Handling**: Build resilient error boundaries with automatic recovery and sophisticated logging
- **Performance-First Coding**: Optimize with React 19 concurrent features, Actions, and advanced code splitting
- **Type-Safe Development**: Master TypeScript strict mode, advanced utility types, and enterprise-grade type patterns
- **Modern State Management**: Implement cutting-edge state patterns with Valtio, Jotai, and advanced Zustand patterns
- **Security-First Development**: Integrate CSP, input sanitization, XSS prevention, and modern security patterns
- **Memory Management Excellence**: Prevent memory leaks with advanced cleanup patterns and performance monitoring
- **API Communication Mastery**: Design robust API patterns with automatic retry, caching, and error recovery
- **Browser API Integration**: Leverage modern Web APIs like View Transitions, Web Locks, and File System Access
- **Component Architecture Excellence**: Master compound components, render props evolution, and advanced React patterns
- **Code Quality Automation**: Implement automated code quality with advanced linting, formatting, and analysis tools

## 🏗️ Advanced SOLID Principles in Modern Frontend

### 1. Single Responsibility Principle (SRP)

**Modern Definition:** A component, hook, or function should have only one reason to change, with clear boundaries of responsibility in the React 19 ecosystem.

#### ❌ Violating SRP (Anti-patterns)
```tsx
// UserProfileCard violating SRP with mixed concerns
const UserProfileCard = ({ userId }: { userId: string }) => {
  const [user, setUser] = useState<User | null>(null);
  const [avatar, setAvatar] = useState<string | null>(null);
  const [isEditing, setIsEditing] = useState(false);
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [analytics, setAnalytics] = useState<AnalyticsData | null>(null);
  
  // Multiple data fetching concerns
  useEffect(() => {
    Promise.all([
      fetch(`/api/users/${userId}`).then(res => res.json()),
      fetch(`/api/users/${userId}/avatar`).then(res => res.text()),
      fetch(`/api/users/${userId}/notifications`).then(res => res.json()),
      fetch(`/api/analytics/user/${userId}`).then(res => res.json()),
    ]).then(([userData, avatarUrl, userNotifications, analyticsData]) => {
      setUser(userData);
      setAvatar(avatarUrl);
      setNotifications(userNotifications);
      setAnalytics(analyticsData);
    });
  }, [userId]);
  
  // Validation logic mixed in
  const validateUserData = (userData: Partial<User>): ValidationErrors => {
    const errors: ValidationErrors = {};
    if (!userData.name?.trim()) errors.name = 'Name is required';
    if (!userData.email?.includes('@')) errors.email = 'Valid email required';
    if (!userData.phoneNumber?.match(/^\+?[\d\s\-\(\)]{10,}$/)) {
      errors.phoneNumber = 'Valid phone number required';
    }
    return errors;
  };
  
  // Update logic mixed in
  const handleSave = async (userData: Partial<User>) => {
    const errors = validateUserData(userData);
    if (Object.keys(errors).length > 0) {
      setNotifications(prev => [...prev, { 
        type: 'error', 
        message: 'Please fix validation errors' 
      }]);
      return;
    }
    
    try {
      await fetch(`/api/users/${userId}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
      });
      
      // Analytics tracking mixed in
      analytics?.trackEvent('user_profile_updated', { userId, fields: Object.keys(userData) });
    } catch (error) {
      setNotifications(prev => [...prev, { 
        type: 'error', 
        message: 'Failed to update profile' 
      }]);
    }
  };
  
  // Complex rendering with business logic
  return (
    <div className="user-profile">
      {/* Complex rendering logic mixed with data handling */}
    </div>
  );
};
```

#### ✅ Following SRP (Best Practices)
```tsx
// 1. Data fetching responsibility - Custom hook with error handling
const useUserData = (userId: string) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  const fetchUser = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      const userData = await response.json();
      setUser(userData);
    } catch (err) {
      setError(err instanceof Error ? err : new Error(String(err)));
    } finally {
      setLoading(false);
    }
  }, [userId]);
  
  useEffect(() => {
    fetchUser();
  }, [fetchUser]);
  
  return { user, loading, error, refetch: fetchUser };
};

// 2. Avatar handling responsibility
const useUserAvatar = (userId: string) => {
  const [avatar, setAvatar] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    const loadAvatar = async () => {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}/avatar`);
        const avatarUrl = await response.text();
        setAvatar(avatarUrl);
      } catch (error) {
        console.warn('Failed to load avatar:', error);
      } finally {
        setLoading(false);
      }
    };
    
    loadAvatar();
  }, [userId]);
  
  return { avatar, loading };
};

// 3. Validation responsibility - Pure function
const validateUserProfile = (userData: Partial<User>): ValidationResult => {
  const errors: ValidationErrors = {};
  
  if (!userData.name?.trim()) {
    errors.name = 'Name is required and cannot be empty';
  }
  
  if (!userData.email?.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
    errors.email = 'Please provide a valid email address';
  }
  
  if (userData.phoneNumber && !userData.phoneNumber.match(/^\+?[\d\s\-\(\)]{10,}$/)) {
    errors.phoneNumber = 'Please provide a valid phone number';
  }
  
  if (userData.dateOfBirth) {
    const birthDate = new Date(userData.dateOfBirth);
    const minAge = new Date();
    minAge.setFullYear(minAge.getFullYear() - 13);
    
    if (birthDate > minAge) {
      errors.dateOfBirth = 'User must be at least 13 years old';
    }
  }
  
  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
};

// 4. User update responsibility - Service layer
class UserUpdateService {
  constructor(
    private apiClient: ApiClient,
    private analytics: AnalyticsService,
    private notifications: NotificationService
  ) {}
  
  async updateUser(userId: string, userData: Partial<User>): Promise<Result<User, UpdateError>> {
    try {
      const updatedUser = await this.apiClient.put<User>(`/users/${userId}`, userData);
      
      // Track successful update
      this.analytics.trackEvent('user_profile_updated', {
        userId,
        fields: Object.keys(userData),
        timestamp: new Date().toISOString()
      });
      
      // Show success notification
      this.notifications.showSuccess('Profile updated successfully');
      
      return { success: true, data: updatedUser };
    } catch (error) {
      const updateError = error instanceof Error ? error : new Error(String(error));
      
      // Track failed update
      this.analytics.trackEvent('user_profile_update_failed', {
        userId,
        error: updateError.message,
        timestamp: new Date().toISOString()
      });
      
      // Show error notification
      this.notifications.showError('Failed to update profile. Please try again.');
      
      return { success: false, error: updateError };
    }
  }
}

// 5. Presentation responsibility - Clean component
const UserProfileCard: React.FC<{ userId: string }> = ({ userId }) => {
  const { user, loading: userLoading, error: userError, refetch } = useUserData(userId);
  const { avatar, loading: avatarLoading } = useUserAvatar(userId);
  const [isEditing, setIsEditing] = useState(false);
  
  const userUpdateService = useService('userUpdateService');
  
  const handleSave = async (userData: Partial<User>) => {
    const validation = validateUserProfile(userData);
    
    if (!validation.isValid) {
      // Handle validation errors
      return;
    }
    
    const result = await userUpdateService.updateUser(userId, userData);
    
    if (result.success) {
      setIsEditing(false);
      refetch(); // Refresh user data
    }
  };
  
  if (userLoading) return <UserProfileSkeleton />;
  if (userError) return <ErrorBoundary error={userError} onRetry={refetch} />;
  if (!user) return <EmptyUserState />;
  
  return (
    <div className="user-profile-card">
      <UserAvatar 
        avatar={avatar} 
        loading={avatarLoading} 
        alt={user.name}
      />
      
      {isEditing ? (
        <UserProfileForm
          user={user}
          onSave={handleSave}
          onCancel={() => setIsEditing(false)}
          validator={validateUserProfile}
        />
      ) : (
        <UserProfileDisplay
          user={user}
          onEdit={() => setIsEditing(true)}
        />
      )}
    </div>
  );
};

// Supporting components with single responsibilities
const UserAvatar: React.FC<{
  avatar: string | null;
  loading: boolean;
  alt: string;
}> = ({ avatar, loading, alt }) => {
  if (loading) return <AvatarSkeleton />;
  
  return (
    <img 
      src={avatar || '/default-avatar.png'} 
      alt={alt}
      className="user-avatar"
      loading="lazy"
    />
  );
};

const UserProfileDisplay: React.FC<{
  user: User;
  onEdit: () => void;
}> = ({ user, onEdit }) => (
  <div className="user-profile-display">
    <h2>{user.name}</h2>
    <p>{user.email}</p>
    <button onClick={onEdit} className="edit-button">
      Edit Profile
    </button>
  </div>
);
```

### 2. Open/Closed Principle (OCP) - Modern Implementation

**Definition:** Software entities should be open for extension but closed for modification, using composition, dependency injection, and TypeScript's advanced type system.

#### ✅ Modern OCP Implementation with TypeScript
```tsx
// Base notification system - closed for modification
interface Notification {
  id: string;
  type: string;
  message: string;
  timestamp: Date;
  metadata?: Record<string, unknown>;
}

interface NotificationHandler {
  canHandle(notification: Notification): boolean;
  handle(notification: Notification): Promise<void>;
  priority: number;
}

// Extensible notification system
class NotificationService {
  private handlers: NotificationHandler[] = [];
  
  registerHandler(handler: NotificationHandler): void {
    this.handlers.push(handler);
    this.handlers.sort((a, b) => b.priority - a.priority);
  }
  
  async processNotification(notification: Notification): Promise<void> {
    const handler = this.handlers.find(h => h.canHandle(notification));
    
    if (!handler) {
      console.warn(`No handler found for notification type: ${notification.type}`);
      return;
    }
    
    await handler.handle(notification);
  }
}

// Extensions - open for extension without modifying base
class EmailNotificationHandler implements NotificationHandler {
  priority = 10;
  
  canHandle(notification: Notification): boolean {
    return notification.type === 'email';
  }
  
  async handle(notification: Notification): Promise<void> {
    // Email-specific handling
    await this.sendEmail(notification);
  }
  
  private async sendEmail(notification: Notification): Promise<void> {
    // Implementation
  }
}

class PushNotificationHandler implements NotificationHandler {
  priority = 15;
  
  canHandle(notification: Notification): boolean {
    return notification.type === 'push';
  }
  
  async handle(notification: Notification): Promise<void> {
    // Push notification specific handling
    await this.sendPushNotification(notification);
  }
  
  private async sendPushNotification(notification: Notification): Promise<void> {
    // Implementation
  }
}

// New handler for Slack notifications - extends without modification
class SlackNotificationHandler implements NotificationHandler {
  priority = 8;
  
  canHandle(notification: Notification): boolean {
    return notification.type === 'slack';
  }
  
  async handle(notification: Notification): Promise<void> {
    await this.sendSlackMessage(notification);
  }
  
  private async sendSlackMessage(notification: Notification): Promise<void> {
    // Slack-specific implementation
  }
}

// Usage
const notificationService = new NotificationService();
notificationService.registerHandler(new EmailNotificationHandler());
notificationService.registerHandler(new PushNotificationHandler());
notificationService.registerHandler(new SlackNotificationHandler());
```

#### Modern Component Extension Pattern
```tsx
// Base component system - closed for modification
interface ComponentProps {
  children?: React.ReactNode;
  className?: string;
  'data-testid'?: string;
}

interface ButtonVariant {
  getStyles(): string;
  getAriaAttributes(): Record<string, string>;
}

// Base button - closed for modification
const Button: React.FC<ComponentProps & {
  variant?: ButtonVariant;
  onClick?: () => void;
  disabled?: boolean;
}> = ({ variant, onClick, disabled, children, className = '', ...props }) => {
  const styles = variant?.getStyles() || 'btn-default';
  const ariaAttributes = variant?.getAriaAttributes() || {};
  
  return (
    <button
      className={`btn ${styles} ${className}`}
      onClick={onClick}
      disabled={disabled}
      {...ariaAttributes}
      {...props}
    >
      {children}
    </button>
  );
};

// Extensions - open for extension
class PrimaryButtonVariant implements ButtonVariant {
  getStyles(): string {
    return 'btn-primary bg-blue-600 hover:bg-blue-700 text-white font-semibold';
  }
  
  getAriaAttributes(): Record<string, string> {
    return { 'aria-describedby': 'primary-button-help' };
  }
}

class DangerButtonVariant implements ButtonVariant {
  getStyles(): string {
    return 'btn-danger bg-red-600 hover:bg-red-700 text-white font-semibold';
  }
  
  getAriaAttributes(): Record<string, string> {
    return { 
      'aria-describedby': 'danger-button-warning',
      'aria-label': 'Destructive action'
    };
  }
}

// New variant without modifying existing code
class GlassmorphismButtonVariant implements ButtonVariant {
  getStyles(): string {
    return 'btn-glass backdrop-blur-md bg-white/20 border border-white/30 text-gray-800';
  }
  
  getAriaAttributes(): Record<string, string> {
    return { 'aria-describedby': 'glass-button-description' };
  }
}

// Usage
const App = () => (
  <div>
    <Button variant={new PrimaryButtonVariant()}>Primary Action</Button>
    <Button variant={new DangerButtonVariant()}>Delete</Button>
    <Button variant={new GlassmorphismButtonVariant()}>Modern Glass</Button>
  </div>
);
```

### 3. Liskov Substitution Principle (LSP) - Advanced TypeScript

**Definition:** Objects of a supertype should be replaceable with objects of a subtype without breaking the application, enforced through TypeScript's type system.

#### ✅ Advanced LSP with TypeScript Constraints
```tsx
// Base data fetcher contract
interface DataFetcher<T> {
  fetch(): Promise<Result<T, Error>>;
  validateResponse(response: unknown): response is T;
  getCacheKey(): string;
  getCacheDuration(): number;
}

// Base implementation
abstract class BaseDataFetcher<T> implements DataFetcher<T> {
  protected cacheService: CacheService;
  
  constructor(cacheService: CacheService) {
    this.cacheService = cacheService;
  }
  
  async fetch(): Promise<Result<T, Error>> {
    const cacheKey = this.getCacheKey();
    const cachedData = await this.cacheService.get<T>(cacheKey);
    
    if (cachedData) {
      return { success: true, data: cachedData };
    }
    
    try {
      const response = await this.fetchFromSource();
      
      if (!this.validateResponse(response)) {
        throw new Error('Invalid response format');
      }
      
      await this.cacheService.set(cacheKey, response, this.getCacheDuration());
      return { success: true, data: response };
    } catch (error) {
      return { 
        success: false, 
        error: error instanceof Error ? error : new Error(String(error))
      };
    }
  }
  
  abstract validateResponse(response: unknown): response is T;
  abstract getCacheKey(): string;
  abstract getCacheDuration(): number;
  protected abstract fetchFromSource(): Promise<T>;
}

// Substitutable implementations
class UserDataFetcher extends BaseDataFetcher<User> {
  constructor(
    cacheService: CacheService,
    private userId: string,
    private apiClient: ApiClient
  ) {
    super(cacheService);
  }
  
  validateResponse(response: unknown): response is User {
    return (
      typeof response === 'object' &&
      response !== null &&
      'id' in response &&
      'name' in response &&
      'email' in response
    );
  }
  
  getCacheKey(): string {
    return `user:${this.userId}`;
  }
  
  getCacheDuration(): number {
    return 300000; // 5 minutes
  }
  
  protected async fetchFromSource(): Promise<User> {
    return this.apiClient.get<User>(`/users/${this.userId}`);
  }
}

class ProductDataFetcher extends BaseDataFetcher<Product> {
  constructor(
    cacheService: CacheService,
    private productId: string,
    private apiClient: ApiClient
  ) {
    super(cacheService);
  }
  
  validateResponse(response: unknown): response is Product {
    return (
      typeof response === 'object' &&
      response !== null &&
      'id' in response &&
      'name' in response &&
      'price' in response
    );
  }
  
  getCacheKey(): string {
    return `product:${this.productId}`;
  }
  
  getCacheDuration(): number {
    return 600000; // 10 minutes
  }
  
  protected async fetchFromSource(): Promise<Product> {
    return this.apiClient.get<Product>(`/products/${this.productId}`);
  }
}

// Generic data hook that works with any DataFetcher (LSP in action)
const useDataFetcher = <T>(fetcher: DataFetcher<T>) => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    const result = await fetcher.fetch();
    
    if (result.success) {
      setData(result.data);
    } else {
      setError(result.error);
    }
    
    setLoading(false);
  }, [fetcher]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return { data, loading, error, refetch: fetchData };
};

// Usage - any DataFetcher can be substituted
const UserProfile: React.FC<{ userId: string }> = ({ userId }) => {
  const cacheService = useService('cacheService');
  const apiClient = useService('apiClient');
  
  const userFetcher = useMemo(
    () => new UserDataFetcher(cacheService, userId, apiClient),
    [cacheService, userId, apiClient]
  );
  
  const { data: user, loading, error } = useDataFetcher(userFetcher);
  
  if (loading) return <ProfileSkeleton />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <EmptyState />;
  
  return <UserCard user={user} />;
};
```

### 4. Interface Segregation Principle (ISP) - Modern TypeScript

**Definition:** Clients should not be forced to depend on interfaces they don't use, implemented through focused TypeScript interfaces and composition.

#### ❌ Violating ISP
```tsx
// Fat interface that violates ISP
interface UserService {
  // Authentication methods
  login(email: string, password: string): Promise<AuthResult>;
  logout(): Promise<void>;
  refreshToken(): Promise<string>;
  
  // Profile management
  getProfile(userId: string): Promise<User>;
  updateProfile(userId: string, data: Partial<User>): Promise<User>;
  uploadAvatar(userId: string, file: File): Promise<string>;
  
  // Preferences
  getPreferences(userId: string): Promise<UserPreferences>;
  updatePreferences(userId: string, prefs: Partial<UserPreferences>): Promise<UserPreferences>;
  
  // Admin operations
  deleteUser(userId: string): Promise<void>;
  suspendUser(userId: string, reason: string): Promise<void>;
  promoteToAdmin(userId: string): Promise<void>;
  
  // Analytics
  trackUserEvent(userId: string, event: string, data: any): Promise<void>;
  getUserAnalytics(userId: string): Promise<UserAnalytics>;
}
```

#### ✅ Following ISP (Best Practices)
```tsx
// Segregated interfaces for different concerns
interface AuthenticationService {
  login(credentials: LoginCredentials): Promise<AuthResult>;
  logout(): Promise<void>;
  refreshToken(): Promise<TokenRefreshResult>;
  validateSession(): Promise<SessionValidationResult>;
}

interface UserProfileService {
  getProfile(userId: string): Promise<User>;
  updateProfile(userId: string, updates: Partial<User>): Promise<User>;
  deleteProfile(userId: string): Promise<void>;
}

interface UserAvatarService {
  getAvatar(userId: string): Promise<string | null>;
  uploadAvatar(userId: string, file: File): Promise<AvatarUploadResult>;
  deleteAvatar(userId: string): Promise<void>;
}

interface UserPreferencesService {
  getPreferences(userId: string): Promise<UserPreferences>;
  updatePreferences(userId: string, preferences: Partial<UserPreferences>): Promise<UserPreferences>;
  resetPreferences(userId: string): Promise<UserPreferences>;
}

interface UserAdministrationService {
  suspendUser(userId: string, reason: string): Promise<AdminActionResult>;
  unsuspendUser(userId: string): Promise<AdminActionResult>;
  promoteUser(userId: string, role: UserRole): Promise<AdminActionResult>;
  demoteUser(userId: string): Promise<AdminActionResult>;
}

interface UserAnalyticsService {
  trackEvent(userId: string, event: AnalyticsEvent): Promise<void>;
  getAnalytics(userId: string, timeRange: TimeRange): Promise<UserAnalytics>;
  generateReport(userId: string, reportType: ReportType): Promise<AnalyticsReport>;
}

// Focused implementations
class AuthenticationServiceImpl implements AuthenticationService {
  constructor(
    private apiClient: ApiClient,
    private tokenStorage: TokenStorage,
    private cryptoService: CryptoService
  ) {}
  
  async login(credentials: LoginCredentials): Promise<AuthResult> {
    const encryptedCredentials = await this.cryptoService.encrypt(credentials);
    const result = await this.apiClient.post<AuthResponse>('/auth/login', encryptedCredentials);
    
    if (result.success) {
      await this.tokenStorage.store(result.data.token, result.data.refreshToken);
    }
    
    return result;
  }
  
  async logout(): Promise<void> {
    await this.apiClient.post('/auth/logout');
    await this.tokenStorage.clear();
  }
  
  async refreshToken(): Promise<TokenRefreshResult> {
    const refreshToken = await this.tokenStorage.getRefreshToken();
    if (!refreshToken) {
      throw new Error('No refresh token available');
    }
    
    const result = await this.apiClient.post<TokenResponse>('/auth/refresh', {
      refreshToken
    });
    
    if (result.success) {
      await this.tokenStorage.store(result.data.token, result.data.refreshToken);
    }
    
    return result;
  }
  
  async validateSession(): Promise<SessionValidationResult> {
    const token = await this.tokenStorage.getToken();
    if (!token) {
      return { valid: false, reason: 'No token found' };
    }
    
    try {
      const result = await this.apiClient.get<SessionInfo>('/auth/validate');
      return { valid: result.success, session: result.data };
    } catch (error) {
      return { valid: false, reason: 'Session validation failed' };
    }
  }
}

class UserProfileServiceImpl implements UserProfileService {
  constructor(
    private apiClient: ApiClient,
    private validator: ProfileValidator,
    private cacheService: CacheService
  ) {}
  
  async getProfile(userId: string): Promise<User> {
    const cacheKey = `profile:${userId}`;
    const cached = await this.cacheService.get<User>(cacheKey);
    
    if (cached) return cached;
    
    const result = await this.apiClient.get<User>(`/users/${userId}`);
    await this.cacheService.set(cacheKey, result, 300000); // 5 minutes
    
    return result;
  }
  
  async updateProfile(userId: string, updates: Partial<User>): Promise<User> {
    const validationResult = await this.validator.validate(updates);
    if (!validationResult.isValid) {
      throw new ValidationError('Invalid profile data', validationResult.errors);
    }
    
    const result = await this.apiClient.put<User>(`/users/${userId}`, updates);
    
    // Invalidate cache
    await this.cacheService.delete(`profile:${userId}`);
    
    return result;
  }
  
  async deleteProfile(userId: string): Promise<void> {
    await this.apiClient.delete(`/users/${userId}`);
    await this.cacheService.delete(`profile:${userId}`);
  }
}

// Composition for complex use cases
interface UserManagementFacade extends 
  UserProfileService, 
  UserPreferencesService {}

class UserManagementFacadeImpl implements UserManagementFacade {
  constructor(
    private profileService: UserProfileService,
    private preferencesService: UserPreferencesService
  ) {}
  
  // Delegate to appropriate services
  async getProfile(userId: string): Promise<User> {
    return this.profileService.getProfile(userId);
  }
  
  async updateProfile(userId: string, updates: Partial<User>): Promise<User> {
    return this.profileService.updateProfile(userId, updates);
  }
  
  async deleteProfile(userId: string): Promise<void> {
    return this.profileService.deleteProfile(userId);
  }
  
  async getPreferences(userId: string): Promise<UserPreferences> {
    return this.preferencesService.getPreferences(userId);
  }
  
  async updatePreferences(userId: string, preferences: Partial<UserPreferences>): Promise<UserPreferences> {
    return this.preferencesService.updatePreferences(userId, preferences);
  }
  
  async resetPreferences(userId: string): Promise<UserPreferences> {
    return this.preferencesService.resetPreferences(userId);
  }
  
  // Composite operations
  async setupNewUser(userId: string, profile: Partial<User>, preferences: Partial<UserPreferences>): Promise<{
    profile: User;
    preferences: UserPreferences;
  }> {
    const [updatedProfile, updatedPreferences] = await Promise.all([
      this.profileService.updateProfile(userId, profile),
      this.preferencesService.updatePreferences(userId, preferences)
    ]);
    
    return { profile: updatedProfile, preferences: updatedPreferences };
  }
}

// Hook usage with specific interfaces
const useAuthentication = (): {
  authService: AuthenticationService;
  login: (credentials: LoginCredentials) => Promise<AuthResult>;
  logout: () => Promise<void>;
  isAuthenticated: boolean;
} => {
  const authService = useService<AuthenticationService>('authenticationService');
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  
  const login = useCallback(async (credentials: LoginCredentials) => {
    const result = await authService.login(credentials);
    if (result.success) {
      setIsAuthenticated(true);
    }
    return result;
  }, [authService]);
  
  const logout = useCallback(async () => {
    await authService.logout();
    setIsAuthenticated(false);
  }, [authService]);
  
  useEffect(() => {
    const validateSession = async () => {
      const validation = await authService.validateSession();
      setIsAuthenticated(validation.valid);
    };
    
    validateSession();
  }, [authService]);
  
  return { authService, login, logout, isAuthenticated };
};

const useUserProfile = (userId: string): {
  profile: User | null;
  loading: boolean;
  error: Error | null;
  updateProfile: (updates: Partial<User>) => Promise<void>;
  refetch: () => void;
} => {
  const profileService = useService<UserProfileService>('userProfileService');
  const [profile, setProfile] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  const fetchProfile = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const userProfile = await profileService.getProfile(userId);
      setProfile(userProfile);
    } catch (err) {
      setError(err instanceof Error ? err : new Error(String(err)));
    } finally {
      setLoading(false);
    }
  }, [profileService, userId]);
  
  const updateProfile = useCallback(async (updates: Partial<User>) => {
    try {
      const updatedProfile = await profileService.updateProfile(userId, updates);
      setProfile(updatedProfile);
    } catch (err) {
      setError(err instanceof Error ? err : new Error(String(err)));
      throw err;
    }
  }, [profileService, userId]);
  
  useEffect(() => {
    fetchProfile();
  }, [fetchProfile]);
  
  return { profile, loading, error, updateProfile, refetch: fetchProfile };
};
```

### 5. Dependency Inversion Principle (DIP) - Advanced Implementation

**Definition:** High-level modules should not depend on low-level modules. Both should depend on abstractions, implemented through dependency injection and IoC containers.

#### ✅ Advanced DIP with Modern Dependency Injection
```tsx
// Abstractions (high-level contracts)
interface Logger {
  info(message: string, meta?: Record<string, unknown>): void;
  warn(message: string, meta?: Record<string, unknown>): void;
  error(message: string, error?: Error, meta?: Record<string, unknown>): void;
  debug(message: string, meta?: Record<string, unknown>): void;
}

interface ApiClient {
  get<T>(url: string, config?: RequestConfig): Promise<T>;
  post<T>(url: string, data?: unknown, config?: RequestConfig): Promise<T>;
  put<T>(url: string, data?: unknown, config?: RequestConfig): Promise<T>;
  delete<T>(url: string, config?: RequestConfig): Promise<T>;
}

interface CacheService {
  get<T>(key: string): Promise<T | null>;
  set<T>(key: string, value: T, ttl?: number): Promise<void>;
  delete(key: string): Promise<void>;
  clear(): Promise<void>;
}

interface EventBus {
  emit<T>(event: string, payload: T): void;
  on<T>(event: string, handler: (payload: T) => void): () => void;
  off(event: string, handler: Function): void;
}

// Container and dependency registration
type ServiceIdentifier<T = any> = string | symbol | (new (...args: any[]) => T);

interface ServiceContainer {
  register<T>(identifier: ServiceIdentifier<T>, factory: () => T): void;
  registerSingleton<T>(identifier: ServiceIdentifier<T>, factory: () => T): void;
  get<T>(identifier: ServiceIdentifier<T>): T;
  has(identifier: ServiceIdentifier): boolean;
}

class DIContainer implements ServiceContainer {
  private services = new Map<ServiceIdentifier, any>();
  private singletons = new Map<ServiceIdentifier, any>();
  private factories = new Map<ServiceIdentifier, () => any>();
  
  register<T>(identifier: ServiceIdentifier<T>, factory: () => T): void {
    this.factories.set(identifier, factory);
  }
  
  registerSingleton<T>(identifier: ServiceIdentifier<T>, factory: () => T): void {
    this.factories.set(identifier, factory);
    this.singletons.set(identifier, null); // Mark as singleton
  }
  
  get<T>(identifier: ServiceIdentifier<T>): T {
    if (this.singletons.has(identifier)) {
      let instance = this.singletons.get(identifier);
      if (!instance) {
        const factory = this.factories.get(identifier);
        if (!factory) throw new Error(`Service ${String(identifier)} not registered`);
        instance = factory();
        this.singletons.set(identifier, instance);
      }
      return instance;
    }
    
    const factory = this.factories.get(identifier);
    if (!factory) throw new Error(`Service ${String(identifier)} not registered`);
    return factory();
  }
  
  has(identifier: ServiceIdentifier): boolean {
    return this.factories.has(identifier);
  }
}

// Service implementations (low-level modules)
class ConsoleLogger implements Logger {
  info(message: string, meta?: Record<string, unknown>): void {
    console.info(`[INFO] ${message}`, meta ? JSON.stringify(meta) : '');
  }
  
  warn(message: string, meta?: Record<string, unknown>): void {
    console.warn(`[WARN] ${message}`, meta ? JSON.stringify(meta) : '');
  }
  
  error(message: string, error?: Error, meta?: Record<string, unknown>): void {
    console.error(`[ERROR] ${message}`, error?.stack || '', meta ? JSON.stringify(meta) : '');
  }
  
  debug(message: string, meta?: Record<string, unknown>): void {
    console.debug(`[DEBUG] ${message}`, meta ? JSON.stringify(meta) : '');
  }
}

class ProductionLogger implements Logger {
  private logLevel: 'debug' | 'info' | 'warn' | 'error';
  
  constructor(logLevel: 'debug' | 'info' | 'warn' | 'error' = 'info') {
    this.logLevel = logLevel;
  }
  
  info(message: string, meta?: Record<string, unknown>): void {
    if (this.shouldLog('info')) {
      this.sendToLogService('info', message, meta);
    }
  }
  
  warn(message: string, meta?: Record<string, unknown>): void {
    if (this.shouldLog('warn')) {
      this.sendToLogService('warn', message, meta);
    }
  }
  
  error(message: string, error?: Error, meta?: Record<string, unknown>): void {
    if (this.shouldLog('error')) {
      this.sendToLogService('error', message, { ...meta, error: error?.stack });
    }
  }
  
  debug(message: string, meta?: Record<string, unknown>): void {
    if (this.shouldLog('debug')) {
      this.sendToLogService('debug', message, meta);
    }
  }
  
  private shouldLog(level: string): boolean {
    const levels = ['debug', 'info', 'warn', 'error'];
    return levels.indexOf(level) >= levels.indexOf(this.logLevel);
  }
  
  private async sendToLogService(level: string, message: string, meta?: Record<string, unknown>): Promise<void> {
    // Send to external logging service (Datadog, New Relic, etc.)
    try {
      await fetch('/api/logs', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          level,
          message,
          meta,
          timestamp: new Date().toISOString(),
          source: 'frontend'
        })
      });
    } catch (error) {
      // Fallback to console in case logging service is down
      console.log(`[${level.toUpperCase()}] ${message}`, meta);
    }
  }
}

class FetchApiClient implements ApiClient {
  constructor(
    private baseUrl: string,
    private logger: Logger,
    private defaultHeaders: Record<string, string> = {}
  ) {}
  
  async get<T>(url: string, config?: RequestConfig): Promise<T> {
    return this.request<T>('GET', url, undefined, config);
  }
  
  async post<T>(url: string, data?: unknown, config?: RequestConfig): Promise<T> {
    return this.request<T>('POST', url, data, config);
  }
  
  async put<T>(url: string, data?: unknown, config?: RequestConfig): Promise<T> {
    return this.request<T>('PUT', url, data, config);
  }
  
  async delete<T>(url: string, config?: RequestConfig): Promise<T> {
    return this.request<T>('DELETE', url, undefined, config);
  }
  
  private async request<T>(
    method: string,
    url: string,
    data?: unknown,
    config?: RequestConfig
  ): Promise<T> {
    const fullUrl = url.startsWith('http') ? url : `${this.baseUrl}${url}`;
    const startTime = performance.now();
    
    this.logger.debug(`${method} ${fullUrl}`, { data, config });
    
    try {
      const response = await fetch(fullUrl, {
        method,
        headers: {
          'Content-Type': 'application/json',
          ...this.defaultHeaders,
          ...config?.headers
        },
        body: data ? JSON.stringify(data) : undefined,
        signal: config?.signal,
        ...config
      });
      
      const duration = performance.now() - startTime;
      
      if (!response.ok) {
        this.logger.error(`API request failed: ${method} ${fullUrl}`, 
          new Error(`HTTP ${response.status}: ${response.statusText}`),
          { duration, status: response.status }
        );
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      const result = await response.json();
      this.logger.info(`API request successful: ${method} ${fullUrl}`, { duration, status: response.status });
      
      return result;
    } catch (error) {
      const duration = performance.now() - startTime;
      this.logger.error(`API request error: ${method} ${fullUrl}`, error as Error, { duration });
      throw error;
    }
  }
}

// High-level module depending on abstractions
class OrderService {
  constructor(
    private apiClient: ApiClient,
    private cacheService: CacheService,
    private logger: Logger,
    private eventBus: EventBus
  ) {}
  
  async createOrder(orderData: CreateOrderRequest): Promise<Order> {
    this.logger.info('Creating new order', { 
      customerId: orderData.customerId, 
      itemCount: orderData.items.length 
    });
    
    try {
      const order = await this.apiClient.post<Order>('/orders', orderData);
      
      // Cache the order
      await this.cacheService.set(`order:${order.id}`, order, 3600000); // 1 hour
      
      // Emit event for other systems
      this.eventBus.emit('order:created', { orderId: order.id, customerId: orderData.customerId });
      
      this.logger.info('Order created successfully', { 
        orderId: order.id, 
        total: order.total 
      });
      
      return order;
    } catch (error) {
      this.logger.error('Failed to create order', error as Error, { 
        customerId: orderData.customerId 
      });
      throw error;
    }
  }
  
  async getOrder(orderId: string): Promise<Order | null> {
    // Try cache first
    const cached = await this.cacheService.get<Order>(`order:${orderId}`);
    if (cached) {
      this.logger.debug('Order retrieved from cache', { orderId });
      return cached;
    }
    
    try {
      const order = await this.apiClient.get<Order>(`/orders/${orderId}`);
      
      // Cache for future requests
      await this.cacheService.set(`order:${orderId}`, order, 3600000);
      
      this.logger.debug('Order retrieved from API', { orderId });
      return order;
    } catch (error) {
      this.logger.warn('Order not found', error as Error, { orderId });
      return null;
    }
  }
}

// Container configuration
const configureDependencies = (container: ServiceContainer, environment: 'development' | 'production'): void => {
  // Logger
  if (environment === 'development') {
    container.registerSingleton<Logger>('logger', () => new ConsoleLogger());
  } else {
    container.registerSingleton<Logger>('logger', () => new ProductionLogger('info'));
  }
  
  // API Client
  container.registerSingleton<ApiClient>('apiClient', () => {
    const logger = container.get<Logger>('logger');
    return new FetchApiClient(process.env.REACT_APP_API_BASE_URL || '', logger);
  });
  
  // Cache Service
  container.registerSingleton<CacheService>('cacheService', () => new MemoryCacheService());
  
  // Event Bus
  container.registerSingleton<EventBus>('eventBus', () => new InMemoryEventBus());
  
  // Business Services
  container.register<OrderService>('orderService', () => {
    const apiClient = container.get<ApiClient>('apiClient');
    const cacheService = container.get<CacheService>('cacheService');
    const logger = container.get<Logger>('logger');
    const eventBus = container.get<EventBus>('eventBus');
    
    return new OrderService(apiClient, cacheService, logger, eventBus);
  });
};

// React integration
const DIContext = createContext<ServiceContainer | null>(null);

export const DIProvider: React.FC<{ 
  container: ServiceContainer; 
  children: React.ReactNode; 
}> = ({ container, children }) => {
  return (
    <DIContext.Provider value={container}>
      {children}
    </DIContext.Provider>
  );
};

export const useService = <T>(identifier: ServiceIdentifier<T>): T => {
  const container = useContext(DIContext);
  if (!container) {
    throw new Error('useService must be used within a DIProvider');
  }
  
  return useMemo(() => container.get<T>(identifier), [container, identifier]);
};

// Usage in components
const OrderCreationForm: React.FC = () => {
  const orderService = useService<OrderService>('orderService');
  const [creating, setCreating] = useState(false);
  
  const handleSubmit = async (orderData: CreateOrderRequest) => {
    setCreating(true);
    try {
      const order = await orderService.createOrder(orderData);
      // Handle successful order creation
    } catch (error) {
      // Handle error
    } finally {
      setCreating(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
    </form>
  );
};

// Application bootstrap
const container = new DIContainer();
configureDependencies(container, process.env.NODE_ENV as 'development' | 'production');

const App: React.FC = () => {
  return (
    <DIProvider container={container}>
      <Router>
        <Routes>
          <Route path="/orders/create" element={<OrderCreationForm />} />
          {/* Other routes */}
        </Routes>
      </Router>
    </DIProvider>
  );
};
```

## ⚡ React 19 Actions & Next.js 15 Features

### React 19 Actions: Modern State Management

React 19 introduces Actions, a powerful new pattern for handling asynchronous operations and state updates:

```tsx
// React 19 Actions with automatic loading states
import { useActionState, useOptimistic } from 'react';

interface FormState {
  message: string;
  errors: Record<string, string>;
  isSubmitting: boolean;
}

async function submitUserData(formData: FormData): Promise<FormState> {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;
  
  // Simulate API call
  await new Promise(resolve => setTimeout(resolve, 1000));
  
  if (!name || !email) {
    return {
      message: '',
      errors: { name: 'Name is required', email: 'Email is required' },
      isSubmitting: false
    };
  }
  
  return {
    message: 'User created successfully!',
    errors: {},
    isSubmitting: false
  };
}

function UserForm() {
  const [state, submitAction, isPending] = useActionState(submitUserData, {
    message: '',
    errors: {},
    isSubmitting: false
  });
  
  return (
    <form action={submitAction}>
      <div>
        <label htmlFor="name">Name:</label>
        <input 
          type="text" 
          id="name" 
          name="name" 
          required 
          aria-invalid={!!state.errors.name}
          aria-describedby={state.errors.name ? 'name-error' : undefined}
        />
        {state.errors.name && (
          <div id="name-error" role="alert">
            {state.errors.name}
          </div>
        )}
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input 
          type="email" 
          id="email" 
          name="email" 
          required 
          aria-invalid={!!state.errors.email}
          aria-describedby={state.errors.email ? 'email-error' : undefined}
        />
        {state.errors.email && (
          <div id="email-error" role="alert">
            {state.errors.email}
          </div>
        )}
      </div>
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create User'}
      </button>
      
      {state.message && (
        <div role="status">{state.message}</div>
      )}
    </form>
  );
}
```

### Optimistic Updates with React 19

```tsx
// Optimistic updates for better UX
function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: Todo) => [...state, newTodo]
  );
  
  async function addTodo(formData: FormData) {
    const text = formData.get('text') as string;
    const newTodo: Todo = {
      id: Math.random().toString(),
      text,
      completed: false,
      createdAt: new Date()
    };
    
    // Optimistically add the todo
    addOptimisticTodo(newTodo);
    
    try {
      await fetch('/api/todos', {
        method: 'POST',
        body: JSON.stringify(newTodo)
      });
    } catch (error) {
      // Handle error - optimistic update will be reverted
      console.error('Failed to add todo:', error);
    }
  }
  
  return (
    <div>
      <form action={addTodo}>
        <input name="text" placeholder="Add a todo..." required />
        <button type="submit">Add Todo</button>
      </form>
      
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id}>
            {todo.text}
            {todo.id.includes('.') && <span> (Saving...)</span>}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Next.js 15 with Turbopack

```typescript
// next.config.ts - TypeScript support in Next.js 15
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  // Turbopack for development (stable in Next.js 15)
  experimental: {
    turbo: {
      rules: {
        '*.svg': {
          loaders: ['@svgr/webpack'],
          as: '*.js',
        },
      },
    },
  },
  
  // Enhanced caching with Next.js 15
  cacheHandler: require.resolve('./cache-handler.js'),
  cacheMaxMemorySize: 0, // Disable in-memory caching
  
  // Async request APIs
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=3600, stale-while-revalidate=86400',
          },
        ],
      },
    ];
  },
  
  // Enhanced error handling
  onDemandEntries: {
    maxInactiveAge: 25 * 1000,
    pagesBufferLength: 2,
  },
};

export default nextConfig;
```

### React 19 Custom Elements Support

```tsx
// React 19 supports custom elements natively
function WebComponentIntegration() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <h2>React 19 + Web Components</h2>
      
      {/* Custom elements work seamlessly */}
      <my-counter 
        value={count}
        onIncrement={() => setCount(c => c + 1)}
        onDecrement={() => setCount(c => c - 1)}
      />
      
      {/* React components can receive custom element events */}
      <my-chart 
        data={chartData}
        onDataChange={(event) => {
          console.log('Chart data changed:', event.detail);
        }}
      />
    </div>
  );
}

// Custom element definition
class MyCounter extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `
      <button id="decrement">-</button>
      <span id="value">0</span>
      <button id="increment">+</button>
    `;
    
    this.value = 0;
    this.setupEventListeners();
  }
  
  setupEventListeners() {
    this.querySelector('#increment')?.addEventListener('click', () => {
      this.value++;
      this.dispatchEvent(new CustomEvent('increment', {
        detail: { value: this.value }
      }));
    });
    
    this.querySelector('#decrement')?.addEventListener('click', () => {
      this.value--;
      this.dispatchEvent(new CustomEvent('decrement', {
        detail: { value: this.value }
      }));
    });
  }
  
  set value(newValue) {
    this._value = newValue;
    this.querySelector('#value')!.textContent = newValue.toString();
  }
  
  get value() {
    return this._value;
  }
}

customElements.define('my-counter', MyCounter);
```

## 🔄 Advanced Error Handling Patterns

### Result Pattern for Type-Safe Error Handling

```tsx
// Modern Result type with error information
type Result<T, E = Error> = {
  success: true;
  data: T;
  metadata?: {
    timestamp: string;
    source: string;
    version: string;
  };
} | {
  success: false;
  error: E;
  metadata?: {
    timestamp: string;
    source: string;
    correlationId: string;
    retryable: boolean;
  };
};

// Error types for better categorization
class NetworkError extends Error {
  readonly type = 'NetworkError';
  constructor(message: string, public readonly status?: number) {
    super(message);
    this.name = 'NetworkError';
  }
}

class ValidationError extends Error {
  readonly type = 'ValidationError';
  constructor(message: string, public readonly fields: Record<string, string>) {
    super(message);
    this.name = 'ValidationError';
  }
}

class BusinessLogicError extends Error {
  readonly type = 'BusinessLogicError';
  constructor(message: string, public readonly code: string) {
    super(message);
    this.name = 'BusinessLogicError';
  }
}

// Safe async wrapper with error handling
export const safeAsync = async <T>(
  operation: () => Promise<T>,
  context?: {
    source: string;
    correlationId?: string;
    retryable?: boolean;
  }
): Promise<Result<T>> => {
  const timestamp = new Date().toISOString();
  const correlationId = context?.correlationId || crypto.randomUUID();
  
  try {
    const data = await operation();
    
    return {
      success: true,
      data,
      metadata: {
        timestamp,
        source: context?.source || 'unknown',
        version: '1.0'
      }
    };
  } catch (error) {
    const normalizedError = error instanceof Error ? error : new Error(String(error));
    
    return {
      success: false,
      error: normalizedError,
      metadata: {
        timestamp,
        source: context?.source || 'unknown',
        correlationId,
        retryable: context?.retryable ?? isRetryableError(normalizedError)
      }
    };
  }
};

// Error classification
const isRetryableError = (error: Error): boolean => {
  if (error instanceof NetworkError) {
    return error.status ? error.status >= 500 : true;
  }
  
  if (error instanceof BusinessLogicError) {
    return false; // Business logic errors are typically not retryable
  }
  
  // Check for common retryable error patterns
  const retryablePatterns = [
    /timeout/i,
    /network/i,
    /connection/i,
    /temporary/i
  ];
  
  return retryablePatterns.some(pattern => pattern.test(error.message));
};

// Hook for async operations with automatic retry
export const useAsyncOperation = <T>(
  operation: () => Promise<T>,
  options: {
    dependencies?: React.DependencyList;
    retryCount?: number;
    retryDelay?: number;
    source?: string;
    onError?: (error: Error) => void;
    onSuccess?: (data: T) => void;
  } = {}
) => {
  const {
    dependencies = [],
    retryCount = 3,
    retryDelay = 1000,
    source = 'useAsyncOperation',
    onError,
    onSuccess
  } = options;
  
  const [result, setResult] = useState<Result<T> | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [attempt, setAttempt] = useState(0);
  
  const executeOperation = useCallback(async (attemptNumber = 0) => {
    setIsLoading(true);
    setAttempt(attemptNumber + 1);
    
    const operationResult = await safeAsync(operation, {
      source,
      correlationId: crypto.randomUUID(),
      retryable: true
    });
    
    if (operationResult.success) {
      setResult(operationResult);
      onSuccess?.(operationResult.data);
    } else {
      // Check if we should retry
      if (
        attemptNumber < retryCount &&
        operationResult.metadata?.retryable &&
        isRetryableError(operationResult.error)
      ) {
        // Exponential backoff
        const delay = retryDelay * Math.pow(2, attemptNumber);
        setTimeout(() => {
          executeOperation(attemptNumber + 1);
        }, delay);
        return;
      }
      
      setResult(operationResult);
      onError?.(operationResult.error);
    }
    
    setIsLoading(false);
  }, [operation, retryCount, retryDelay, source, onError, onSuccess]);
  
  useEffect(() => {
    executeOperation();
  }, dependencies);
  
  const retry = useCallback(() => {
    executeOperation(0);
  }, [executeOperation]);
  
  return {
    result,
    isLoading,
    attempt,
    retry,
    isSuccess: result?.success === true,
    isError: result?.success === false,
    data: result?.success === true ? result.data : null,
    error: result?.success === false ? result.error : null
  };
};
```

### Advanced Error Boundary with Recovery Strategies

```tsx
// Error boundary with sophisticated recovery
interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
  errorInfo: React.ErrorInfo | null;
  retryCount: number;
  errorId: string;
  recoveryStrategy: 'retry' | 'fallback' | 'refresh' | 'report';
}

interface ErrorRecoveryStrategy {
  canRecover(error: Error, errorInfo: React.ErrorInfo): boolean;
  getStrategy(): 'retry' | 'fallback' | 'refresh' | 'report';
  getRetryDelay(retryCount: number): number;
  shouldAutoRecover(retryCount: number): boolean;
}

class ChunkLoadErrorRecovery implements ErrorRecoveryStrategy {
  canRecover(error: Error): boolean {
    return error.message.includes('ChunkLoadError') ||
           error.message.includes('Loading chunk');
  }
  
  getStrategy(): 'retry' | 'fallback' | 'refresh' | 'report' {
    return 'refresh';
  }
  
  getRetryDelay(): number {
    return 1000;
  }
  
  shouldAutoRecover(retryCount: number): boolean {
    return retryCount < 2;
  }
}

class NetworkErrorRecovery implements ErrorRecoveryStrategy {
  canRecover(error: Error): boolean {
    return error.message.includes('fetch') ||
           error.message.includes('NetworkError') ||
           error.message.includes('timeout');
  }
  
  getStrategy(): 'retry' | 'fallback' | 'refresh' | 'report' {
    return 'retry';
  }
  
  getRetryDelay(retryCount: number): number {
    return Math.min(1000 * Math.pow(2, retryCount), 10000);
  }
  
  shouldAutoRecover(retryCount: number): boolean {
    return retryCount < 3;
  }
}

class RenderErrorRecovery implements ErrorRecoveryStrategy {
  canRecover(error: Error, errorInfo: React.ErrorInfo): boolean {
    return errorInfo.componentStack.includes('useState') ||
           errorInfo.componentStack.includes('useEffect');
  }
  
  getStrategy(): 'retry' | 'fallback' | 'refresh' | 'report' {
    return 'fallback';
  }
  
  getRetryDelay(): number {
    return 0;
  }
  
  shouldAutoRecover(): boolean {
    return false;
  }
}

export class ResilientErrorBoundary extends React.Component<
  {
    children: React.ReactNode;
    fallback?: React.ComponentType<{ error: Error; retry: () => void; errorId: string }>;
    maxRetries?: number;
    onError?: (error: Error, errorInfo: React.ErrorInfo, errorId: string) => void;
    recoveryStrategies?: ErrorRecoveryStrategy[];
  },
  ErrorBoundaryState
> {
  private retryTimeoutId: NodeJS.Timeout | null = null;
  private recoveryStrategies: ErrorRecoveryStrategy[];
  
  constructor(props: any) {
    super(props);
    
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
      retryCount: 0,
      errorId: '',
      recoveryStrategy: 'report'
    };
    
    this.recoveryStrategies = props.recoveryStrategies || [
      new ChunkLoadErrorRecovery(),
      new NetworkErrorRecovery(),
      new RenderErrorRecovery()
    ];
  }
  
  static getDerivedStateFromError(error: Error): Partial<ErrorBoundaryState> {
    return {
      hasError: true,
      error,
      errorId: crypto.randomUUID()
    };
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    this.setState({ errorInfo });
    
    // Find appropriate recovery strategy
    const strategy = this.recoveryStrategies.find(s => s.canRecover(error, errorInfo));
    const recoveryStrategy = strategy?.getStrategy() || 'report';
    
    this.setState({ recoveryStrategy });
    
    // Report error to monitoring service
    this.reportError(error, errorInfo);
    
    // Handle error based on strategy
    if (strategy && strategy.shouldAutoRecover(this.state.retryCount)) {
      this.scheduleRecovery(strategy, error);
    }
    
    // Call error callback
    this.props.onError?.(error, errorInfo, this.state.errorId);
  }
  
  componentWillUnmount() {
    if (this.retryTimeoutId) {
      clearTimeout(this.retryTimeoutId);
    }
  }
  
  private scheduleRecovery(strategy: ErrorRecoveryStrategy, error: Error) {
    const delay = strategy.getRetryDelay(this.state.retryCount);
    
    this.retryTimeoutId = setTimeout(() => {
      this.handleRecovery(strategy.getStrategy());
    }, delay);
  }
  
  private handleRecovery = (strategy: 'retry' | 'fallback' | 'refresh' | 'report') => {
    switch (strategy) {
      case 'retry':
        this.setState(prevState => ({
          hasError: false,
          error: null,
          errorInfo: null,
          retryCount: prevState.retryCount + 1
        }));
        break;
        
      case 'refresh':
        window.location.reload();
        break;
        
      case 'fallback':
        // Keep error state but show fallback UI
        break;
        
      case 'report':
      default:
        // Just report, no automatic recovery
        break;
    }
  };
  
  private reportError = (error: Error, errorInfo: React.ErrorInfo) => {
    const errorReport = {
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      errorId: this.state.errorId,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      url: window.location.href,
      retryCount: this.state.retryCount
    };
    
    // Send to error tracking service
    if (window.Sentry) {
      window.Sentry.captureException(error, {
        contexts: {
          react: errorInfo,
          errorBoundary: errorReport
        }
      });
    }
    
    // Send to custom analytics
    if (window.analytics) {
      window.analytics.track('Error Boundary Triggered', errorReport);
    }
    
    // Console error for development
    if (process.env.NODE_ENV === 'development') {
      console.group('🚨 Error Boundary');
      console.error('Error:', error);
      console.error('Error Info:', errorInfo);
      console.error('Error Report:', errorReport);
      console.groupEnd();
    }
  };
  
  private handleRetry = () => {
    this.handleRecovery('retry');
  };
  
  render() {
    if (this.state.hasError && this.state.error) {
      // Custom fallback component
      if (this.props.fallback) {
        const FallbackComponent = this.props.fallback;
        return (
          <FallbackComponent
            error={this.state.error}
            retry={this.handleRetry}
            errorId={this.state.errorId}
          />
        );
      }
      
      // Default fallback UI
      return (
        <div className="error-boundary">
          <div className="error-boundary__content">
            <h2>Something went wrong</h2>
            <p>We're sorry, but something unexpected happened.</p>
            
            {process.env.NODE_ENV === 'development' && (
              <details className="error-boundary__details">
                <summary>Error Details (Development)</summary>
                <pre>{this.state.error.stack}</pre>
                {this.state.errorInfo && (
                  <pre>{this.state.errorInfo.componentStack}</pre>
                )}
              </details>
            )}
            
            <div className="error-boundary__actions">
              <button onClick={this.handleRetry} className="btn btn-primary">
                Try Again ({this.state.retryCount + 1}/{this.props.maxRetries || 3})
              </button>
              
              <button onClick={() => window.location.reload()} className="btn btn-secondary">
                Refresh Page
              </button>
              
              <button 
                onClick={() => window.location.href = '/'}
                className="btn btn-secondary"
              >
                Go Home
              </button>
            </div>
            
            <p className="error-boundary__id">
              Error ID: {this.state.errorId}
            </p>
          </div>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage with custom fallback
const CustomErrorFallback: React.FC<{
  error: Error;
  retry: () => void;
  errorId: string;
}> = ({ error, retry, errorId }) => {
  const [reportSent, setReportSent] = useState(false);
  
  const sendFeedback = async (feedback: string) => {
    try {
      await fetch('/api/error-feedback', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          errorId,
          feedback,
          timestamp: new Date().toISOString()
        })
      });
      setReportSent(true);
    } catch (err) {
      console.error('Failed to send feedback:', err);
    }
  };
  
  return (
    <div className="error-fallback">
      <h2>Oops! Something went wrong</h2>
      <p>We apologize for the inconvenience. Our team has been notified.</p>
      
      <div className="error-actions">
        <button onClick={retry}>Try Again</button>
        <button onClick={() => window.location.reload()}>Refresh</button>
      </div>
      
      {!reportSent && (
        <div className="error-feedback">
          <p>Help us improve by sharing what you were doing:</p>
          <textarea
            placeholder="Optional: Describe what you were trying to do..."
            onBlur={(e) => e.target.value && sendFeedback(e.target.value)}
          />
        </div>
      )}
      
      {reportSent && (
        <p className="feedback-sent">Thank you for your feedback!</p>
      )}
    </div>
  );
};

// App-level error boundary setup
const App: React.FC = () => (
  <ResilientErrorBoundary
    fallback={CustomErrorFallback}
    maxRetries={3}
    onError={(error, errorInfo, errorId) => {
      // Custom error handling
      console.log('Error boundary caught:', { error, errorInfo, errorId });
    }}
  >
    <Router>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/dashboard" element={
          <ResilientErrorBoundary fallback={DashboardErrorFallback}>
            <Dashboard />
          </ResilientErrorBoundary>
        } />
      </Routes>
    </Router>
  </ResilientErrorBoundary>
);
```

This comprehensive enhancement of Module 5 covers:

1. **Advanced SOLID Principles** - Modern implementations with TypeScript and React 19
2. **Sophisticated Error Handling** - Result patterns, recovery strategies, and resilient error boundaries
3. **Dependency Injection** - Enterprise-grade IoC container with React integration

The content builds upon the existing foundation while incorporating cutting-edge patterns and practices. Should I continue with the remaining sections of this module?