# Module 2: Definition of Done (DoD)

## 🎯 Learning Objectives

By the end of this module, you will:
- Distinguish between Definition of Done and Acceptance Criteria
- Create comprehensive DoD checklists for different contexts
- Understand how DoD scales across team, organization, and product levels
- Apply DoD effectively in agile development workflows
- Customize DoD for different types of work (features, bugs, refactoring)

## 📖 DoD vs Acceptance Criteria

Understanding the difference between these two concepts is crucial for effective agile development.

### Acceptance Criteria
- **What:** Specific requirements for a particular user story
- **Scope:** Single story or feature
- **Owner:** Product Owner
- **Changes:** Can vary per story
- **Purpose:** Defines when a story meets business requirements

**Example Acceptance Criteria for "User Login":**
- User can log in with email and password
- Invalid credentials show error message
- Successful login redirects to dashboard
- "Remember me" option keeps user logged in
- Password reset link is available

### Definition of Done
- **What:** Quality standards that apply to ALL work
- **Scope:** Universal across the team/organization
- **Owner:** Development Team
- **Changes:** Evolves slowly, with team consensus
- **Purpose:** Ensures consistent quality and production readiness

**Example DoD for "User Login":**
- [ ] All acceptance criteria met
- [ ] Code reviewed and approved
- [ ] Unit tests written (>80% coverage)
- [ ] Integration tests pass
- [ ] Security review completed
- [ ] Accessibility tested
- [ ] Performance benchmarks met
- [ ] Documentation updated

## 🏗️ Building Your DoD: The Pyramid Approach

DoD should be structured in layers, from fundamental to advanced requirements.

```
        Advanced DoD
    (Team/Product Specific)
         /           \
    Standard DoD         
   (Organization)        
      /       \           
 Basic DoD              
(Industry)               
```

### Level 1: Basic DoD (Industry Standards)
These are non-negotiable basics that any professional software should meet:

- [ ] **Code Compiles** - No build errors
- [ ] **Code Runs** - Basic functionality works
- [ ] **Code is Committed** - Changes are in version control
- [ ] **No Debug Code** - Console.logs, debugger statements removed
- [ ] **No Commented Code** - Unless specifically documented why

### Level 2: Standard DoD (Organization)
Organization-wide standards that ensure consistency across teams:

#### Code Quality
- [ ] **Code Review** - At least one approval from senior developer
- [ ] **Linting Passes** - ESLint, Prettier, or equivalent
- [ ] **Type Safety** - TypeScript strict mode compliance
- [ ] **Naming Conventions** - Follow team standards
- [ ] **Architecture Patterns** - Consistent with codebase patterns

#### Testing
- [ ] **Unit Tests** - Minimum coverage threshold met
- [ ] **Integration Tests** - Key user flows covered
- [ ] **Test Quality** - Tests are readable and maintainable
- [ ] **All Tests Pass** - Including existing regression tests

#### Security
- [ ] **No Secrets** - No hardcoded credentials or API keys
- [ ] **Input Validation** - All user inputs properly validated
- [ ] **HTTPS Only** - Secure communication enforced
- [ ] **Dependency Check** - No known security vulnerabilities

#### Documentation
- [ ] **Code Documentation** - Complex logic explained
- [ ] **API Documentation** - Public interfaces documented
- [ ] **README Updated** - If public interfaces changed
- [ ] **Change Log** - Breaking changes documented

### Level 3: Advanced DoD (Team/Product Specific)
Tailored to specific product needs and team maturity:

#### Performance
- [ ] **Core Web Vitals** - LCP < 2.5s, FID < 100ms, CLS < 0.1
- [ ] **Lighthouse Score** - > 90 for Performance, Accessibility, Best Practices
- [ ] **Bundle Size** - Impact on bundle size analyzed and approved
- [ ] **Loading Performance** - Lazy loading implemented where appropriate

#### Accessibility
- [ ] **WCAG 2.1 AA** - Compliance verified with automated tools
- [ ] **Screen Reader** - Manually tested with screen reader
- [ ] **Keyboard Navigation** - All functionality accessible via keyboard
- [ ] **Color Contrast** - Meets minimum contrast ratios

#### User Experience
- [ ] **Responsive Design** - Works on mobile, tablet, desktop
- [ ] **Error Handling** - Graceful error states and messages
- [ ] **Loading States** - Appropriate loading indicators
- [ ] **Empty States** - Meaningful empty state designs

