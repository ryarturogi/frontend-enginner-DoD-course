# Module 9: Team & Workflow Alignment

## 🎯 Learning Objectives

By the end of this module, you will:
- Integrate Definition of Done effectively within Scrum and Kanban workflows
- Master pull request best practices that enforce quality standards
- Implement branching strategies that support team collaboration and deployment
- Set up CI/CD pipelines that automatically enforce DoD compliance
- Create team processes that scale with growth and maintain quality
- Align engineering practices with business delivery requirements

## 🏃‍♂️ DoD in Agile Methodologies

### DoD in Scrum Framework

#### Sprint Planning Integration

```typescript
// Sprint planning DoD checklist template
interface SprintPlanningDoD {
  storyReadiness: {
    acceptanceCriteriaDefined: boolean;
    designsApproved: boolean;
    technicalApproachAgreed: boolean;
    dependenciesIdentified: boolean;
    estimationCompleted: boolean;
  };
  
  teamCapacity: {
    developmentCapacityCalculated: boolean;
    testingCapacityAllocated: boolean;
    reviewCapacityReserved: boolean;
    bufferForUnplannedWork: boolean;
  };
  
  qualityStandards: {
    testingStrategyDefined: boolean;
    performanceTargetsSet: boolean;
    accessibilityRequirementsCleared: boolean;
    securityConsiderationsAddressed: boolean;
  };
}

// Scrum DoD workflow implementation
class ScrumDoD {
  private stories: UserStory[];
  private dodCriteria: DoDCriteria;
  
  constructor(stories: UserStory[], dodCriteria: DoDCriteria) {
    this.stories = stories;
    this.dodCriteria = dodCriteria;
  }
  
  /**
   * Validate story readiness for sprint commitment
   */
  validateSprintCommitment(): SprintValidationResult {
    const results = this.stories.map(story => ({
      storyId: story.id,
      readinessScore: this.calculateReadinessScore(story),
      blockers: this.identifyBlockers(story),
      dodCompliance: this.assessDoDCompliance(story)
    }));
    
    const overallReadiness = this.calculateOverallReadiness(results);
    const sprintRisk = this.assessSprintRisk(results);
    
    return {
      canCommit: overallReadiness >= 0.8 && sprintRisk === 'low',
      readinessScore: overallReadiness,
      riskLevel: sprintRisk,
      storyResults: results,
      recommendations: this.generateRecommendations(results)
    };
  }
  
  /**
   * Generate daily standup DoD reminders
   */
  generateStandupReminders(story: UserStory): StandupReminder[] {
    const reminders: StandupReminder[] = [];
    
    // Check progress against DoD
    const compliance = this.assessDoDCompliance(story);
    
    if (!compliance.codeReview && story.status === 'in-review') {
      reminders.push({
        type: 'warning',
        message: `Story ${story.id} needs code review before moving to testing`,
        actionRequired: 'Schedule code review session'
      });
    }
    
    if (!compliance.testingComplete && story.status === 'testing') {
      reminders.push({
        type: 'info',
        message: `Story ${story.id} testing in progress - ensure all DoD test criteria are met`,
        actionRequired: 'Review test checklist'
      });
    }
    
    if (story.daysInStatus > 2) {
      reminders.push({
        type: 'alert',
        message: `Story ${story.id} has been in ${story.status} for ${story.daysInStatus} days`,
        actionRequired: 'Investigate potential blockers'
      });
    }
    
    return reminders;
  }
  
  /**
   * Sprint review DoD validation
   */
  validateSprintReview(): SprintReviewResult {
    const completedStories = this.stories.filter(s => s.status === 'done');
    const dodCompliance = completedStories.map(story => ({
      storyId: story.id,
      dodScore: this.calculateDoDScore(story),
      missingCriteria: this.findMissingDoDCriteria(story),
      qualityMetrics: this.collectQualityMetrics(story)
    }));
    
    const sprintDoD = this.assessSprintDoD(dodCompliance);
    
    return {
      storiesCompleted: completedStories.length,
      dodComplianceRate: sprintDoD.complianceRate,
      qualityScore: sprintDoD.qualityScore,
      storiesReadyForRelease: dodCompliance.filter(s => s.dodScore >= 0.9).length,
      improvementAreas: sprintDoD.improvementAreas,
      recommendations: this.generateSprintRecommendations(sprintDoD)
    };
  }
}
```

#### Sprint Retrospective DoD Integration

