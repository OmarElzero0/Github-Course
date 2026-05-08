# Job Search Website - Technical Audit Report

## 1. Executive Summary
- Overall project quality: Low. The UI is present, but most backend functionality is not wired or secure.
- Architecture quality: Weak. Django/DRF are scaffolded but the UI uses localStorage as the primary data store.
- Main strengths: Basic data models, clean template structure, and usable UI layouts.
- Main weaknesses: No real authentication/authorization, open APIs, missing server-side CRUD, inconsistent routing, and no tests.
- Readiness level: Not production-ready. Significant rework required.

## 2. Requirement Coverage Analysis
| Requirement | Status | Evidence | Problems | Recommendation |
| --- | --- | --- | --- | --- |
| Company admin sign up (username, password, confirm, email, is_company_admin, company name required) | Partially Implemented | [mainApp/templates/HTMLpages/signup.html](mainApp/templates/HTMLpages/signup.html#L26), [mainApp/static/JS/signup-script.js](mainApp/static/JS/signup-script.js#L48-L78), [mainApp/models.py](mainApp/models.py#L5-L12) | No server-side registration, roles stored in localStorage, company name requirement not enforced in model | Use Django auth forms, store roles in DB, enforce company name for admins |
| Company admin login | Partially Implemented | [mainApp/templates/HTMLpages/login.html](mainApp/templates/HTMLpages/login.html#L25), [mainApp/static/JS/LoginScript.js](mainApp/static/JS/LoginScript.js#L12-L22) | Client-only auth, no session handling, no backend validation | Use Django auth and sessions |
| Add new job opportunity | Partially Implemented | [mainApp/templates/HTMLpages/add-job.html](mainApp/templates/HTMLpages/add-job.html#L24), [mainApp/static/JS/addjob-script.js](mainApp/static/JS/addjob-script.js#L46-L79) | Client-only storage, no server-side validation, no role checks | Create server-side job creation with permissions |
| Job fields (id, title, salary, company, status, description, years exp, created by admin) | Partially Implemented | [mainApp/models.py](mainApp/models.py#L38-L57), [mainApp/templates/HTMLpages/add-job.html](mainApp/templates/HTMLpages/add-job.html#L31-L127) | Creator not enforced, salary is free text, job ownership not validated | Enforce creator from request.user, validate salary and status |
| View jobs created by admin company | Partially Implemented | [mainApp/static/JS/admin-dashboard.js](mainApp/static/JS/admin-dashboard.js#L22-L31) | Filtering is client-only and can be bypassed | Filter at the database level by creator/company |
| Edit job details | Partially Implemented | [mainApp/templates/HTMLpages/edit-job.html](mainApp/templates/HTMLpages/edit-job.html#L24), [mainApp/static/JS/editjob-script.js](mainApp/static/JS/editjob-script.js#L33-L73) | No server-side edit endpoint or auth | Add backend update with object-level permissions |
| Delete jobs | Partially Implemented | [mainApp/static/JS/admin-dashboard.js](mainApp/static/JS/admin-dashboard.js#L93-L113) | Client-only deletion, no server validation | Add backend delete with object-level permissions |
| Regular user sign up (username, password, confirm, email, is_admin) | Incorrectly Implemented | [mainApp/templates/HTMLpages/signup.html](mainApp/templates/HTMLpages/signup.html#L26), [mainApp/static/JS/signup-script.js](mainApp/static/JS/signup-script.js#L48-L78) | No is_admin field, no backend, role is implied by checkbox | Use proper role field in DB and server-side validation |
| Regular user login | Partially Implemented | [mainApp/templates/HTMLpages/login.html](mainApp/templates/HTMLpages/login.html#L25), [mainApp/static/JS/LoginScript.js](mainApp/static/JS/LoginScript.js#L12-L22) | Client-only auth, no sessions | Use Django auth and sessions |
| Search for jobs by title and years of experience | Partially Implemented | [mainApp/static/JS/browsescript.js](mainApp/static/JS/browsescript.js#L6-L52) | Client-only filtering, no server-side search | Implement DB search with filters |
| View available jobs | Partially Implemented | [mainApp/templates/HTMLpages/browse.html](mainApp/templates/HTMLpages/browse.html#L1-L61), [mainApp/static/JS/browsescript.js](mainApp/static/JS/browsescript.js#L6-L28) | Client-only jobs from localStorage | Serve jobs from DB |
| View job details page | Partially Implemented | [mainApp/templates/HTMLpages/job-details.html](mainApp/templates/HTMLpages/job-details.html#L1-L101), [mainApp/static/JS/jobdetails.js](mainApp/static/JS/jobdetails.js#L3-L44) | Client-only data, no server-side fetch | Render from DB or fetch API |
| Apply for jobs | Partially Implemented | [mainApp/static/JS/jobdetails.js](mainApp/static/JS/jobdetails.js#L90-L109) | Application stored in localStorage, no server persistence | Create Application records on the server |
| View list of applied jobs | Incorrectly Implemented | [mainApp/templates/HTMLpages/applied-jobs.html](mainApp/templates/HTMLpages/applied-jobs.html#L1-L75), [mainApp/static/JS/applied-jobs.js](mainApp/static/JS/applied-jobs.js#L3-L47) | Uses backend API that is not connected to login or application creation | Unify data source and use authenticated API |
| Navigation bar on all pages | Partially Implemented | [mainApp/templates/HTMLpages/home.html](mainApp/templates/HTMLpages/home.html#L10-L21), [mainApp/templates/HTMLpages/browse.html](mainApp/templates/HTMLpages/browse.html#L10-L19) | Separate static nav per page, no shared template | Use a base template and include nav |
| Navigation bar changes based on login | Missing | [mainApp/templates/HTMLpages/home.html](mainApp/templates/HTMLpages/home.html#L10-L21) | No template logic for login state | Use Django template context or JS + real auth |

## 3. Architecture and Code Quality Review
**Good practices**
- Data models for Profile, Job, and Application exist and match many domain concepts. See [mainApp/models.py](mainApp/models.py#L5-L77).
- UI is consistent and uses shared CSS. See [mainApp/static/css/StyleSheet.css](mainApp/static/css/StyleSheet.css).

**Anti-patterns and risks**
- Two separate sources of truth (localStorage vs database) with no synchronization.
- Server-side views mostly render static templates without data binding or validation.
- DRF exists but is not integrated with the UI and has no authentication.
- Repeated navbar markup in every template instead of base template inheritance.
- Job ownership and role constraints are not enforced in models or views.

## 4. REST API and Integration Review
- **Not RESTful in practice**: DRF viewsets are exposed but not used by the UI, while the UI uses localStorage for CRUD. See [mainApp/api/views.py](mainApp/api/views.py#L5-L18) and [mainApp/static/JS/addjob-script.js](mainApp/static/JS/addjob-script.js#L46-L79).
- **Security gap**: No authentication or permissions on viewsets. Any user can read or modify all jobs and applications. See [mainApp/api/views.py](mainApp/api/views.py#L5-L18).
- **Endpoint inconsistency**: /api/applications/ is defined in two places. The custom endpoint in [mainApp/urls.py](mainApp/urls.py#L6) shadows the router in [jobsearch/urls.py](jobsearch/urls.py#L22-L23).
- **Overuse of AJAX**: The UI is static HTML and could be simpler with server-rendered templates. If a real SPA is desired, then all data should come from a secured API.

**Recommended approach**
- Choose one architecture: server-rendered Django views with forms, or a real API + SPA.
- If using API: add authentication (session or token), permissions, pagination, and consistent endpoints.
- If server-rendered: remove localStorage usage, use Django forms, and render data from the DB.

## 5. Security Review
- **Authentication**: Client-only auth with plaintext passwords is not acceptable. See [mainApp/static/JS/signup-script.js](mainApp/static/JS/signup-script.js#L48-L78).
- **Authorization**: No object-level permissions. Any user can edit or delete any job if using the API. See [mainApp/api/views.py](mainApp/api/views.py#L5-L18).
- **CSRF protection**: HTML forms do not include csrf_token. See [mainApp/templates/HTMLpages/login.html](mainApp/templates/HTMLpages/login.html#L25).
- **XSS risks**: User-provided strings are injected via innerHTML. See [mainApp/static/JS/jobdetails.js](mainApp/static/JS/jobdetails.js#L9-L30).
- **Sensitive data exposure**: Secret key is committed and DEBUG is enabled. See [jobsearch/settings.py](jobsearch/settings.py#L23-L26).

## 6. Performance Review
- Client-side filtering is fine for small data sets but will not scale beyond a few hundred jobs.
- No pagination or query optimization on API endpoints.
- The applications endpoint performs multiple queries without optimization. See [mainApp/views.py](mainApp/views.py#L10-L25).

## 7. UI and UX Review
- Navigation is visually consistent but not dynamic; it does not reflect actual login state.
- Forms do not provide server-side validation feedback.
- Several links point to raw HTML files, which will break in production routing. See [mainApp/templates/HTMLpages/home.html](mainApp/templates/HTMLpages/home.html#L28-L144).

## 8. File-by-File Issues List

### Issue #1 - Unauthenticated REST API viewsets
Severity: Critical

File:
[mainApp/api/views.py](mainApp/api/views.py#L5-L18)

Lines:
[mainApp/api/views.py](mainApp/api/views.py#L5-L18)

Problem:
All Profile, Job, and Application endpoints are exposed with no authentication or authorization controls.

Why This Matters:
Any user can read or modify all data, including applications and admin-created jobs.

Recommended Fix:
Add authentication and permissions, then filter querysets by owner.

Example:
```python
from rest_framework.permissions import IsAuthenticated

class JobViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Job.objects.filter(creator=self.request.user)

    def perform_create(self, serializer):
        serializer.save(creator=self.request.user)
```

### Issue #2 - Client-side auth with plaintext passwords
Severity: Critical

File:
[mainApp/static/JS/signup-script.js](mainApp/static/JS/signup-script.js#L48-L78)

Lines:
[mainApp/static/JS/signup-script.js](mainApp/static/JS/signup-script.js#L48-L78)

Problem:
Passwords and users are stored in localStorage and handled entirely on the client.

Why This Matters:
Any script or user can read and modify credentials. There is no real authentication.

Recommended Fix:
Use Django auth for registration and login. Store only session identifiers in cookies.

### Issue #3 - Login validates against localStorage
Severity: Critical

File:
[mainApp/static/JS/LoginScript.js](mainApp/static/JS/LoginScript.js#L12-L17)

Lines:
[mainApp/static/JS/LoginScript.js](mainApp/static/JS/LoginScript.js#L12-L17)

Problem:
Login checks a plaintext password from localStorage, not the server.

Why This Matters:
Anyone can bypass or tamper with authentication and impersonate users.

Recommended Fix:
Authenticate via Django and use session or token-based auth.

### Issue #4 - Client-only job and application persistence
Severity: High

File:
[mainApp/static/JS/addjob-script.js](mainApp/static/JS/addjob-script.js#L46-L79), [mainApp/static/JS/editjob-script.js](mainApp/static/JS/editjob-script.js#L14-L73), [mainApp/static/JS/jobdetails.js](mainApp/static/JS/jobdetails.js#L106-L108)

Lines:
[mainApp/static/JS/addjob-script.js](mainApp/static/JS/addjob-script.js#L46-L79), [mainApp/static/JS/editjob-script.js](mainApp/static/JS/editjob-script.js#L14-L73), [mainApp/static/JS/jobdetails.js](mainApp/static/JS/jobdetails.js#L106-L108)

Problem:
Jobs and applications are created and updated only in localStorage.

Why This Matters:
Data is not durable, can be manipulated, and is not shared between users.

Recommended Fix:
Move CRUD to server endpoints and store data in the database.

### Issue #5 - XSS risk due to innerHTML
Severity: High

File:
[mainApp/static/JS/admin-dashboard.js](mainApp/static/JS/admin-dashboard.js#L49-L73), [mainApp/static/JS/browsescript.js](mainApp/static/JS/browsescript.js#L10-L28), [mainApp/static/JS/jobdetails.js](mainApp/static/JS/jobdetails.js#L9-L30), [mainApp/static/JS/applied-jobs.js](mainApp/static/JS/applied-jobs.js#L27-L44)

Lines:
[mainApp/static/JS/admin-dashboard.js](mainApp/static/JS/admin-dashboard.js#L49-L73), [mainApp/static/JS/browsescript.js](mainApp/static/JS/browsescript.js#L10-L28), [mainApp/static/JS/jobdetails.js](mainApp/static/JS/jobdetails.js#L9-L30), [mainApp/static/JS/applied-jobs.js](mainApp/static/JS/applied-jobs.js#L27-L44)

Problem:
User-supplied job data is injected directly with innerHTML.

Why This Matters:
Stored XSS is possible when job titles or descriptions include HTML.

Recommended Fix:
Use textContent or sanitize input before rendering.

### Issue #6 - Applied jobs flow is broken
Severity: High

File:
[mainApp/static/JS/applied-jobs.js](mainApp/static/JS/applied-jobs.js#L3-L28), [mainApp/views.py](mainApp/views.py#L10-L36), [mainApp/static/JS/jobdetails.js](mainApp/static/JS/jobdetails.js#L106-L109)

Lines:
[mainApp/static/JS/applied-jobs.js](mainApp/static/JS/applied-jobs.js#L3-L28), [mainApp/views.py](mainApp/views.py#L10-L36), [mainApp/static/JS/jobdetails.js](mainApp/static/JS/jobdetails.js#L106-L109)

Problem:
Applications are stored in localStorage but the applied jobs page fetches from a backend API that requires Django auth.

Why This Matters:
Users do not see their applications and the API returns 401.

Recommended Fix:
Unify the data flow and use authenticated server endpoints for both create and list.

### Issue #7 - Job details link uses application id
Severity: Medium

File:
[mainApp/static/JS/applied-jobs.js](mainApp/static/JS/applied-jobs.js#L47)

Lines:
[mainApp/static/JS/applied-jobs.js](mainApp/static/JS/applied-jobs.js#L47)

Problem:
The job details link uses app.id instead of the job id.

Why This Matters:
Links will resolve to the wrong page or 404.

Recommended Fix:
Include job id in the API response and use it for the URL.

### Issue #8 - Missing REST framework dependency and invalid Django version
Severity: High

File:
[requirements.txt](requirements.txt#L1-L4), [jobsearch/settings.py](jobsearch/settings.py#L42)

Lines:
[requirements.txt](requirements.txt#L1-L4), [jobsearch/settings.py](jobsearch/settings.py#L42)

Problem:
The project uses rest_framework in settings but the dependency is not in requirements, and Django 6.0.4 is likely invalid.

Why This Matters:
Installations will fail and the API will not run.

Recommended Fix:
Add djangorestframework and use a supported Django release.

### Issue #9 - Hardcoded secret key and DEBUG enabled
Severity: High

File:
[jobsearch/settings.py](jobsearch/settings.py#L23-L26)

Lines:
[jobsearch/settings.py](jobsearch/settings.py#L23-L26)

Problem:
Secret key is committed and DEBUG is True.

Why This Matters:
Exposes sensitive settings and weakens security posture.

Recommended Fix:
Load SECRET_KEY from environment and disable DEBUG in production.

### Issue #10 - ALLOWED_HOSTS is empty
Severity: Low

File:
[jobsearch/settings.py](jobsearch/settings.py#L28)

Lines:
[jobsearch/settings.py](jobsearch/settings.py#L28)

Problem:
ALLOWED_HOSTS is empty.

Why This Matters:
Production deployments will fail or be misconfigured.

Recommended Fix:
Set ALLOWED_HOSTS via environment variables.

### Issue #11 - Dashboard URL is referenced but not routed
Severity: Medium

File:
[mainApp/urls.py](mainApp/urls.py#L14), [mainApp/templates/HTMLpages/dashboard.html](mainApp/templates/HTMLpages/dashboard.html#L17)

Lines:
[mainApp/urls.py](mainApp/urls.py#L14), [mainApp/templates/HTMLpages/dashboard.html](mainApp/templates/HTMLpages/dashboard.html#L17)

Problem:
The dashboard route is commented out, but the template references it.

Why This Matters:
Template rendering will raise a reverse error or link to a missing route.

Recommended Fix:
Add a dashboard view and route, or update the template to a valid path.

### Issue #12 - Forms are not wired to Django and lack CSRF
Severity: Medium

File:
[mainApp/templates/HTMLpages/login.html](mainApp/templates/HTMLpages/login.html#L25), [mainApp/templates/HTMLpages/signup.html](mainApp/templates/HTMLpages/signup.html#L26), [mainApp/templates/HTMLpages/add-job.html](mainApp/templates/HTMLpages/add-job.html#L24), [mainApp/templates/HTMLpages/edit-job.html](mainApp/templates/HTMLpages/edit-job.html#L24), [mainApp/templates/HTMLpages/job-details.html](mainApp/templates/HTMLpages/job-details.html#L77)

Lines:
[mainApp/templates/HTMLpages/login.html](mainApp/templates/HTMLpages/login.html#L25), [mainApp/templates/HTMLpages/signup.html](mainApp/templates/HTMLpages/signup.html#L26), [mainApp/templates/HTMLpages/add-job.html](mainApp/templates/HTMLpages/add-job.html#L24), [mainApp/templates/HTMLpages/edit-job.html](mainApp/templates/HTMLpages/edit-job.html#L24), [mainApp/templates/HTMLpages/job-details.html](mainApp/templates/HTMLpages/job-details.html#L77)

Problem:
Forms post to # or static HTML pages and do not include csrf_token.

Why This Matters:
Forms will not work with Django and will be vulnerable if wired to the backend.

Recommended Fix:
Use Django form views and include csrf_token in templates.

### Issue #13 - Static asset and routing links bypass Django
Severity: Medium

File:
[mainApp/templates/HTMLpages/home.html](mainApp/templates/HTMLpages/home.html#L28-L144), [mainApp/templates/HTMLpages/browse.html](mainApp/templates/HTMLpages/browse.html#L61), [mainApp/templates/HTMLpages/guestBrowse.html](mainApp/templates/HTMLpages/guestBrowse.html#L63), [mainApp/templates/HTMLpages/add-job.html](mainApp/templates/HTMLpages/add-job.html#L139), [mainApp/templates/HTMLpages/edit-job.html](mainApp/templates/HTMLpages/edit-job.html#L139), [mainApp/templates/HTMLpages/job-details.html](mainApp/templates/HTMLpages/job-details.html#L74-L101)

Lines:
[mainApp/templates/HTMLpages/home.html](mainApp/templates/HTMLpages/home.html#L28-L144), [mainApp/templates/HTMLpages/browse.html](mainApp/templates/HTMLpages/browse.html#L61), [mainApp/templates/HTMLpages/guestBrowse.html](mainApp/templates/HTMLpages/guestBrowse.html#L63), [mainApp/templates/HTMLpages/add-job.html](mainApp/templates/HTMLpages/add-job.html#L139), [mainApp/templates/HTMLpages/edit-job.html](mainApp/templates/HTMLpages/edit-job.html#L139), [mainApp/templates/HTMLpages/job-details.html](mainApp/templates/HTMLpages/job-details.html#L74-L101)

Problem:
Templates use raw HTML paths and relative static paths rather than Django url and static tags.

Why This Matters:
Links break when deployed and static files may not be served.

Recommended Fix:
Replace hardcoded links with Django url and static tags.

### Issue #14 - Company name not enforced for company admins
Severity: Medium

File:
[mainApp/models.py](mainApp/models.py#L11-L12)

Lines:
[mainApp/models.py](mainApp/models.py#L11-L12)

Problem:
company_name is optional for all profiles, but should be required for company admins.

Why This Matters:
Admin accounts can exist without company context, breaking job ownership rules.

Recommended Fix:
Validate in model clean(), forms, or serializer validation.

### Issue #15 - Creator and applications fields are user-editable
Severity: Medium

File:
[mainApp/templates/HTMLpages/add-job.html](mainApp/templates/HTMLpages/add-job.html#L54-L55), [mainApp/templates/HTMLpages/add-job.html](mainApp/templates/HTMLpages/add-job.html#L126-L127), [mainApp/templates/HTMLpages/edit-job.html](mainApp/templates/HTMLpages/edit-job.html#L54-L55), [mainApp/templates/HTMLpages/edit-job.html](mainApp/templates/HTMLpages/edit-job.html#L126-L127)

Lines:
[mainApp/templates/HTMLpages/add-job.html](mainApp/templates/HTMLpages/add-job.html#L54-L55), [mainApp/templates/HTMLpages/add-job.html](mainApp/templates/HTMLpages/add-job.html#L126-L127), [mainApp/templates/HTMLpages/edit-job.html](mainApp/templates/HTMLpages/edit-job.html#L54-L55), [mainApp/templates/HTMLpages/edit-job.html](mainApp/templates/HTMLpages/edit-job.html#L126-L127)

Problem:
The form asks users to enter application count and creator name.

Why This Matters:
These should be derived from the system, not user input.

Recommended Fix:
Remove these fields from the form and set values on the server.

### Issue #16 - API route conflict for /api/applications/
Severity: Low

File:
[jobsearch/urls.py](jobsearch/urls.py#L22-L23), [mainApp/urls.py](mainApp/urls.py#L6)

Lines:
[jobsearch/urls.py](jobsearch/urls.py#L22-L23), [mainApp/urls.py](mainApp/urls.py#L6)

Problem:
A custom applications endpoint exists in mainApp.urls while the DRF router also registers applications.

Why This Matters:
Routing is ambiguous and harder to maintain.

Recommended Fix:
Pick one endpoint strategy and remove the duplicate.

### Issue #17 - Missing automated tests
Severity: Low

File:
[mainApp/tests.py](mainApp/tests.py#L1-L2)

Lines:
[mainApp/tests.py](mainApp/tests.py#L1-L2)

Problem:
No tests exist for authentication, job CRUD, or applications.

Why This Matters:
Regressions are likely and refactoring is risky.

Recommended Fix:
Add unit tests for models and integration tests for API and views.

### Issue #18 - Job details redirects to invalid path
Severity: Low

File:
[mainApp/static/JS/jobdetails.js](mainApp/static/JS/jobdetails.js#L109)

Lines:
[mainApp/static/JS/jobdetails.js](mainApp/static/JS/jobdetails.js#L109)

Problem:
Redirects to ../pages/applied-jobs.html which does not exist in the Django routing.

Why This Matters:
Users will land on a 404 after applying.

Recommended Fix:
Use the Django route for applied jobs.

### Issue #19 - View name shadows Django login
Severity: Low

File:
[mainApp/views.py](mainApp/views.py#L50)

Lines:
[mainApp/views.py](mainApp/views.py#L50)

Problem:
The view function is named login, which conflicts with django.contrib.auth.login.

Why This Matters:
It can cause confusion or future import conflicts.

Recommended Fix:
Rename the view to login_view or auth_login.

## 9. Best Practices Compliance
- REST API best practices: Not compliant (no auth, no pagination, no validation).
- Clean code principles: Partially compliant (basic structure, but duplication and mixed responsibilities).
- SOLID principles: Mostly not applicable at this scale, but separation of concerns is weak.
- MVT architecture: Partially compliant (templates exist, but views do not use models).
- DRY principle: Not compliant in templates (repeated navbar).
- Security best practices: Not compliant (plaintext passwords, no CSRF, open API).
- Frontend best practices: Partially compliant (basic layout, but client-side auth and XSS risk).

## 10. Refactoring Recommendations
**Immediate fixes**
- Implement real authentication with Django and remove localStorage credentials.
- Lock down API endpoints with permissions and object-level filtering.
- Replace client-side CRUD with server-side views or API endpoints.

**Important improvements**
- Consolidate routing and use Django url and static tags.
- Add model validation for company admins and job ownership.
- Create a base template for the navbar and common layout.

**Nice-to-have improvements**
- Add pagination and search endpoints.
- Improve input validation and error messages.
- Expand test coverage across models, views, and API.

## 11. Final Verdict
- Implementation correctness: Not correct for production use.
- Architecture choices: Inconsistent and underengineered on the backend, over-reliant on localStorage.
- API/AJAX usage: Not justified in its current form; either use full API or full server-rendered views.
- Engineering practices: Below expectations due to security gaps and missing validation.
- Biggest risks: Data integrity, unauthorized access, and broken user flows.
- Overall engineering score: 3/10
