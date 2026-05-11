# Sequence Diagram 1: User Registration & Authentication
 
## Participants
 
| Symbol | Component | Technology |
|---|---|---|
| U | User (Actor) | Human |
| FE | Frontend | Next.js / React.js |
| MW | Auth Middleware | Express.js + JWT |
| BE | Backend API | Node.js / Express |
| DB | Database | MongoDB |
| OTP | OTP Service | Console / SMS Gateway |
| G | Google Auth Server | Google OAuth 2.0 |
 
---
 
## Flows Covered
 
- **Flow A** — Customer & Worker Signup + OTP Verification  
- **Flow B** — Worker Onboarding (5 Steps)  
- **Flow C** — Password-Based Login (Returning Users)  
- **Flow D** — Admin Login with TOTP MFA (Two-Step)  
- **Flow E** — OTP-Based Login (Passwordless)  
- **Flow F** — Login via Google OAuth 2.0 *(planned — UI implemented)*
---
 
```mermaid
sequenceDiagram
    autonumber
 
    participant U  as User
    participant FE as Frontend<br/>(Next.js)
    participant BE as Backend API<br/>(Express)
    participant DB as MongoDB
    participant OTP as OTP Service
 
    %% ══════════════════════════════════════════════════════════════
    %% FLOW A — Customer / Worker Signup + OTP Verification
    %% ══════════════════════════════════════════════════════════════
 
    rect rgb(230, 240, 255)
        Note over U,OTP: ── FLOW A : Customer & Worker Signup + OTP Verification ──
 
        U->>FE: Fill registration form<br/>(name, phone, email?, password, role)
 
        FE->>BE: POST /auth/check-availability { phone, email }
        activate BE
        BE->>DB: User.exists({ phone }) · User.exists({ email })
        DB-->>BE: { phoneExists, emailExists }
        BE-->>FE: 200 { phoneAvailable, emailAvailable }
        deactivate BE
 
        alt phone or email already taken
            FE->>U: Inline validation error — field highlighted
        else fields available
            FE->>BE: POST /auth/signup { name, phone, email, password, role }
            activate BE
            BE->>DB: findOne({ phone })
            DB-->>BE: null
            BE->>BE: bcrypt.hash(password, 10)
            BE->>BE: generateOtp() — 6-digit random<br/>otpExpiry = now + 10 min
            BE->>DB: User.create({ ...fields,<br/>isVerified: false, otp, otpExpiry })
            DB-->>BE: saved user { _id }
            BE->>OTP: Dispatch OTP to phone<br/>(SMS / console in dev)
            BE-->>FE: 201 { userId, message }
            deactivate BE
 
            FE->>U: Navigate → OTP verification screen
 
            loop Until OTP verified or max-attempts reached
                U->>FE: Enter 6-digit OTP
 
                opt User requests resend
                    FE->>BE: POST /auth/resend-otp { userId }
                    activate BE
                    BE->>DB: findById(userId)
                    DB-->>BE: user
                    BE->>BE: generateOtp() + new otpExpiry
                    BE->>DB: user.save({ otp, otpExpiry })
                    BE->>OTP: Resend OTP to phone
                    BE-->>FE: 200 { message: "OTP resent" }
                    deactivate BE
                    FE->>U: Countdown timer reset
                end
 
                FE->>BE: POST /auth/verify-otp { userId, otp }
                activate BE
                BE->>DB: findById(userId)
                DB-->>BE: user
 
                alt OTP matches & not expired
                    BE->>DB: user.save({ isVerified: true,<br/>otp: null, otpExpiry: null })
                    BE->>BE: createToken(user) — JWT signed<br/>{ _id, phone, role } · exp 7d
                    BE->>BE: setAuthCookies(token, serviqo_session)
                    BE-->>FE: 200 { user } + Set-Cookie: token · serviqo_session
                    deactivate BE
                    FE->>U: Redirect → customer/dashboard<br/>or /onboarding (if worker)
                else OTP invalid
                    BE-->>FE: 400 { error: "Invalid OTP" }
                    deactivate BE
                    FE->>U: Show error — retry allowed
                else OTP expired
                    BE-->>FE: 400 { error: "OTP expired" }
                    deactivate BE
                    FE->>U: Show expiry notice — request resend
                end
            end
        end
    end
 
    %% ══════════════════════════════════════════════════════════════
    %% FLOW B — Worker Onboarding (5 Steps)
    %% ══════════════════════════════════════════════════════════════
 
    rect rgb(230, 255, 235)
        Note over U,OTP: ── FLOW B : Worker Onboarding — 5 Steps ──
        Note over FE,BE: Precondition: worker authenticated (JWT cookie present)
 
        FE->>BE: GET /auth/me
        activate BE
        BE->>DB: findById(req.user._id)
        DB-->>BE: user profile
        BE-->>FE: { user }
        deactivate BE
        FE->>U: Render onboarding wizard — Step 1 of 5
 
        Note over U,FE: Step 1 — Profile Details
        U->>FE: Enter display name, bio, city,<br/>area, years experience · upload avatar
        FE->>FE: Validate Step 1 fields (name, city required)
        FE->>BE: PUT /worker/profile<br/>multipart/form-data { name, bio, city,<br/>address, experience, profilePicture }
        activate BE
        BE->>DB: findByIdAndUpdate(userId, profileData, { new:true })
        DB-->>BE: updated user
        BE-->>FE: 200 { message, user }
        deactivate BE
        FE->>U: Advance → Step 2
 
        Note over U,FE: Step 2 — Skills & Rates
        U->>FE: Select primary category,<br/>add skill tags, set hourly rate
        FE->>FE: Validate (category + ≥1 skill required)
        FE->>BE: PUT /worker/skills<br/>{ skills[], serviceCategory, hourlyRate }
        activate BE
        BE->>DB: findByIdAndUpdate(userId, skillData)
        DB-->>BE: updated user
        BE-->>FE: 200 { message, user }
        deactivate BE
        FE->>U: Advance → Step 3
 
        Note over U,FE: Step 3 — Portfolio Upload
        U->>FE: Upload work photos + captions
        loop For each portfolio image
            FE->>BE: POST /worker/portfolio<br/>multipart { image, title/caption }
            activate BE
            BE->>DB: $push portfolioItem<br/>{ title, image: cloudinaryURL }
            DB-->>BE: updated portfolio
            BE-->>FE: 201 { message, item }
            deactivate BE
        end
        FE->>U: Advance → Step 4
 
        Note over U,FE: Step 4 — CNIC Verification
        U->>FE: Upload CNIC front + back images
        FE->>FE: Validate both files present
        FE->>BE: POST /worker/cnic<br/>multipart { cnicFront, cnicBack }
        activate BE
        BE->>DB: save({ cnicFront: URL,<br/>cnicBack: URL, cnicVerified: false })
        DB-->>BE: updated user
        BE-->>FE: 200 { cnicFront, cnicBack,<br/>cnicVerified: false }
        deactivate BE
        Note over BE,DB: Admin independently reviews<br/>& approves/rejects CNIC
        FE->>U: Advance → Step 5
 
        Note over U,FE: Step 5 — Availability Settings
        U->>FE: Toggle working days,<br/>set time slots + response time
        FE->>FE: Validate (≥1 day + time range required)
        FE->>BE: PUT /worker/availability<br/>{ availability{}, responseTime }
        activate BE
        BE->>DB: findByIdAndUpdate(userId,<br/>{ availability, responseTime })
        DB-->>BE: updated user
        BE-->>FE: 200 { availability, responseTime }
        deactivate BE
 
        FE->>U: Redirect → /worker/dashboard?welcome=true
    end
 
    %% ══════════════════════════════════════════════════════════════
    %% FLOW C — Password-Based Login (Returning Users)
    %% ══════════════════════════════════════════════════════════════
 
    rect rgb(255, 245, 220)
        Note over U,OTP: ── FLOW C : Password-Based Login (Returning Users) ──
 
        U->>FE: Enter phone/email + password · select portal
        FE->>BE: POST /auth/login { identifier, password }
        activate BE
        BE->>BE: Block if identifier == admin email<br/>→ redirect to admin portal
 
        BE->>DB: findOne({ phone|email })
        DB-->>BE: user || null
 
        alt User not found
            BE-->>FE: 401 { error: "Invalid credentials" }
            deactivate BE
            FE->>U: Show generic error
        else User role is admin
            BE-->>FE: 403 { error: "Use admin portal" }
            deactivate BE
            FE->>U: Redirect notice → /admin/login
        else User found
            BE->>BE: bcrypt.compare(password, user.password)
            activate BE
            alt Password mismatch
                BE-->>FE: 401 { error: "Invalid credentials" }
                deactivate BE
                FE->>U: Show invalid credentials error
            else Account not verified
                BE-->>FE: 403 { error: "Not verified", userId }
                deactivate BE
                FE->>U: Prompt OTP verification
            else Account suspended
                BE-->>FE: 403 { error: "Account suspended" }
                deactivate BE
                FE->>U: Show suspension notice
            else All checks pass
                BE->>BE: createToken(user) — JWT 7d
                BE->>BE: setAuthCookies(token, serviqo_session)
                BE-->>FE: 200 { user } + Set-Cookie
                deactivate BE
                FE->>U: Redirect → role-based dashboard
            end
        end
    end
 
    %% ══════════════════════════════════════════════════════════════
    %% FLOW D — Admin Login with TOTP MFA (Two-Step)
    %% ══════════════════════════════════════════════════════════════
 
    rect rgb(255, 230, 255)
        Note over U,OTP: ── FLOW D : Admin Login with TOTP MFA ──
 
        Note over U,FE: Step 1 — Credential Pre-Validation
        U->>FE: Enter admin email + password
        FE->>BE: POST /auth/admin-login/verify-credentials<br/>{ identifier, password }
        activate BE
        BE->>BE: resolveAdminCredentials(identifier, password)
        BE->>DB: findOne({ email|phone, role:'admin' })
        DB-->>BE: admin user || null
 
        alt Invalid credentials
            BE-->>FE: 401 { error: "Invalid admin credentials" }
            deactivate BE
            FE->>U: Show error — stay on Step 1
        else Credentials valid
            BE-->>FE: 200 { message: "Proceed to verification" }
            deactivate BE
            FE->>U: Reveal TOTP input field
        end
 
        Note over U,FE: Step 2 — TOTP Submission
        U->>FE: Enter 6-digit TOTP code
        FE->>BE: POST /auth/admin-login<br/>{ identifier, password, totp }
        activate BE
        BE->>BE: Validate TOTP code
 
        alt TOTP invalid
            BE-->>FE: 401 { error: "Invalid verification code" }
            deactivate BE
            FE->>U: Show TOTP error — allow retry
        else TOTP valid
            BE->>BE: resolveAdminCredentials(identifier, password)
            BE->>BE: createToken(adminUser) — JWT 7d
            BE->>BE: setAuthCookies(token, serviqo_session)
            BE-->>FE: 200 { user: adminUser } + Set-Cookie
            deactivate BE
            FE->>U: Redirect → /admin/dashboard
        end
    end
 
    %% ══════════════════════════════════════════════════════════════
    %% FLOW E — OTP-Based Login (Passwordless)
    %% ══════════════════════════════════════════════════════════════
 
    rect rgb(235, 255, 245)
        Note over U,OTP: ── FLOW E : OTP-Based Login (Passwordless) ──
 
        U->>FE: Enter phone number + select role
        FE->>BE: POST /auth/request-login-otp { phone, role }
        activate BE
        BE->>DB: findOne({ phone, role })
        DB-->>BE: user || null
 
        alt User not registered for this role
            BE-->>FE: 404 { error: "Not registered for role" }
            deactivate BE
            FE->>U: Show registration prompt
        else Account suspended
            BE-->>FE: 403 { error: "Account suspended" }
            deactivate BE
            FE->>U: Show suspension notice
        else User found & active
            BE->>BE: generateOtp() + otpExpiry (now + 10 min)
            BE->>DB: user.save({ otp, otpExpiry })
            BE->>OTP: Send OTP to phone
            BE-->>FE: 200 { userId, message }
            deactivate BE
 
            FE->>U: Navigate → OTP input screen
 
            U->>FE: Enter 6-digit OTP
            FE->>BE: POST /auth/verify-login-otp { userId, otp }
            activate BE
            BE->>DB: findById(userId)
            DB-->>BE: user
 
            alt OTP invalid
                BE-->>FE: 400 { error: "Invalid OTP" }
                deactivate BE
                FE->>U: Show error — allow retry
            else OTP expired
                BE-->>FE: 400 { error: "OTP expired" }
                deactivate BE
                FE->>U: Show expiry — request new OTP
            else OTP valid
                BE->>DB: user.save({ otp: null,<br/>otpExpiry: null, isVerified: true })
                BE->>BE: createToken(user) — JWT 7d
                BE->>BE: setAuthCookies(token, serviqo_session)
                BE-->>FE: 200 { user } + Set-Cookie
                deactivate BE
                FE->>U: Redirect → role-based dashboard
            end
        end
    end
 
    %% ══════════════════════════════════════════════════════════════
    %% FLOW F — Login via Google OAuth 2.0 (Planned / UI Present)
    %% ══════════════════════════════════════════════════════════════
 
    rect rgb(255, 240, 230)
        Note over U,OTP: ── FLOW F : Login via Google OAuth 2.0 (Planned) ──
 
        participant G as Google<br/>Auth Server
 
        U->>FE: Click "Continue with Google"
        FE->>G: Redirect → Google OAuth consent screen<br/>scope: profile, email
        activate G
        G->>U: Display Google account selector
        U->>G: Select account + grant consent
        G-->>FE: Authorization Code + redirect URI callback
        deactivate G
 
        FE->>BE: POST /auth/google<br/>{ authorizationCode }
        activate BE
        BE->>G: Exchange code → { access_token, id_token }
        activate G
        G-->>BE: { access_token, id_token }
        deactivate G
 
        BE->>BE: Verify id_token — decode { email, name, sub }
        BE->>DB: findOne({ email })
        DB-->>BE: user || null
 
        alt Existing user found
            BE->>BE: createToken(user) — JWT 7d
            BE->>BE: setAuthCookies(token, serviqo_session)
            BE-->>FE: 200 { user } + Set-Cookie
            deactivate BE
            FE->>U: Redirect → role-based dashboard
        else New user — auto-register
            BE->>DB: User.create({ name, email,<br/>role:'customer', isVerified:true,<br/>googleId: sub })
            DB-->>BE: new user { _id }
            BE->>BE: createToken(newUser) — JWT 7d
            BE->>BE: setAuthCookies(token, serviqo_session)
            BE-->>FE: 201 { user } + Set-Cookie
            deactivate BE
            FE->>U: Redirect → /customer/dashboard
        end
    end
```
 