```typescript
// Retrospective template focusing on DoD effectiveness
interface DoDRetrospective {
  dodEffectiveness: {
    criteriaClarity: number; // 1-5 scale
    criteriaRelevance: number;
    teamAlignment: number;
    automationLevel: number;
  };
  
  qualityMetrics: {
    defectEscapeRate: number;
    reworkPercentage: number;
    cycleTime: number;
    customerSatisfaction: number;
  };
  
  processImprovements: {
    dodUpdatesNeeded: string[];
    toolingImprovements: string[];
    trainingNeeds: string[];
    automationOpportunities: string[];
  };
}

class DoDRetrospectiveFacilitator {
  
  generateRetrospectiveInsights(sprintData: SprintData[]): DoDInsights {
    const trends = this.analyzeTrends(sprintData);
    
    return {
      dodEvolution: this.trackDoDEvolution(sprintData),
      qualityTrends: this.analyzeQualityTrends(sprintData),
      teamMaturity: this.assessTeamMaturity(sprintData),
      actionItems: this.generateActionItems(trends),
      experiments: this.suggestExperiments(trends)
    };
  }
  
  createActionPlan(insights: DoDInsights): ActionPlan {
    return {
      immediate: [
        'Update DoD checklist based on recent failures',
        'Automate manual DoD checks where possible',
        'Schedule DoD training for new team members'
      ],
      nextSprint: [
        'Implement automated DoD validation in CI/CD',
        'Create DoD dashboard for real-time visibility',
        'Establish DoD compliance metrics baseline'
      ],
      longTerm: [
        'Integrate DoD metrics into team OKRs',
        'Develop DoD maturity assessment framework',
        'Create cross-team DoD standards'
      ]
    };
  }
}
```

### DoD in Kanban Workflow

```typescript
// Kanban DoD implementation with flow metrics
class KanbanDoD {
  private board: KanbanBoard;
  private flowMetrics: FlowMetrics;
  
  constructor(board: KanbanBoard) {
    this.board = board;
    this.flowMetrics = new FlowMetrics(board);
  }
  
  /**
   * Implement DoD gates between columns
   */
  configureColumnGates(): ColumnGate[] {
    return [
      {
        from: 'development',
        to: 'code-review',
        dodRequirements: [
          'All unit tests passing',
          'Code coverage > 80%',
          'No linting errors',
          'Feature flag implemented if needed'
        ],
        automatedChecks: [
          'ci-pipeline-passed',
          'coverage-threshold-met',
          'lint-checks-passed'
        ]
      },
      {
        from: 'code-review',
        to: 'testing',
        dodRequirements: [
          'Code reviewed by senior developer',
          'All review comments addressed',
          'Architecture approved',
          'Documentation updated'
        ],
        automatedChecks: [
          'pr-approved',
          'comments-resolved',
          'docs-updated'
        ]
      },
      {
        from: 'testing',
        to: 'done',
        dodRequirements: [
          'All test cases passed',
          'Performance benchmarks met',
          'Accessibility requirements verified',
          'Security scan passed',
          'Product owner acceptance'
        ],
        automatedChecks: [
          'test-suite-passed',
          'performance-check-passed',
          'security-scan-clean',
          'a11y-tests-passed'
        ]
      }
    ];
  }
  
  /**
   * Monitor DoD compliance across the flow
   */
  monitorFlowHealth(): FlowHealthReport {
    const blockedItems = this.identifyBlockedItems();
    const dodViolations = this.detectDoDViolations();
    const flowMetrics = this.flowMetrics.calculate();
    
    return {
      flowEfficiency: flowMetrics.flowEfficiency,
      averageCycleTime: flowMetrics.avgCycleTime,
      dodComplianceRate: this.calculateDoDCompliance(),
      blockedItemsCount: blockedItems.length,
      qualityGateFailures: dodViolations.length,
      recommendations: this.generateFlowRecommendations(flowMetrics, dodViolations)
    };
  }
  
  /**
   * Implement WIP limits with DoD considerations
   */
  optimizeWIPLimits(): WIPOptimization {
    const currentLimits = this.board.getWIPLimits();
    const dodComplexity = this.analyzeDoDComplexity();
    const teamCapacity = this.assessTeamCapacity();
    
    return {
      recommendedLimits: this.calculateOptimalWIP(currentLimits, dodComplexity, teamCapacity),
      rationaleReport: this.generateWIPRationale(dodComplexity),
      expectedImpact: this.predictWIPImpact()
    };
  }
}
```

## 🔄 Pull Request Best Practices

### Comprehensive PR Template

