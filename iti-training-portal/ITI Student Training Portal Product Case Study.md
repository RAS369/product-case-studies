# ITI Student Training Portal: Product Case Study

**Role:** solo (business analyst, product owner, developer, admin) · **Context:** final project, ITI × Deloitte ServiceNow programme, 2026 · **Platform:** ServiceNow scoped app · **Outcome:** working end-to-end system.

## Summary

I defined, designed and built a single ServiceNow app that replaces the paper, email and WhatsApp processes ITI used to run its training tracks. It covers onboarding, complaints, disciplinary warnings, assignments and grades for three groups of users: students, instructors and coordinators.

No brief was given; I chose the problem from my own experience as a trainee. The product decisions in this document were mine, and so was the build.

## Problem

At ITI, complaints, warnings and student records moved through paper, email and WhatsApp messages, which were often lost or ignored. As a trainee, I saw the effects first-hand:

- **No complaint path.** Issues were raised informally, with no routing to the right person, no escalation and no record.
- **Untracked warnings.** Warnings were informal, so nobody could reliably tell when a student was at risk.
- **Scattered records.** No single place linked a student to their track, instructor, assignments, grades and warnings.
- **Manual onboarding.** Accounts and access were set up by hand for each student, inviting duplicates and errors.
- **No visibility.** Coordinators had no view of enrolment, complaint volumes or at-risk students.

**Evidence:** my own experience as a trainee, which is a sample of one. I did not interview coordinators or instructors.

## Users

| User group | Role | Main need | What they can do |
| --- | --- | --- | --- |
| Student | iti\_student | Raise issues and see their own progress | Branded self-service portal only: own profile, assignments, grades, complaints, and their track's learning materials |
| Instructor | iti\_instructor | Manage their track without admin work | Staff portal: create assignments, record grades, issue warnings (pending approval), handle and escalate complaints |
| Coordinator | iti\_coordinator | Oversight and the final say on sensitive actions | Everything instructors can do, plus onboard students, approve or reject warnings, receive escalations, dashboard |

An admin role maintains the application.

## Requirements

I turned the problems into 11 functional requirements (F-01 to F-11) and four non-functional areas: security, performance, usability and maintainability. The guiding rule was that **each user sees and does only what their role needs**.

| Problem | Requirement it drove |
| --- | --- |
| No complaint path | Structured complaint intake, automatic routing, escalation to a coordinator |
| Untracked warnings | Warning engine with coordinator approval and an automatic at-risk flag |
| Scattered records | One student profile linked to track, assignments, grades and warnings |
| Manual onboarding | One-action onboarding with duplicate and invalid-ID checks |
| No visibility | Coordinator dashboard: enrolment, complaints, grades, at-risk students |

## Solution

**Complaint routing**

1. A student submits a complaint through the portal. The form shows only relevant fields (for example, a lab number appears only for lab issues), and duplicate complaints are blocked.
2. Complaints about an instructor's conduct go straight to the coordinators. All others go to the track's instructor.
3. The instructor resolves the complaint or escalates it to a coordinator.

**Warning engine**

1. An instructor issues a warning from the student's profile. It starts as pending.
2. A coordinator approves or rejects it. A rejection requires a reason and notifies the instructor.
3. On approval, the student is notified and their approved-warning count goes up.
4. At three approved warnings, the student is automatically flagged at-risk and the coordinator is notified.

**Also delivered:** one-action onboarding (account, role, track access, profile and welcome email); track-restricted learning materials; assignments sent to every student in a track, with due-today and overdue alerts; automatic letter grades and running averages; a coordinator dashboard; and branded email notifications for every step.

## Decisions and trade-offs

| Decision | Why | What it cost |
| --- | --- | --- |
| A coordinator must approve every warning | A warning affects a student's record, so no single instructor's judgement should count alone. A second reviewer protects students from mistaken or biased warnings. | Warnings take longer to take effect, and coordinators carry more workload. |
| Complaints about an instructor skip that instructor | An instructor should never judge a complaint about themselves. | Coordinators receive these directly, adding to their queue. |
| Only approved warnings count toward at-risk | Rejected or pending warnings should never trigger consequences. | At-risk status can lag while warnings wait for approval. |
| Students use a portal only, never the platform interface | A simpler experience, and they can't reach data outside their role. | Two portals to build and maintain (student and staff). |
| One place for lifecycle logic | Two components were tracking warning counts at the same time, which risked inconsistent states. I moved everything into one flow. | Some rework late in the build. |

**Deferred, with reasons:** complaint SLAs and auto-escalation, a virtual agent, single sign-on and mobile, automated grade import, and trend analytics. Each adds value, but none was needed to prove the core workflows within a solo, time-boxed project.

## Testing and results

I tested each workflow with positive, negative and edge cases, such as duplicate and invalid national IDs, and a student opening another track's materials. I tested every role by logging in as that user rather than as admin, because admin access bypasses security rules and would hide access defects.

Five end-to-end scenarios passed: onboarding, track-restricted learning access, the complaint lifecycle, the warning-to-at-risk engine, and assignments and grades.

**Result:** a fully working system, demonstrated end to end.

**What it did not prove:** it was never used by real students or staff, so there are no usage or outcome numbers. I have evidence that it works, not that it improves outcomes.

## The rebuild: what I'd change and how I'd measure it

**What changes**

1. **Start with the real users.** Interview 3–5 coordinators and instructors before building, to test whether my trainee's-eye view of the problem matches theirs.
2. **Complaint SLAs with automatic escalation**, so unresolved complaints reach a coordinator without anyone having to remember.
3. **Trend reporting**, so coordinators see whether complaints and at-risk rates are improving, not just today's numbers.

**How I'd measure it** (targets set after a baseline)

| Metric | What it tells us |
| --- | --- |
| Median time to resolve a complaint | Is the main pain fixed? (North Star) |
| Share of complaints escalated | Are instructors resolving issues, or passing them up? |
| Share of warnings a coordinator rejects | Is the fairness check doing real work? Near 0% may mean a rubber stamp; a high share may mean unclear warning rules. |
| At-risk students flagged before final assessments | Does the warning engine give time to intervene? |
| Time to onboard a student | Did automation remove the manual work? |

## What I learned

Being the user gave me a clear problem quickly, but it also meant I only saw the student's side. Coordinators and instructors carry most of the workflow, and I designed for them from assumptions. On the rebuild, I am starting with their view.