---
 
## Diagram Notes
 
**Flow A** — Signup triggers a `check-availability` preflight to prevent duplicate phone/email entries before the actual `POST /auth/signup`. OTP is valid for 10 minutes; resend is rate-limited. On successful `verify-otp`, the backend sets two HTTP-only cookies: `token` (JWT) and `serviqo_session` (role:userId string).
 
**Flow B** — Worker onboarding is a sequential 5-step wizard executed fully client-side with state managed via `useReducer`. Each step invokes a separate API endpoint. Portfolio items are uploaded one-by-one. CNIC verification sets `cnicVerified: false` pending asynchronous admin approval.
 
**Flow C** — The `handleLogin` controller explicitly blocks admin credentials from the customer/worker login route, forcing admins to use the dedicated `/admin/login` portal. Account suspension and unverified-account states return distinct `403` error codes.
 
**Flow D** — Admin MFA uses a two-phase flow: credential pre-validation first (`/admin-login/verify-credentials`), then TOTP submission (`/admin-login`). This prevents timing-based credential probing. Current implementation uses a hardcoded TOTP for demonstration; production will integrate a proper TOTP library.
 
**Flow E** — The OTP login route additionally calls `findOne({ phone, role })` to prevent cross-role authentication (e.g., a phone registered as `customer` cannot use OTP login on the `worker` portal). Successfully verifying login OTP also sets `isVerified: true` if not already set.
 
**Flow F** — Google OAuth 2.0 Authorization Code flow. The backend exchanges the authorization code for tokens with Google, decodes the `id_token` to extract the verified user identity, and either authenticates the existing account or auto-creates a new customer account with `isVerified: true`. This flow is present in the UI mockups and is planned for backend implementation in FYP Phase 2.