```markdown
# Pull Request Template

## Summary
Brief description of changes and their purpose.

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Refactoring (no functional changes, just code improvements)
- [ ] Documentation update
- [ ] Performance improvement
- [ ] Security enhancement

## Related Issues
- Fixes #(issue number)
- Relates to #(issue number)
- Part of Epic #(epic number)

## Changes Made
### Frontend Changes
- [ ] Component modifications
- [ ] New UI components
- [ ] Style updates
- [ ] Responsive design changes
- [ ] Accessibility improvements

### Backend Changes
- [ ] API endpoint changes
- [ ] Database schema changes
- [ ] Business logic updates
- [ ] Integration changes

### Infrastructure Changes
- [ ] CI/CD pipeline updates
- [ ] Configuration changes
- [ ] Deployment scripts
- [ ] Environment variables

## Definition of Done Checklist

### Code Quality
- [ ] Code follows team conventions and style guide
- [ ] All new code has appropriate comments/documentation
- [ ] Complex business logic is well-documented
- [ ] No hardcoded values (use configuration/constants)
- [ ] Error handling implemented appropriately
- [ ] Logging added for debugging/monitoring

### Testing
- [ ] Unit tests written for new functionality
- [ ] Integration tests updated/added as needed
- [ ] E2E tests updated for user-facing changes
- [ ] All tests passing locally
- [ ] Test coverage maintained/improved (>80% for new code)
- [ ] Edge cases and error scenarios tested

### Performance
- [ ] Performance impact assessed
- [ ] No performance regression introduced
- [ ] Bundle size impact measured (<10KB increase)
- [ ] Database query efficiency verified
- [ ] Memory usage optimized
- [ ] Core Web Vitals impact considered

### Security
- [ ] Input validation implemented
- [ ] Authentication/authorization verified
- [ ] No sensitive data exposed
- [ ] Security best practices followed
- [ ] Dependency vulnerabilities checked
- [ ] OWASP guidelines considered

### Accessibility
- [ ] WCAG 2.1 AA compliance verified
- [ ] Keyboard navigation tested
- [ ] Screen reader compatibility checked
- [ ] Color contrast verified (4.5:1 minimum)
- [ ] Focus management implemented
- [ ] Semantic HTML used

### Documentation
- [ ] README updated if needed
- [ ] API documentation updated
- [ ] Component documentation added/updated
- [ ] Architecture decisions documented (ADR if significant)
- [ ] Deployment notes provided if needed

### Integration
- [ ] Feature flag implemented if needed
- [ ] Database migrations included and tested
- [ ] Environment variables documented
- [ ] Monitoring/alerting configured
- [ ] Rollback plan considered

## Testing Instructions

### Manual Testing Steps
1. Step-by-step instructions for testing the changes
2. Include specific test data or scenarios
3. Mention any special setup required
4. List browsers/devices tested

### Automated Testing
- [ ] All CI checks passing
- [ ] Lighthouse score maintained (>90)
- [ ] Security scan passed
- [ ] Accessibility tests passed

## Screenshots/Videos
<!-- Include screenshots for UI changes, recordings for complex interactions -->

## Performance Impact
### Before
- Lighthouse score: X/100
- Bundle size: X KB
- Load time: X ms

### After
- Lighthouse score: Y/100
- Bundle size: Y KB
- Load time: Y ms

## Deployment Notes
- [ ] Requires database migration
- [ ] Requires environment variable updates
- [ ] Requires cache invalidation
- [ ] Requires feature flag activation
- [ ] Backward compatible
- [ ] Safe to deploy during business hours

## Rollback Plan
Describe how to rollback this change if needed:
1. Step 1
2. Step 2
3. Step 3

## Additional Context
Any additional information that reviewers should know.

## Reviewer Checklist
For reviewers to verify before approval:

### Code Review
- [ ] Code is readable and maintainable
- [ ] Architecture decisions are sound
- [ ] Performance considerations addressed
- [ ] Security implications reviewed
- [ ] Error handling is appropriate

### Testing Review
- [ ] Test coverage is adequate
- [ ] Test quality is good (not just coverage)
- [ ] Edge cases are tested
- [ ] Integration points are tested

### Documentation Review
- [ ] Code documentation is clear
- [ ] User-facing docs are updated
- [ ] API changes are documented

## Post-Merge Checklist
- [ ] Monitor deployment for errors
- [ ] Verify feature flag rollout
- [ ] Check performance metrics
- [ ] Monitor user feedback
- [ ] Update project board/tracking

---

## Review Guidelines

### For Authors
1. **Self-review first** - Review your own PR before requesting review
2. **Keep PRs small** - Aim for <500 lines of code changes
3. **Write clear descriptions** - Help reviewers understand the context
4. **Test thoroughly** - Don't rely on reviewers to catch bugs
5. **Respond promptly** - Address feedback quickly to maintain flow

### For Reviewers
1. **Review promptly** - Aim to review within 24 hours
2. **Be constructive** - Suggest improvements, don't just point out problems
3. **Focus on important issues** - Don't nitpick style if automated tools handle it
4. **Ask questions** - If something is unclear, ask for clarification
5. **Approve when ready** - Don't hold up good code for minor issues

### Review Priorities
1. **Correctness** - Does the code do what it's supposed to do?
2. **Security** - Are there any security vulnerabilities?
3. **Performance** - Will this impact performance negatively?
4. **Maintainability** - Is the code easy to understand and modify?
5. **Style** - Does it follow team conventions? (lowest priority if automated)
```

### Advanced PR Workflow