#### Monitoring & Observability
- [ ] **Error Tracking** - Errors logged to monitoring system
- [ ] **Performance Monitoring** - Key metrics instrumented
- [ ] **User Analytics** - Important interactions tracked
- [ ] **Health Checks** - Monitoring endpoints available

## 📋 DoD Templates by Work Type

Different types of work may require different DoD emphasis:

### Feature Development DoD
```markdown
## Feature Development DoD

### Requirements
- [ ] All acceptance criteria met
- [ ] UX/UI matches approved designs
- [ ] Product owner acceptance

### Code Quality
- [ ] Code review approved
- [ ] TypeScript strict compliance
- [ ] ESLint/Prettier passes
- [ ] No TODO comments

### Testing
- [ ] Unit tests written (>80% coverage)
- [ ] Integration tests for user flows
- [ ] E2E tests for critical paths
- [ ] Manual testing completed

### Performance
- [ ] Core Web Vitals thresholds met
- [ ] Bundle size impact < 10KB
- [ ] Loading performance tested

### Security & Accessibility
- [ ] Security review completed
- [ ] WCAG 2.1 AA compliance
- [ ] No accessibility regressions

### Documentation
- [ ] Technical documentation updated
- [ ] User-facing docs updated
- [ ] API changes documented
```

### Bug Fix DoD
```markdown
## Bug Fix DoD

### Requirements
- [ ] Root cause identified and documented
- [ ] Fix addresses root cause, not just symptoms
- [ ] Regression test written to prevent recurrence

### Code Quality
- [ ] Code review approved
- [ ] Minimal, focused changes
- [ ] No unrelated changes included

### Testing
- [ ] Reproduction test created
- [ ] Fix verified in test environment
- [ ] Related functionality regression tested
- [ ] All existing tests pass

### Verification
- [ ] Original reporter confirms fix
- [ ] QA verification in staging
- [ ] Monitoring confirms resolution

### Documentation
- [ ] Bug ticket updated with resolution
- [ ] Post-mortem completed if critical
- [ ] Knowledge base updated if applicable
```

### Refactoring DoD
```markdown
## Refactoring DoD

### Requirements
- [ ] Business functionality unchanged
- [ ] Performance impact measured and acceptable
- [ ] Migration plan documented

### Code Quality
- [ ] Code review approved
- [ ] Architecture improvement documented
- [ ] Code complexity reduced (measured)

### Testing
- [ ] All existing tests pass
- [ ] Test coverage maintained or improved
- [ ] Performance tests verify no regression

### Risk Management
- [ ] Rollback plan documented
- [ ] Feature flags used for risky changes
- [ ] Gradual rollout strategy planned

### Documentation
- [ ] Architecture decisions recorded (ADR)
- [ ] Developer documentation updated
- [ ] Team knowledge sharing completed
```

## 🎯 DoD Implementation Strategies

### 1. Start Small, Grow Gradually

**Week 1-2: Basic DoD**
```markdown
- [ ] Code compiles and runs
- [ ] Code review approved
- [ ] Basic tests written
- [ ] No debug code
```

**Week 3-4: Add Quality Gates**
```markdown
- [ ] Linting passes
- [ ] TypeScript strict mode
- [ ] Test coverage > 70%
- [ ] Documentation updated
```

**Week 5-8: Add Production Standards**
```markdown
- [ ] Performance benchmarks met
- [ ] Security review completed
- [ ] Accessibility basics verified
- [ ] Monitoring instrumented
```

### 2. Make it Visible and Trackable

#### Pull Request Template
```markdown
## DoD Checklist

### Code Quality
- [ ] Code review approved
- [ ] Linting passes
- [ ] TypeScript strict compliance
- [ ] No TODO comments

### Testing
- [ ] Unit tests written
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] All existing tests pass

### Security
- [ ] No hardcoded secrets
- [ ] Input validation implemented
- [ ] Security review completed

### Performance
- [ ] Core Web Vitals measured
- [ ] Bundle size impact assessed
- [ ] Loading performance verified

### Documentation
- [ ] Code documented
- [ ] README updated if needed
- [ ] API docs updated

## Acceptance Criteria
- [ ] [List specific requirements for this PR]

## Testing Notes
[Describe testing approach and results]

## Performance Impact
[Describe any performance considerations]
```

#### DoD Dashboard
Track DoD compliance across your team:

