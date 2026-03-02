---
name: app-store
description: Master App Store deployment - Submission, TestFlight, CI/CD, release management
sasmp_version: "1.3.0"
version: "2.0.0"
bonded_agent: 07-app-store
bond_type: PRIMARY_BOND
last_updated: "2024-12"
---

# App Store Deployment Skill

> Ship iOS apps to the App Store with confidence

## Learning Objectives

By completing this skill, you will:
- Navigate App Store Connect effectively
- Master the submission and review process
- Automate deployments with Fastlane
- Manage TestFlight beta testing
- Handle review rejections professionally

## Prerequisites

| Requirement | Level |
|-------------|-------|
| iOS Development | Intermediate |
| Apple Developer Account | Required |
| Git basics | Required |

## Curriculum

### Module 1: App Store Connect (3 hours)

**Topics:**
- App creation and configuration
- Pricing and availability
- Age ratings and content
- In-app purchase setup

### Module 2: Code Signing (4 hours)

**Topics:**
- Certificates and profiles
- Automatic vs manual signing
- Fastlane Match
- Troubleshooting signing issues

### Module 3: App Submission (4 hours)

**Topics:**
- Metadata requirements
- Screenshot specifications
- App Privacy details
- Review notes best practices

**Submission Checklist:**
```markdown
[ ] App icon (all sizes)
[ ] Screenshots (all device sizes)
[ ] Description and keywords
[ ] Privacy policy URL
[ ] Support URL
[ ] Age rating questionnaire
[ ] Export compliance
[ ] Privacy nutrition labels
```

### Module 4: TestFlight (3 hours)

**Topics:**
- Internal vs external testing
- Build distribution
- Tester feedback collection
- Beta app review

### Module 5: CI/CD with Fastlane (5 hours)

**Topics:**
- Fastlane setup
- Lanes for test/beta/release
- GitHub Actions integration
- Xcode Cloud

**Fastlane Example:**
```ruby
lane :beta do
  increment_build_number
  build_app(scheme: "MyApp")
  upload_to_testflight(
    skip_waiting_for_build_processing: true
  )
end
```

### Module 6: Review Guidelines (3 hours)

**Topics:**
- Common rejection reasons
- Guideline interpretation
- Appeal process
- Expedited review requests

**Common Rejections:**

| Reason | Guideline | Fix |
|--------|-----------|-----|
| Crashes | 2.1 | Fix and test |
| Placeholder content | 2.3.3 | Remove placeholders |
| Privacy issues | 5.1.1 | Add privacy labels |
| Login required | 4.2.3 | Add demo account |

### Module 7: Post-Launch (2 hours)

**Topics:**
- App Analytics
- Crash reports (Organizer)
- User reviews response
- Update strategy

## Assessment Criteria

| Criteria | Weight |
|----------|--------|
| Code signing mastery | 25% |
| Submission quality | 25% |
| CI/CD automation | 25% |
| Review handling | 25% |

## Timeline Reference

| Stage | Duration |
|-------|----------|
| Initial submission | 1-2 hours |
| Review (new app) | 24-48 hours |
| Review (update) | 24 hours |
| Rejection response | Same day |
| Expedited review | 24 hours |

## Skill Validation

1. **First Submission**: Submit app to TestFlight
2. **CI Pipeline**: Automated beta deployment
3. **Full Release**: App Store submission
4. **Update Cycle**: Version update with release notes