```typescript
/**
 * Automated PR workflow manager
 * Integrates with GitHub/GitLab APIs to enforce DoD compliance
 */
class PRWorkflowManager {
  private githubClient: GitHubClient;
  private dodValidator: DoDValidator;
  private metricsCollector: MetricsCollector;
  
  constructor(githubClient: GitHubClient) {
    this.githubClient = githubClient;
    this.dodValidator = new DoDValidator();
    this.metricsCollector = new MetricsCollector();
  }
  
  /**
   * Automated PR validation on creation/update
   */
  async validatePR(prNumber: number): Promise<PRValidationResult> {
    const pr = await this.githubClient.getPR(prNumber);
    const files = await this.githubClient.getPRFiles(prNumber);
    const checks = await this.githubClient.getChecks(pr.head.sha);
    
    const validation: PRValidationResult = {
      id: prNumber,
      title: pr.title,
      author: pr.user.login,
      validations: {
        titleFormat: this.validateTitle(pr.title),
        description: this.validateDescription(pr.body),
        size: this.validatePRSize(files),
        dodChecklist: this.validateDoDChecklist(pr.body),
        ciChecks: this.validateCIChecks(checks),
        conflicts: await this.checkConflicts(pr),
        reviewers: this.validateReviewers(pr)
      },
      score: 0,
      blockers: [],
      warnings: [],
      suggestions: []
    };
    
    validation.score = this.calculateValidationScore(validation.validations);
    validation.blockers = this.identifyBlockers(validation.validations);
    validation.warnings = this.identifyWarnings(validation.validations);
    validation.suggestions = this.generateSuggestions(validation.validations);
    
    // Post validation results as PR comment
    await this.postValidationComment(prNumber, validation);
    
    // Update PR status checks
    await this.updateStatusChecks(pr.head.sha, validation);
    
    return validation;
  }
  
  /**
   * Validate PR title follows conventional commits
   */
  validateTitle(title: string): ValidationCheck {
    const conventionalCommitRegex = /^(feat|fix|docs|style|refactor|perf|test|chore)(\(.+\))?: .{1,50}/;
    const isValid = conventionalCommitRegex.test(title);
    
    return {
      name: 'title-format',
      passed: isValid,
      message: isValid 
        ? 'Title follows conventional commit format' 
        : 'Title should follow conventional commit format: type(scope): description',
      severity: 'error'
    };
  }
  
  /**
   * Validate PR description has required sections
   */
  validateDescription(body: string): ValidationCheck {
    const requiredSections = [
      /## Summary/,
      /## Type of Change/,
      /## Definition of Done Checklist/,
      /## Testing Instructions/
    ];
    
    const missingSections = requiredSections.filter(regex => !regex.test(body));
    const isValid = missingSections.length === 0;
    
    return {
      name: 'description-completeness',
      passed: isValid,
      message: isValid 
        ? 'All required sections present' 
        : `Missing sections: ${missingSections.length}`,
      severity: 'warning'
    };
  }
  
  /**
   * Validate PR size is manageable
   */
  validatePRSize(files: PRFile[]): ValidationCheck {
    const totalChanges = files.reduce((sum, file) => sum + file.additions + file.deletions, 0);
    const fileCount = files.length;
    
    let severity: 'info' | 'warning' | 'error' = 'info';
    let message = `${totalChanges} lines changed across ${fileCount} files`;
    
    if (totalChanges > 1000) {
      severity = 'error';
      message += ' - Consider splitting into smaller PRs';
    } else if (totalChanges > 500) {
      severity = 'warning';
      message += ' - Large PR, ensure thorough testing';
    }
    
    return {
      name: 'pr-size',
      passed: totalChanges <= 500,
      message,
      severity
    };
  }
  
  /**
   * Track PR metrics for continuous improvement
   */
  async trackPRMetrics(prNumber: number): Promise<void> {
    const pr = await this.githubClient.getPR(prNumber);
    const timeline = await this.githubClient.getPRTimeline(prNumber);
    const reviews = await this.githubClient.getPRReviews(prNumber);
    
    const metrics: PRMetrics = {
      id: prNumber,
      author: pr.user.login,
      createdAt: new Date(pr.created_at),
      mergedAt: pr.merged_at ? new Date(pr.merged_at) : null,
      
      // Size metrics
      linesChanged: pr.additions + pr.deletions,
      filesChanged: pr.changed_files,
      
      // Review metrics
      timeToFirstReview: this.calculateTimeToFirstReview(timeline),
      totalReviewTime: this.calculateTotalReviewTime(timeline),
      reviewCycles: this.countReviewCycles(reviews),
      reviewersCount: new Set(reviews.map(r => r.user.login)).size,
      
      // Quality metrics
      dodComplianceScore: await this.calculateDoDCompliance(prNumber),
      ciFailures: this.countCIFailures(timeline),
      conflictsCount: this.countConflicts(timeline),
      
      // Outcome metrics
      merged: pr.merged,
      reverted: await this.checkIfReverted(prNumber)
    };
    
    await this.metricsCollector.recordPRMetrics(metrics);
  }
  
  /**
   * Generate PR analytics and insights
   */
  async generatePRAnalytics(timeframe: TimeFrame): Promise<PRAnalytics> {
    const metrics = await this.metricsCollector.getPRMetrics(timeframe);
    
    return {
      timeframe,
      totalPRs: metrics.length,
      
      // Efficiency metrics
      averageTimeToMerge: this.calculateAverage(metrics.map(m => m.totalReviewTime)),
      averageReviewCycles: this.calculateAverage(metrics.map(m => m.reviewCycles)),
      averagePRSize: this.calculateAverage(metrics.map(m => m.linesChanged)),
      
      // Quality metrics
      averageDoDCompliance: this.calculateAverage(metrics.map(m => m.dodComplianceScore)),
      revertRate: metrics.filter(m => m.reverted).length / metrics.length,
      ciFailureRate: this.calculateAverage(metrics.map(m => m.ciFailures)),
      
      // Team metrics
      topContributors: this.getTopContributors(metrics),
      reviewParticipation: this.calculateReviewParticipation(metrics),
      
      // Trends
      trends: this.calculateTrends(metrics),
      recommendations: this.generateRecommendations(metrics)
    };
  }
}
```