| Metric | Target | Current | Trend |
|--------|--------|---------|-------|
| Code Review Coverage | 100% | 98% | ↗️ |
| Test Coverage | >80% | 85% | ↗️ |
| Security Scan Pass | 100% | 100% | ➡️ |
| Performance Budget | <500KB | 480KB | ↗️ |
| Accessibility Score | >90 | 92% | ↗️ |

### 3. Automate DoD Checks

Use CI/CD to automatically verify DoD compliance:

```yaml
# .github/workflows/dod-check.yml
name: Definition of Done Check

on: [pull_request]

jobs:
  dod-check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Code Quality
        run: |
          npm run lint
          npm run type-check
          npm run build
      
      - name: Testing
        run: |
          npm run test:coverage
          npm run test:integration
      
      - name: Security
        run: |
          npm audit
          npm run security-scan
      
      - name: Performance
        run: |
          npm run lighthouse:ci
          npm run bundle-analyzer
      
      - name: Accessibility
        run: |
          npm run a11y-test
```

## 🔧 DoD Customization by Context

### Startup DoD (Move Fast, Stay Safe)
Focus on essential quality while maintaining velocity:

```markdown
### Essential DoD
- [ ] Code review (can be async)
- [ ] Basic tests (happy path)
- [ ] No secrets in code
- [ ] Core functionality works
- [ ] Performance is acceptable

### Nice-to-Have
- [ ] Comprehensive tests
- [ ] Full accessibility audit
- [ ] Performance optimization
- [ ] Comprehensive documentation
```

### Enterprise DoD (Comprehensive Coverage)
Extensive requirements for large-scale, critical systems:

```markdown
### Required DoD
- [ ] Senior engineer review
- [ ] Security architect approval
- [ ] QA testing completion
- [ ] Performance benchmarking
- [ ] Accessibility audit
- [ ] Documentation review
- [ ] Monitoring setup
- [ ] Rollback plan
- [ ] Training materials
- [ ] Compliance check
```

### Open Source DoD (Community Standards)
Focused on community contribution and long-term maintenance:

```markdown
### Community DoD
- [ ] Maintainer review
- [ ] Backward compatibility verified
- [ ] Documentation updated
- [ ] Examples provided
- [ ] Changelog updated
- [ ] Breaking changes noted
- [ ] License compliance
- [ ] Community guidelines followed
```

## 📊 Measuring DoD Effectiveness

### Leading Indicators (Preventive)
- **Code Review Coverage:** % of changes reviewed
- **Test Coverage:** % of code covered by tests
- **DoD Compliance Rate:** % of stories meeting full DoD
- **Time to DoD:** Average time from code complete to DoD

### Lagging Indicators (Reactive)
- **Production Incidents:** Bugs found in production
- **Customer Complaints:** User-reported issues
- **Technical Debt:** Code quality metrics over time
- **Team Velocity:** Story points completed per sprint

### DoD Health Metrics

```typescript
interface DoDMetrics {
  compliance: {
    overall: number; // 0-100%
    codeReview: number;
    testing: number;
    security: number;
    performance: number;
    accessibility: number;
  };
  velocity: {
    storiesCompleted: number;
    averageTimeToDoD: number; // hours
    blockers: number;
  };
  quality: {
    productionIncidents: number;
    bugEscapeRate: number; // bugs found in prod vs caught in dev
    customerSatisfaction: number;
  };
}
```

## 🎓 Real-World DoD Examples

### Example 1: E-commerce Feature DoD

**Feature:** Product Search with Filters

**DoD Checklist:**
- [ ] **Functional Requirements**
  - [ ] Search returns relevant products
  - [ ] Filters work correctly (price, category, brand)
  - [ ] Results are paginated
  - [ ] Empty state shows helpful message
  - [ ] Loading states are smooth

- [ ] **Code Quality**
  - [ ] TypeScript interfaces for all data structures
  - [ ] React components follow team patterns
  - [ ] Custom hooks for search logic
  - [ ] Error boundaries implemented
  - [ ] Code review by 2 senior developers

- [ ] **Testing**
  - [ ] Unit tests for search hook (>90% coverage)
  - [ ] Integration tests for filter combinations
  - [ ] E2E test for complete search flow
  - [ ] Performance tests for large result sets
  - [ ] Mobile testing completed

- [ ] **Performance**
  - [ ] Search debounced to reduce API calls
  - [ ] Results virtualized for large lists
  - [ ] Images lazy loaded
  - [ ] Bundle size increase < 15KB
  - [ ] LCP < 2.5s on 3G connection

