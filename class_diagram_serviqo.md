# Software Design Specification: Class Diagram

The following UML class diagram illustrates the structural architecture of the Serviqo platform, including user roles, core marketplace entities, communication modules, and AI agents.

```mermaid
classDiagram
    direction TB

    %% --- User Management Module ---
    namespace UserManagement {
        class User {
            +ObjectId id
            +String name
            +String phone
            +String email
            +String role
            +String profilePicture
            +String city
            +String address
            +Boolean isVerified
            +Boolean isActive
            +String otp
            +login(identifier, password)
            +verifyOtp(otp)
            +resetPassword(newPassword)
        }

        class Customer {
            +createJob(details)
            +getBids(jobId)
            +acceptBid(bidId)
            +markJobComplete(jobId)
        }

        class Worker {
            +String serviceCategory
            +String[] skills
            +String bio
            +Number experience
            +Number hourlyRate
            +Number rating
            +Number completedJobs
            +Boolean cnicVerified
            +Object availability
            +submitBid(jobId, amount, proposal)
            +updateProfile(bio, skills)
            +managePortfolio()
        }

        class Admin {
            +verifyWorker(workerId)
            +handleAdminLogin(totp)
            +resolveDispute(disputeId)
            +monitorAnalytics()
        }
    }

    %% --- Marketplace Module ---
    namespace Marketplace {
        class Job {
            +ObjectId id
            +String title
            +String description
            +String category
            +String budgetRange
            +String urgency
            +Object location
            +String status
            +String[] images
            +Number totalBids
            +getJobById()
            +updateJob(updates)
            +deleteJob()
        }

        class Bid {
            +ObjectId id
            +Number amount
            +String proposal
            +Number estimatedDays
            +String status
            +submit()
            +accept()
            +reject()
        }
    }

    %% --- Communication & Feedback ---
    namespace Communication {
        class Conversation {
            +ObjectId id
            +String lastMessage
            +DateTime lastMessageAt
            +getOrCreate(participantId)
            +getMessages()
        }

        class Message {
            +ObjectId id
            +String text
            +Boolean isRead
            +DateTime createdAt
            +sendMessage(text)
            +markAsRead()
        }

        class Review {
            +ObjectId id
            +Number rating
            +String comment
            +submitReview()
            +getWorkerReviews()
        }

        class Dispute {
            +ObjectId id
            +String reason
            +String description
            +String status
            +String resolution
            +raiseDispute()
            +updateResolution(decision)
        }
    }

    %% --- AI Agent Module ---
    namespace AIAgents {
        class RecruiterAgent {
            +recommendWorkers(jobId)
            +rankProfiles(skills, ratings)
        }

        class PricingAdvisorAgent {
            +predictPrice(jobDetails)
            +analyzeMarketTrends()
        }
    }

    %% --- Relationships ---

    %% Specialization
    User <|-- Customer : Inheritance
    User <|-- Worker : Inheritance
    User <|-- Admin : Inheritance

    %% Marketplace Relationships
    Customer "1" --> "*" Job : "posts"
    Worker "1" --> "*" Bid : "submits"
    Job "1" -- "*" Bid : "contains"
    Job "1" -- "0..1" Worker : "assigned to"
    Job "1" -- "0..1" Bid : "accepted bid"

    %% Communication Relationships
    Conversation "1" -- "*" Message : "contains"
    Conversation "*" -- "1" Job : "discusses"
    User "2" -- "*" Conversation : "participates"
    
    %% Feedback & Dispute
    Review "*" -- "1" Job : "feedback for"
    Review "*" -- "1" Customer : "written by"
    Review "*" -- "1" Worker : "rates"
    
    Dispute "*" -- "1" Job : "concerning"
    Dispute "*" -- "1" User : "raised by"
    Admin "1" -- "*" Dispute : "resolves"

    %% AI Agent Interactions
    RecruiterAgent ..> Worker : "analyzes"
    RecruiterAgent ..> Job : "suggests for"
    PricingAdvisorAgent ..> Job : "guides"
```

### Diagram Description
*   **User Management**: Implements role-based access control (RBAC). The `User` class acts as the base entity for `Customer`, `Worker`, and `Admin` roles.
*   **Marketplace**: Manages the core lifecycle of a `Job` and the competitive `Bid` system. A job transitions through statuses from `open` to `completed`.
*   **Communication**: Facilitates real-time interaction via `Conversation` and `Message` entities, linked directly to specific jobs to maintain context.
*   **AI Agents**: Specialized agentic modules that provide intelligent assistance:
    *   **RecruiterAgent**: Automates worker-job matching.
    *   **PricingAdvisorAgent**: Provides data-driven budget guidance.
*   **Relationships**: 
    *   **Inheritance** is used for user roles.
    *   **Association** represents structural links (e.g., Customer posting a Job).
    *   **Dependency** (dashed arrows) indicates temporary usage or analysis by AI services.