## 🌳 Branching Strategies

### Git Flow for Teams

```bash
# Git Flow implementation with DoD integration

# Feature development workflow
git flow feature start user-authentication
# ... develop feature with DoD compliance
# Run DoD checks before finishing
npm run dod:check
git flow feature finish user-authentication

# Release workflow with DoD validation
git flow release start v2.1.0
# ... final DoD verification
npm run dod:validate:release
# ... update version and changelog
git flow release finish v2.1.0

# Hotfix workflow
git flow hotfix start critical-security-fix
# ... implement fix with security DoD
npm run dod:check:security
git flow hotfix finish critical-security-fix
```

### GitHub Flow with DoD Gates

```typescript
/**
 * GitHub Flow workflow manager with DoD enforcement
 */
class GitHubFlowManager {
  private repository: Repository;
  private dodGates: DoDGate[];
  
  constructor(repository: Repository) {
    this.repository = repository;
    this.dodGates = this.configureDoDGates();
  }
  
  configureDoDGates(): DoDGate[] {
    return [
      {
        name: 'development-complete',
        branch: 'feature/*',
        checks: [
          'unit-tests-pass',
          'lint-check-pass',
          'type-check-pass',
          'coverage-threshold',
          'no-debug-code'
        ],
        required: true
      },
      {
        name: 'review-ready',
        branch: 'feature/*',
        checks: [
          'pr-description-complete',
          'dod-checklist-filled',
          'conflicts-resolved',
          'ci-pipeline-green'
        ],
        required: true
      },
      {
        name: 'merge-ready',
        branch: 'main',
        checks: [
          'code-review-approved',
          'qa-sign-off',
          'security-scan-pass',
          'performance-check-pass',
          'accessibility-verified'
        ],
        required: true
      },
      {
        name: 'release-ready',
        branch: 'main',
        checks: [
          'integration-tests-pass',
          'e2e-tests-pass',
          'smoke-tests-pass',
          'changelog-updated',
          'documentation-updated'
        ],
        required: true
      }
    ];
  }
  
  /**
   * Trunk-based development with short-lived branches
   */
  async createFeatureBranch(featureName: string): Promise<Branch> {
    // Ensure main is up to date
    await this.repository.pull('main');
    
    // Create short-lived feature branch
    const branchName = `feature/${featureName}-${Date.now()}`;
    const branch = await this.repository.createBranch(branchName, 'main');
    
    // Set up branch protection with DoD gates
    await this.setupBranchProtection(branch, this.getDoDGatesForBranch(branchName));
    
    return branch;
  }
  
  /**
   * Validate branch is ready for merge based on DoD gates
   */
  async validateMergeReadiness(branchName: string): Promise<MergeValidation> {
    const applicableGates = this.getDoDGatesForBranch(branchName);
    const validationResults: GateValidation[] = [];
    
    for (const gate of applicableGates) {
      const gateResult = await this.validateGate(gate, branchName);
      validationResults.push(gateResult);
    }
    
    const allRequired = validationResults
      .filter(r => r.gate.required)
      .every(r => r.passed);
    
    const overallScore = validationResults.reduce((sum, r) => sum + r.score, 0) / validationResults.length;
    
    return {
      canMerge: allRequired,
      overallScore,
      gateResults: validationResults,
      blockers: validationResults.filter(r => r.gate.required && !r.passed),
      warnings: validationResults.filter(r => !r.gate.required && !r.passed),
      recommendations: this.generateMergeRecommendations(validationResults)
    };
  }
}
```

### Release Branching Strategy

```yaml
# .github/workflows/release-workflow.yml
name: Release Workflow with DoD Validation

on:
  push:
    branches: [release/*]
  pull_request:
    branches: [main]
    types: [opened, synchronize]

jobs:
  dod-validation:
    runs-on: ubuntu-latest
    name: Definition of Done Validation
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Run DoD Checks
        run: |
          echo "::group::Code Quality Checks"
          pnpm run lint
          pnpm run type-check
          pnpm run format:check
          echo "::endgroup::"
          
          echo "::group::Testing Checks"
          pnpm run test:coverage
          pnpm run test:integration
          echo "::endgroup::"
          
          echo "::group::Security Checks"
          pnpm audit --audit-level moderate
          pnpm run security:scan
          echo "::endgroup::"
          
          echo "::group::Performance Checks"
          pnpm run build
          pnpm run analyze:bundle
          pnpm run lighthouse:ci
          echo "::endgroup::"
          
          echo "::group::Accessibility Checks"
          pnpm run test:a11y
          echo "::endgroup::"
      
      - name: Validate DoD Completion
        run: |
          node scripts/validate-dod.js
        env:
          PR_NUMBER: ${{ github.event.number }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Generate DoD Report
        uses: actions/upload-artifact@v3
        with:
          name: dod-report
          path: reports/dod-validation.json
  
  release-validation:
    runs-on: ubuntu-latest
    needs: dod-validation
    if: startsWith(github.ref, 'refs/heads/release/')
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Validate Release Readiness
        run: |
          echo "::group::Release DoD Validation"
          
          # Check changelog is updated
          if ! git diff --name-only HEAD~1 | grep -q "CHANGELOG.md"; then
            echo "::error::CHANGELOG.md must be updated for release"
            exit 1
          fi
          
          # Check version is bumped
          if ! git diff HEAD~1 package.json | grep -q '"version"'; then
            echo "::error::Version must be bumped in package.json"
            exit 1
          fi
          
          # Validate migration scripts if any
          if ls database/migrations/*.sql 1> /dev/null 2>&1; then
            echo "::warning::Database migrations detected - ensure rollback plan is ready"
          fi
          
          # Check documentation is updated
          if git diff --name-only HEAD~10 | grep -q "src/"; then
            if ! git diff --name-only HEAD~10 | grep -q "README.md\|docs/"; then
              echo "::warning::Code changes detected but no documentation updates"
            fi
          fi
          
          echo "::endgroup::"
      
      - name: Run E2E Tests
        run: |
          pnpm run build
          pnpm run start &
          sleep 30
          pnpm run test:e2e:ci
        env:
          CI: true
      
      - name: Create Release Notes
        run: |
          node scripts/generate-release-notes.js > release-notes.md
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Upload Release Artifacts
        uses: actions/upload-artifact@v3
        with:
          name: release-artifacts
          path: |
            dist/
            release-notes.md
            reports/
```

