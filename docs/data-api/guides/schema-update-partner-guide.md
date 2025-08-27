# Data API Schema Update Process - Partner Guide

## Overview
This document outlines how Pixellot manages schema updates for the Data API. Our coordinated approach ensures all partners have adequate time to prepare while maintaining system stability.



## Update Types and Notice Periods

### Schema Version Types
A **Major Version (X.0.0)** indicates substantial changes to the API schema that typically require partners to update their integration code. These changes may include the removal of fields, new required fields, structural modifications, or changes to existing field formats.

A **Minor Version (0.Y.0)** introduces new features or optional fields while maintaining backward compatibility. These updates enhance functionality without forcing partners to modify their existing integration code.

### Advance Notice
- **Major Version**: 30-day advance notice
- **Minor Version**: 14-day advance notice  

## Critical Bug Fixes
In cases where we identify critical structural issues that could impact system stability or data integrity, we may need to implement fixes on an accelerated timeline. When such situations arise, we will immediately notify all partners and provide detailed information about the changes and their impact. This ensures we can maintain system reliability while keeping you informed of any necessary adjustments.

## Major Version Release Process

### Timeline Overview
When we announce a major schema update, here's what you can expect:

**Day 0**: Schema update announcement with draft schema, and release document available. Request for comments email sent to partners  
**Day 7**: Partner consultation period ends  
**Day 20**: Integration testing should be completed  
**Day 25**: Final rollout email sent to customers  
**Day 30**: New schema goes live at predefined time

### Key Milestones

#### Day 0: Initial Release
- Draft schema available
- Release document published
- Request for comments email sent to partners

#### Day 7: Consultation Period End
- Partner feedback collection period concludes
- Review of partner comments and concerns

#### Day 20: Testing Completion
- All integration testing should be completed

#### Day 25: Final Rollout
- Final rollout email sent to customers
- Last chance to raise any blocking issues

#### Day 30: Go-Live
- New schema becomes active at predefined time
- System switches to new schema version

## Minor Version Release Process

### Timeline Overview
For minor schema updates, we follow a streamlined process:

**Day 0**: Schema update announcement with:
- Draft schema
- Release document
- Notification email to partners

**Day 14**: New schema goes live at predefined time

### Key Milestones

#### Day 0: Initial Release
- Draft schema available
- Release document published
- Notification email sent to partners

#### Day 14: Go-Live
- New schema becomes active at predefined time
- System switches to new schema version

## What We Provide

### Documentation Package
For each major release, you'll receive:
- **Schema Update Document**: Complete overview of changes and impact assessment
- **Technical Details**: Schema file locations and reference materials

### Schema Documentation
All schema versions and related materials are available in our public GitHub repository:
- Current and historical schema versions
- Migration guides and documentation

## Your Responsibilities

### Testing Requirements
Before confirming readiness, please ensure you have:
- [ ] Updated your integration code to handle the new schema structure
- [ ] Verified backward compatibility handling where applicable
- [ ] Conducted performance testing with larger payloads
- [ ] Updated any dependent systems or processes

### Communication Requirements
- **Report integration issues immediately** during the testing period


## Emergency Procedures

### Rollback Protocol
In rare cases where critical issues are discovered post-release:
1. We will immediately rollback to the previous schema version
2. All partners will be notified within 1 hour
3. A root cause analysis will be conducted
4. A revised timeline for re-release will be provided


## Best Practices for Partners

### Preparation
- **Start Early**: Begin integration work as soon as the draft schema is available
- **Test Thoroughly**: Create test scenarios and test thoroughly
- **Plan for Rollback**: Ensure you can quickly revert changes if needed
- **Document Changes**: Keep track of modifications made to your systems

### During Transition
- **Monitor Closely**: Watch for any unexpected behavior in your integration
- **Report Issues Quickly**: Don't wait - reach out immediately if you encounter problems
- **Stay Connected**: Participate in office hours and respond to our check-ins

### Post-Release
- **Validate Operations**: Confirm all functionality is working as expected
- **Performance Monitoring**: Check that performance meets your requirements
- **Provide Feedback**: Help us improve the process for future releases

## Frequently Asked Questions

### Q: What happens if we're not ready by the deadline?
If a partner cannot meet the deadline, we may postpone the release. However, this affects all partners, so early communication about potential delays is essential.

### Q: Can we test against the new schema before the release?
Yes, draft schemas are available immediately upon announcement.

### Q: What if we discover issues during testing?
Please report issues immediately. We'll work with you to resolve them, and if necessary, adjust the schema or timeline.

### Q: Are there any costs associated with schema updates?
No, schema updates are part of your existing Data API subscription. However, you may incur internal development costs for integration updates.

### Q: How often do major schema updates occur?
We aim to minimize breaking changes and release major versions only when necessary. Advance planning and timing of major updates will be shared during our regular partner communications.

---

This process is designed to ensure smooth transitions while maintaining the reliability and functionality of your Data API integration. We're committed to supporting you throughout each update cycle.
