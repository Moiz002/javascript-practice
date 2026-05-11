# Sequence Diagram: User Registration & Authentication

This document outlines the detailed sequence flows for the User Registration and Authentication module of the Serviqo platform. It covers customer and worker registration, the multi-step worker onboarding process, standard and administrative login procedures, and alternative authentication methods.

## 1. Sequence Diagram (Mermaid)

```mermaid
sequenceDiagram
    autonumber
    participant U as User / Admin
    participant FE as Frontend (React/Next.js)
    participant BE as Backend (Node.js/Express)
    participant DB as Database (MongoDB)
    participant OTP as OTP Service (Demo Console)
    participant G as Google OAuth (External)

    %% Flow A: Customer & Worker Signup + OTP Verification
    rect rgb(240, 248, 255)
    Note over U, OTP: Flow A: Customer & Worker Signup + OTP Verification
    U->>FE: Enter Signup Details (Name, Phone, Password, Role)
    FE->>BE: POST /api/auth/signup
    BE->>DB: Check if Phone exists
    DB-->>BE: No record found
    BE->>BE: Hash Password & Generate 6-digit OTP
    BE->>DB: Create User Record (isVerified: false)
    BE->>OTP: Send OTP via SMS/Email (Console Log)
    BE-->>FE: 201 Created (userId, otp in dev)
    FE->>U: Redirect to /verify
    U->>FE: Enter 6-digit OTP
    FE->>BE: POST /api/auth/verify-otp (userId, otp)
    BE->>DB: Fetch User & Validate OTP
    BE->>DB: Update User (isVerified: true)
    BE->>BE: Generate JWT Session Token
    BE-->>FE: 200 OK (Token, User Info)
    FE->>U: Redirect to Onboarding (Worker) or Dashboard (Customer)
    end

    %% Flow B: Worker Onboarding (5 Steps)
    rect rgb(245, 255, 250)
    Note over U, DB: Flow B: Worker Onboarding (/onboarding) — 5 Steps
    U->>FE: Start Onboarding Process
    Note right of FE: Step 1: Profile (Name, Bio, City, Exp)<br/>Step 2: Skills (Category, Tags, Rate)<br/>Step 3: Portfolio (Project Photos)<br/>Step 4: Identity (CNIC Front/Back)<br/>Step 5: Availability (Schedule, Response Time)
    U->>FE: Complete all 5 steps
    FE->>U: Click "Complete Profile"
    
    par Update Profile & Skills
        FE->>BE: POST /api/worker/profile (FormData: Name, Bio, Avatar)
        FE->>BE: POST /api/worker/skills (Category, Skills, Rate)
    and Upload Assets
        FE->>BE: POST /api/worker/portfolio (Loop: Upload each Image)
        FE->>BE: POST /api/worker/upload-cnic (CNIC Front & Back)
    and Set Availability
        FE->>BE: POST /api/worker/availability (Days, Hours, Response Time)
    end
    
    BE->>DB: Update Worker Document Fields
    BE-->>FE: Success Responses
    FE->>U: Redirect to /worker/dashboard
    end

    %% Flow C: Login (Returning Users)
    rect rgb(255, 250, 240)
    Note over U, DB: Flow C: Login (Returning Users)
    U->>FE: Enter Credentials (Email/Phone, Password)
    FE->>BE: POST /api/auth/login
    BE->>DB: Find User by Identity
    BE->>BE: Compare Bcrypt Hash
    BE->>BE: Generate JWT Session Token
    BE-->>FE: 200 OK (Token, User Profile)
    FE->>U: Redirect to Dashboard (Role-based)
    end

    %% Flow D: Admin Login with TOTP MFA
    rect rgb(255, 240, 245)
    Note over U, DB: Flow D: Admin Login with TOTP MFA
    U->>FE: Enter Admin Email & Password
    FE->>BE: POST /api/auth/admin-login/verify-credentials
    BE->>DB: Validate Admin Password
    BE-->>FE: 200 OK (Credentials Valid)
    FE->>U: Prompt for 6-digit TOTP Code
    U->>FE: Enter TOTP from Authenticator App
    FE->>BE: POST /api/auth/admin-login (ident, pass, totp)
    BE->>BE: Verify TOTP Secret (e.g., 123456)
    BE->>BE: Generate Admin Session Token
    BE-->>FE: 200 OK (Admin Token)
    FE->>U: Redirect to /admin/dashboard
    end

    %% Flow E: OTP-Based Login (Alternative)
    rect rgb(240, 255, 255)
    Note over U, OTP: Flow E: OTP-Based Login (Alternative)
    U->>FE: Enter Phone Number
    FE->>BE: POST /api/auth/request-login-otp
    BE->>DB: Find Active User
    BE->>OTP: Send Login OTP
    BE-->>FE: 200 OK
    U->>FE: Enter OTP Received
    FE->>BE: POST /api/auth/verify-login-otp
    BE->>DB: Validate OTP & Mark Verified
    BE-->>FE: 200 OK (Token, User)
    FE->>U: Redirect to Dashboard
    end

    %% Flow F: Login via Google (Returning Users)
    rect rgb(245, 245, 245)
    Note over U, G: Flow F: Login via Google (Returning Users)
    U->>FE: Click "Continue with Google"
    FE->>G: Redirect to Google Auth Server
    G-->>FE: Redirect with Auth Code
    FE->>BE: POST /api/auth/google (Auth Code)
    BE->>G: Exchange Code for Profile/Token
    BE->>DB: Find or Create User by Google ID
    BE-->>FE: 200 OK (Token, User)
    FE->>U: Redirect to Dashboard
    end
```

## 2. Flow Descriptions

### Flow A: Customer & Worker Signup + OTP Verification
This is the primary entry point for new users. The system captures basic details and ensures phone number ownership through a one-time password (OTP) before allowing access to the platform.

### Flow B: Worker Onboarding
For users registered as workers, a comprehensive 5-step onboarding process is required to build their professional profile. This includes:
1. **Profile:** Basic professional info and avatar.
2. **Skills:** Technical categorization and pricing.
3. **Portfolio:** Visual proof of work.
4. **Identity:** Mandatory CNIC verification (Front & Back).
5. **Availability:** Setting operational hours for the marketplace.

### Flow C: Login (Standard)
Standard authentication using either email or phone number with a password. The backend issues a JWT token stored in an HTTP-only cookie for secure session management.

### Flow D: Admin Login with MFA
To ensure platform security, administrators must undergo a two-step verification process. After providing valid credentials, they must enter a Time-based One-Time Password (TOTP) from an authenticator app.

### Flow E: OTP-Based Login
An alternative, passwordless login method that allows users to access their accounts using only their registered phone number and a dynamic OTP.

### Flow F: Login via Google
OAuth2 integration allowing users to sign in using their Google accounts. This flow simplifies the registration and login process for returning users.