## ⚙️ CI/CD Integration

### Comprehensive CI Pipeline

```yaml
# .github/workflows/ci-pipeline.yml
name: Continuous Integration with DoD Enforcement

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  NODE_VERSION: '18'
  PNPM_VERSION: '8'

jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      cache-key: ${{ steps.cache-key.outputs.key }}
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Generate cache key
        id: cache-key
        run: |
          echo "key=pnpm-${{ hashFiles('**/pnpm-lock.yaml') }}" >> $GITHUB_OUTPUT
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install pnpm
        run: npm install -g pnpm@${{ env.PNPM_VERSION }}
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: |
            node_modules
            ~/.pnpm-store
          key: ${{ steps.cache-key.outputs.key }}

  code-quality:
    runs-on: ubuntu-latest
    needs: setup
    name: Code Quality Checks
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Restore dependencies
        uses: actions/cache@v3
        with:
          path: |
            node_modules
            ~/.pnpm-store
          key: ${{ needs.setup.outputs.cache-key }}
      
      - name: ESLint
        run: |
          pnpm run lint --format=@microsoft/eslint-formatter-sarif --output-file=eslint-results.sarif
        continue-on-error: true
      
      - name: Upload ESLint results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: eslint-results.sarif
      
      - name: TypeScript Check
        run: pnpm run type-check
      
      - name: Code Formatting
        run: pnpm run format:check
      
      - name: Dependency Audit
        run: pnpm audit --audit-level moderate
      
      - name: Check for TODO/FIXME
        run: |
          if grep -r "TODO\|FIXME" src/ --exclude-dir=node_modules; then
            echo "::warning::TODO/FIXME comments found - consider addressing before merge"
          fi

  testing:
    runs-on: ubuntu-latest
    needs: setup
    name: Testing Suite
    
    strategy:
      matrix:
        test-type: [unit, integration, e2e]
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Restore dependencies
        uses: actions/cache@v3
        with:
          path: |
            node_modules
            ~/.pnpm-store
          key: ${{ needs.setup.outputs.cache-key }}
      
      - name: Unit Tests
        if: matrix.test-type == 'unit'
        run: |
          pnpm run test:coverage --reporter=default --reporter=github-actions
        env:
          CI: true
      
      - name: Upload Coverage
        if: matrix.test-type == 'unit'
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true
      
      - name: Integration Tests
        if: matrix.test-type == 'integration'
        run: pnpm run test:integration
        env:
          CI: true
      
      - name: E2E Tests
        if: matrix.test-type == 'e2e'
        run: |
          pnpm run build
          pnpm run start &
          sleep 30
          pnpm run test:e2e:ci
        env:
          CI: true
      
      - name: Upload Test Results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: test-results-${{ matrix.test-type }}
          path: |
            test-results/
            coverage/

  security:
    runs-on: ubuntu-latest
    needs: setup
    name: Security Scanning
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Restore dependencies
        uses: actions/cache@v3
        with:
          path: |
            node_modules
            ~/.pnpm-store
          key: ${{ needs.setup.outputs.cache-key }}
      
      - name: Security Audit
        run: pnpm audit --audit-level moderate
      
      - name: SAST Scan
        uses: github/codeql-action/init@v2
        with:
          languages: javascript
      
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2
      
      - name: Dependency Vulnerability Scan
        uses: actions/dependency-review-action@v3
        if: github.event_name == 'pull_request'
      
      - name: Docker Security Scan
        if: hashFiles('Dockerfile') != ''
        run: |
          docker build -t app:latest .
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy:latest image app:latest

  performance:
    runs-on: ubuntu-latest
    needs: setup
    name: Performance Testing
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Restore dependencies
        uses: actions/cache@v3
        with:
          path: |
            node_modules
            ~/.pnpm-store
          key: ${{ needs.setup.outputs.cache-key }}
      
      - name: Build Application
        run: pnpm run build
        env:
          NODE_ENV: production
      
      - name: Bundle Analysis
        run: |
          pnpm run analyze:bundle --json > bundle-stats.json
          node scripts/check-bundle-size.js
      
      - name: Lighthouse CI
        run: |
          pnpm run start &
          sleep 30
          pnpm run lighthouse:ci
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
      
      - name: Upload Performance Reports
        uses: actions/upload-artifact@v3
        with:
          name: performance-reports
          path: |
            bundle-stats.json
            lighthouse-report.html

  accessibility:
    runs-on: ubuntu-latest
    needs: setup
    name: Accessibility Testing
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Restore dependencies
        uses: actions/cache@v3
        with:
          path: |
            node_modules
            ~/.pnpm-store
          key: ${{ needs.setup.outputs.cache-key }}
      
      - name: Accessibility Tests
        run: pnpm run test:a11y
      
      - name: Pa11y CI
        run: |
          pnpm run build
          pnpm run start &
          sleep 30
          pnpm run pa11y:ci
      
      - name: Upload A11y Reports
        uses: actions/upload-artifact@v3
        with:
          name: accessibility-reports
          path: pa11y-report.json

  dod-validation:
    runs-on: ubuntu-latest
    needs: [code-quality, testing, security, performance, accessibility]
    name: DoD Validation
    if: always()
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Download all artifacts
        uses: actions/download-artifact@v3
      
      - name: Validate DoD Compliance
        run: |
          node scripts/validate-dod-compliance.js
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_NUMBER: ${{ github.event.number }}
      
      - name: Generate DoD Report
        run: |
          node scripts/generate-dod-report.js > dod-report.md
      
      - name: Comment PR with DoD Status
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const report = fs.readFileSync('dod-report.md', 'utf8');
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: report
            });
      
      - name: Fail if DoD not met
        run: |
          if [ ! -f dod-passed.flag ]; then
            echo "::error::Definition of Done criteria not met"
            exit 1
          fi

  deploy-staging:
    runs-on: ubuntu-latest
    needs: dod-validation
    if: github.ref == 'refs/heads/develop' && github.event_name == 'push'
    name: Deploy to Staging
    
    environment:
      name: staging
      url: https://staging.myapp.com
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Deploy to Staging
        run: |
          echo "Deploying to staging..."
          # Deployment commands here
      
      - name: Run Smoke Tests
        run: |
          sleep 60  # Wait for deployment
          pnpm run test:smoke:staging
      
      - name: Notify Team
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Staging deployment completed'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### DoD Validation Script

```typescript
// scripts/validate-dod-compliance.js
import { GitHubAPI } from '@octokit/rest';
import { promises as fs } from 'fs';