- [ ] **Security**
  - [ ] Search queries sanitized
  - [ ] No PII in search analytics
  - [ ] Rate limiting implemented
  - [ ] SQL injection prevention verified

- [ ] **Accessibility**
  - [ ] Screen reader announces result counts
  - [ ] Keyboard navigation works for filters
  - [ ] Color contrast verified for filter states
  - [ ] Focus management tested

- [ ] **UX/Design**
  - [ ] Pixel-perfect to design specs
  - [ ] Responsive on all breakpoints
  - [ ] Smooth animations (60fps)
  - [ ] Error states are user-friendly

- [ ] **Monitoring**
  - [ ] Search analytics instrumented
  - [ ] Error tracking for failed searches
  - [ ] Performance monitoring enabled
  - [ ] A/B test framework ready

### Example 2: Critical Bug Fix DoD

**Bug:** Payment Process Failing for 5% of Users

**DoD Checklist:**
- [ ] **Root Cause Analysis**
  - [ ] Identified specific browser/payment combinations
  - [ ] Determined race condition in payment flow
  - [ ] Documented edge cases causing failure
  - [ ] Estimated impact on business metrics

- [ ] **Fix Implementation**
  - [ ] Added proper async/await handling
  - [ ] Implemented retry mechanism
  - [ ] Added better error logging
  - [ ] Created fallback payment flow

- [ ] **Testing**
  - [ ] Reproduction test created and passes
  - [ ] Edge cases tested manually
  - [ ] Cross-browser testing completed
  - [ ] Payment flow E2E tests updated
  - [ ] Load testing with concurrent payments

- [ ] **Verification**
  - [ ] QA verified fix in staging
  - [ ] Product owner approved solution
  - [ ] Customer support informed of resolution
  - [ ] Monitoring shows error rate decrease

- [ ] **Risk Management**
  - [ ] Feature flag controls new payment flow
  - [ ] Rollback plan documented and tested
  - [ ] Gradual rollout strategy planned
  - [ ] Incident response plan updated

- [ ] **Communication**
  - [ ] Post-mortem scheduled
  - [ ] Customer communication plan ready
  - [ ] Team retrospective completed
  - [ ] Knowledge base updated

## 💡 DoD Best Practices

### 1. **Make DoD Visible**
- Post DoD checklist in team workspace
- Include DoD in pull request templates
- Review DoD in sprint planning
- Celebrate DoD compliance achievements

### 2. **Keep DoD Living**
- Review and update DoD quarterly
- Gather team feedback on DoD effectiveness
- Adjust DoD based on production incidents
- Evolve DoD as team maturity grows

### 3. **Balance Rigor with Pragmatism**
- Allow emergency bypasses with documented technical debt
- Have different DoD levels for different work types
- Consider business context and deadlines
- Focus on outcomes, not just process compliance

### 4. **Automate What You Can**
- Use CI/CD to verify technical requirements
- Automate code quality checks
- Set up automatic security scans
- Create dashboards for DoD metrics

### 5. **Train the Team**
- Regular DoD training sessions
- Pair programming to spread knowledge
- Code review guidelines aligned with DoD
- Share DoD success stories and learnings

## 📝 Exercise: Build Your Team's DoD

Create a comprehensive DoD for your team by working through these steps:

### Step 1: Assessment
Evaluate your current state:
- What quality issues do you face in production?
- Where do bugs typically come from?
- What takes the most time to fix later?
- What causes the most customer complaints?

### Step 2: Stakeholder Input
Gather requirements from:
- **Developers:** Technical quality needs
- **QA:** Testing and verification needs
- **Product:** Business and user needs
- **DevOps:** Deployment and monitoring needs
- **Support:** Common support issues

### Step 3: Draft DoD
Create your initial DoD with:
- 5-10 essential items (non-negotiable)
- 5-10 standard items (normal requirement)
- 5-10 advanced items (high-quality threshold)

### Step 4: Trial Period
- Use DoD for 2-3 sprints
- Track compliance and pain points
- Gather team feedback
- Measure impact on quality and velocity

### Step 5: Refinement
- Remove items that don't add value
- Add items for common issues
- Adjust thresholds based on team capability
- Finalize DoD v2.0

**Deliverable:** A comprehensive DoD document that includes:
1. DoD checklist organized by category
2. Automation strategy for technical checks
3. Process for handling DoD exceptions
4. Metrics for measuring DoD effectiveness

---

**Next:** [Module 3: Testing Patterns for Frontend](../03-testing-patterns/) - Master comprehensive testing strategies and patterns.