interface DoDCriteria {
  name: string;
  required: boolean;
  automated: boolean;
  validator: () => Promise<boolean>;
}

class DoDValidator {
  private github: GitHubAPI;
  private criteria: DoDCriteria[];
  
  constructor() {
    this.github = new GitHubAPI({
      auth: process.env.GITHUB_TOKEN
    });
    this.criteria = this.setupCriteria();
  }
  
  setupCriteria(): DoDCriteria[] {
    return [
      {
        name: 'Code Review Approved',
        required: true,
        automated: true,
        validator: () => this.checkCodeReviewApproval()
      },
      {
        name: 'All Tests Passing',
        required: true,
        automated: true,
        validator: () => this.checkTestsPassing()
      },
      {
        name: 'Code Coverage > 80%',
        required: true,
        automated: true,
        validator: () => this.checkCodeCoverage()
      },
      {
        name: 'No Security Vulnerabilities',
        required: true,
        automated: true,
        validator: () => this.checkSecurityVulnerabilities()
      },
      {
        name: 'Performance Benchmarks Met',
        required: true,
        automated: true,
        validator: () => this.checkPerformanceBenchmarks()
      },
      {
        name: 'Accessibility Tests Pass',
        required: true,
        automated: true,
        validator: () => this.checkAccessibilityTests()
      },
      {
        name: 'Documentation Updated',
        required: false,
        automated: false,
        validator: () => this.checkDocumentationUpdated()
      }
    ];
  }
  
  async validateAll(): Promise<DoDValidationResult> {
    const results: DoDCriteriaResult[] = [];
    
    for (const criteria of this.criteria) {
      try {
        const passed = await criteria.validator();
        results.push({
          name: criteria.name,
          required: criteria.required,
          automated: criteria.automated,
          passed,
          error: null
        });
      } catch (error) {
        results.push({
          name: criteria.name,
          required: criteria.required,
          automated: criteria.automated,
          passed: false,
          error: error.message
        });
      }
    }
    
    const requiredPassed = results.filter(r => r.required).every(r => r.passed);
    const score = results.filter(r => r.passed).length / results.length;
    
    return {
      passed: requiredPassed,
      score,
      results,
      blockers: results.filter(r => r.required && !r.passed),
      warnings: results.filter(r => !r.required && !r.passed)
    };
  }
  
  async checkCodeReviewApproval(): Promise<boolean> {
    if (!process.env.PR_NUMBER) return true; // Not a PR
    
    const { data: reviews } = await this.github.pulls.listReviews({
      owner: process.env.GITHUB_REPOSITORY_OWNER,
      repo: process.env.GITHUB_REPOSITORY.split('/')[1],
      pull_number: parseInt(process.env.PR_NUMBER)
    });
    
    const approvedReviews = reviews.filter(review => review.state === 'APPROVED');
    return approvedReviews.length > 0;
  }
  
  async checkTestsPassing(): Promise<boolean> {
    try {
      const coverage = JSON.parse(await fs.readFile('coverage/coverage-summary.json', 'utf8'));
      const testResults = JSON.parse(await fs.readFile('test-results/results.json', 'utf8'));
      
      return testResults.success && coverage.total.lines.pct >= 80;
    } catch {
      return false;
    }
  }
  
  async checkCodeCoverage(): Promise<boolean> {
    try {
      const coverage = JSON.parse(await fs.readFile('coverage/coverage-summary.json', 'utf8'));
      return coverage.total.lines.pct >= 80;
    } catch {
      return false;
    }
  }
  
  async checkSecurityVulnerabilities(): Promise<boolean> {
    try {
      const auditResults = JSON.parse(await fs.readFile('audit-results.json', 'utf8'));
      return auditResults.vulnerabilities.high === 0 && auditResults.vulnerabilities.critical === 0;
    } catch {
      return false;
    }
  }
  
  async checkPerformanceBenchmarks(): Promise<boolean> {
    try {
      const lighthouse = JSON.parse(await fs.readFile('lighthouse-report.json', 'utf8'));
      return lighthouse.lhr.categories.performance.score >= 0.9;
    } catch {
      return false;
    }
  }
  
  async checkAccessibilityTests(): Promise<boolean> {
    try {
      const a11yResults = JSON.parse(await fs.readFile('accessibility-reports/pa11y-report.json', 'utf8'));
      return a11yResults.every(result => result.issues.length === 0);
    } catch {
      return false;
    }
  }
  
  async checkDocumentationUpdated(): Promise<boolean> {
    // This would typically check if documentation files were updated
    // when source code changes were made
    return true; // Placeholder
  }
  
  async generateReport(result: DoDValidationResult): Promise<void> {
    const report = `
# Definition of Done Validation Report

## Overall Status: ${result.passed ? '✅ PASSED' : '❌ FAILED'}
**Score:** ${Math.round(result.score * 100)}%

## Required Criteria
${result.results
  .filter(r => r.required)
  .map(r => `- ${r.passed ? '✅' : '❌'} ${r.name}`)
  .join('\n')}

## Optional Criteria
${result.results
  .filter(r => !r.required)
  .map(r => `- ${r.passed ? '✅' : '⚠️'} ${r.name}`)
  .join('\n')}

${result.blockers.length > 0 ? `
## Blockers (Must Fix)
${result.blockers.map(b => `- ❌ ${b.name}: ${b.error || 'Criteria not met'}`).join('\n')}
` : ''}

${result.warnings.length > 0 ? `
## Warnings (Recommended)
${result.warnings.map(w => `- ⚠️ ${w.name}: ${w.error || 'Criteria not met'}`).join('\n')}
` : ''}

## Summary
${result.passed 
  ? 'All required Definition of Done criteria have been met. This change is ready for deployment.' 
  : 'Some required Definition of Done criteria have not been met. Please address the blockers above before proceeding.'
}
`;
    
    await fs.writeFile('dod-report.md', report);
    
    if (result.passed) {
      await fs.writeFile('dod-passed.flag', 'true');
    }
  }
}

// Main execution
async function main() {
  const validator = new DoDValidator();
  const result = await validator.validateAll();
  await validator.generateReport(result);
  
  console.log(`DoD Validation ${result.passed ? 'PASSED' : 'FAILED'}`);
  console.log(`Score: ${Math.round(result.score * 100)}%`);
  
  if (!result.passed) {
    console.log('\nBlockers:');
    result.blockers.forEach(blocker => {
      console.log(`- ${blocker.name}`);
    });
  }
}

main().catch(console.error);
```

## 📝 Exercise: Complete Team Workflow Setup

Create a comprehensive team workflow that enforces DoD compliance:

### Step 1: Define Team DoD
- Create a comprehensive DoD checklist for your team
- Establish different DoD levels for different work types
- Define automation strategy for DoD enforcement
- Set up metrics to track DoD effectiveness

### Step 2: Implement PR Workflow
- Create detailed PR templates with DoD integration
- Set up automated PR validation
- Implement review assignment automation
- Create PR analytics and reporting

### Step 3: Configure Branching Strategy
- Choose and implement appropriate branching strategy
- Set up branch protection rules
- Create automated DoD gates between branches
- Implement release workflow with DoD validation

### Step 4: Build CI/CD Pipeline
- Create comprehensive CI pipeline with DoD checks
- Implement automated testing at all levels
- Set up security and performance scanning
- Create automated deployment with rollback capabilities

### Step 5: Team Process Documentation
- Document complete workflow procedures
- Create troubleshooting guides
- Set up team training materials
- Establish continuous improvement process

**Deliverable:** A complete team workflow setup with automated DoD enforcement, comprehensive documentation, and metrics for continuous improvement.

---

**Next:** [Module 10: Capstone Project](../10-capstone-project/) - Apply everything learned by building a production-ready feature with complete DoD compliance.