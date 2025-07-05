# A DefinitiveSpec User Guide & Onboarding: The PrimeCart App (v4.1)

**Welcome to the DefinitiveSpec Guide for PrimeCart!**

This guide introduces you to the Definitive Development Methodology (DDM) and the DSAC v4.1 Autonomous Agent. It is your comprehensive manual for building the PrimeCart application, where clarity, verifiability, and partnership with AI are the cornerstones of our process.

This guide is structured to teach you **how to think** about the DDM process and **what to write** in your `.dspec` files, using concrete examples from the PrimeCart e-commerce platform.

**Target Audience:** Developers, QAs, Product Owners, and Architects.

---
## Chapter 1: The "Why" of DDM: Core Principles & Benefits

DDM is built on a few powerful principles designed to solve chronic software development challenges like ambiguity, spec-code drift, and the gap between business goals and technical implementation.

*   **Verifiable Certainty is Paramount:** The agent's—and our—primary directive. All actions, analysis, and code generation must be grounded in facts from `.dspec` files. The agent will refuse to invent information and will pause to demand clarity when faced with ambiguity.
    *   *Benefit:* Eliminates guesswork. Every decision is traceable to a specific, written requirement or design choice. This builds trust and reduces errors.

*   **AI as a Strategic Partner, Not Just an Implementer:** The v4.1 agent is stateful; it maintains an in-memory `Project Index` of all specs. This allows it to act as a true partner, analyzing the impact of changes, exploring ideas strategically, and detecting architectural-level issues.
    *   *Benefit:* We leverage AI for high-value strategic thinking, not just for automating boilerplate. This leads to better designs and faster, more informed decisions.

*   **The Specification is the Single Source of Truth:** All behavior, structure, and constraints are defined in `.dspec` files. The code is a *product* of the spec.
    *   *Benefit:* Documentation is never out of date because it *is* the blueprint. Onboarding is faster and maintenance is simpler.

*   **Interactive Refinement & The Feedback Loop:** The development process is a conversation. When the agent detects a problem or an opportunity, it doesn't just halt; it `[PAUSE]`s and provides a `[RECOMMENDATION]`. This makes the feedback loop between human intent and system implementation explicit and collaborative.
    *   *Benefit:* Problems are caught and discussed early. The system evolves safely, with human oversight at critical junctures.

---
## Chapter 2: The DDM Ecosystem: Your Tools

DDM is supported by a conceptual toolset, whose functions are largely embodied by the DSAC v4.1 agent itself.

*   **The AI Agent (DSAC):** The core engine. It operates on the stateful **DDM Operational Lifecycle**. It runs all analysis, generation, and verification, communicating via its `[Fact]/[Assumption]/[Uncertainty]` protocol.
*   **The Project Index:** The agent's in-memory model of the entire project. This is what enables powerful commands like `Analyze Impact of Change`. It's built from all the `.dspec` files you provide.
*   **The Architectural Profile:** A special, project-specific `.dspec` file containing `directive` artifacts. It serves as the "cookbook" for your project's specific technology stack, safely extending the agent's core knowledge.

---

## Chapter 3: The Building Blocks: Core DSpec Artifacts

Your primary job as an Operator is to express intent through these artifacts. Let's use the PrimeCart User Registration feature as our running example.

### 3.1 `requirement`: Capturing Intent

This is the starting point, defining the "what" and "why" for a feature.

```definitive_spec
// From: users.dspec

requirement UserRegistration {
    title: "New User Account Registration";
    description: `
        **Goal:** Enable new visitors to create a PrimeCart account.
        **User Story:** As a new visitor to PrimeCart, I want to be able to register for an account using my email and a secure password, so that I can make purchases and track my orders.
    `;
    priority: "High";
    status: "Accepted";
    acceptance_criteria: [
        "Given I am on the registration page",
        "When I enter a unique email 'newuser@example.com'",
        "And I enter a strong password 'ValidPass123!'",
        "Then my account should be created successfully",
        "And I should receive an email confirming my registration."
    ];
    source: "Product Roadmap Q1 - User Features";
}
```

### 3.2 `model`: Defining Your Data

This defines the shape, types, and validation rules for your data. The agent uses this to generate interfaces, DTOs, and validation logic.

```definitive_spec
// From: users.dspec

model UserRegistrationPayload {
    description: "Data payload for new user registration. Constraints are enforced by the API gateway/framework.";
    email: String {
        description: "User's email address. Must be unique system-wide.";
        format: "email";
        required: true;
        maxLength: 255;
        pii_category: "ContactInfo"; // Triggers security analysis by the DataFlowSecurityAnalyzer module.
    }
    password: String {
        description: "User's desired password. Must meet strength requirements.";
        minLength: 10;
        pattern: "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d).{10,}$";
        required: true;
    }
}

model UserEntity {
    description: "Internal representation of a user in the data store (e.g., a database table).";
    user_id: String { format: "uuid"; required: true; }
    email: String { format: "email"; required: true; pii_category: "ContactInfo"; }
    hashed_password: String { required: true; }
    created_at: DateTime { required: true; }
}
```

### 3.3 `api`: Specifying Service Contracts

This defines the public contract for an API endpoint.

```definitive_spec
// From: users.dspec

api RegisterUser {
    title: "Register New User";
    part_of: designs.UserService; // Links to a design component
    path: "/v1/users/register";
    method: "POST";
    request_model: models.UserRegistrationPayload;
    response_model: models.UserProfileResponse; // Assume this model is defined elsewhere
    errors: [ policies.ErrorCatalog.ValidationFailed, policies.ErrorCatalog.EmailAlreadyInUse ];
}
```

### 3.4 `code`: Detailing Business Logic

This is the most critical spec for implementation. `detailed_behavior` is the verifiable source of truth for the agent.

```definitive_spec
// From: users.dspec

code HandleUserRegistration {
    title: "Core Logic for User Registration Process";
    implements_api: apis.RegisterUser;
    language: "TypeScript";
    implementation_location: {
        filepath: "primecart-app/src/modules/users/handlers/registration_handler.ts",
        entry_point_name: "handleRegistration"
    };
    signature: "async function(payload: models.UserRegistrationPayload): Promise<models.UserProfileResponse>";
    preconditions: [ "Input `payload` has passed schema validation." ];
    postconditions: [ "If successful, a new user record is created in the Data Store." ];

    detailed_behavior: `
        // AI Agent Target: Translate this abstract logic into TypeScript.
        // Human Review Focus: Correctness of this business logic sequence.

        // 1. Check Email Uniqueness (Abstract Data Operation)
        DECLARE existingUser AS OPTIONAL models.UserEntity;
        CALL Abstract.UserDataStore.CheckByEmail WITH { email: payload.email } RETURNING existingUser;

        IF existingUser IS_PRESENT THEN
            RETURN_ERROR policies.ErrorCatalog.EmailAlreadyInUse;
        END_IF;

        // 2. Prepare User Entity (Abstract Service Call)
        DECLARE hashedPassword AS String;
        CALL Abstract.PasswordHasher.Hash WITH { password: payload.password } RETURNING hashedPassword;

        DECLARE newUserEntity AS models.UserEntity;
        CREATE_INSTANCE models.UserEntity WITH {
            email: payload.email,
            hashed_password: hashedPassword,
            created_at: System.CurrentUTCDateTime // Another abstract call
        } ASSIGN_TO newUserEntity;

        // 3. Persist User
        PERSIST newUserEntity TO Abstract.UserDataStore;

        // 4. Construct and Return Success Response
        DECLARE response AS models.UserProfileResponse;
        CREATE_INSTANCE models.UserProfileResponse FROM newUserEntity ASSIGN_TO response;
        RETURN_SUCCESS response;
    `;
    throws_errors: [
        policies.ErrorCatalog.EmailAlreadyInUse,
        policies.ErrorCatalog.InternalServerError
    ];
    dependencies: [ "Abstract.UserDataStore", "Abstract.PasswordHasher", "Abstract.SystemDateTimeProvider" ];
}
```

### 3.5 `behavior` and `interaction`: Modeling Complex Flows

While `code` handles logic inside one component, `behavior` and `interaction` model how multiple components work together. Use a `behavior { fsm ... }` for the stateful lifecycle of a single entity (like an order) and an `interaction` for the sequenced conversation between services.

```definitive_spec
// From: checkout.dspec

behavior CheckoutProcess {
    title: "Manages the state transitions of the order checkout process.";

    fsm MainCheckoutFSM {
        description: "Models the customer's journey through the PrimeCart checkout process.";
        initial: CartNotEmpty;

        state CartNotEmpty { description: "User has items in cart and initiates checkout."; }
        state ShippingAddressProvided { description: "User has provided or confirmed shipping address."; }
        state PaymentProcessing { description: "Payment is being processed with the gateway."; }
        state OrderConfirmed { description: "Payment successful, order placed."; }

        transition { from: CartNotEmpty, event: "ProceedToShipping", to: ShippingAddressProvided; }
        transition { from: ShippingAddressProvided, event: "ConfirmAndPay", to: PaymentProcessing; }
        transition { from: PaymentProcessing, event: "PaymentGatewaySuccess", to: OrderConfirmed; }
    }
}
```

---

## Chapter 4: The DDM Lifecycle & Workflow

This is how you, the Operator, collaborate with the DSAC v4.1 agent. It is a continuous, interactive cycle.

**The Agent's Internal 5-Phase Lifecycle:**
1.  **Phase 0: Project Sync:** Agent loads all specs and builds its `Project Index`.
2.  **Phase 1: Focus & Review:** Agent receives your command and identifies the relevant specs.
3.  **Phase 2: Pre-Generation Analysis:** Agent runs guardrail checks (`SpecFirstEnforcer`, `NPlusOneDetector`, etc.). **This is where most `[PAUSE]` interactions occur.**
4.  **Phase 3: Core Task Execution:** Agent performs the main command.
5.  **Phase 4: Post-Generation Verification:** Agent runs final self-checks with `IntegrityVerifier`.
6.  **Phase 5: System Refinement:** Agent's learning modules are invoked via the `Distill Pattern from...` command.

### The Workflow in Action: Building the PrimeCart Registration

1.  **Explore an Idea (Phase 3):**
    *   **You say:** `DSpec: Explore Idea "A loyalty points system for PrimeCart."`
    *   **Agent acts:** Generates three `requirement` specs: a simple "Earn Points" MVP, a "Redeem Points" Core feature, and a "Tiered Status" Ambitious vision.
    *   **You act:** Your team discusses and chooses a strategic direction.

2.  **Generate & Refine the Blueprint (Phase 3):**
    *   **You say:** `DSpec: Generate from Requirement requirement.users.UserRegistration`
    *   **Agent acts:** Scaffolds all necessary draft specs, including the `model.users.UserRegistrationPayload`, `api.users.RegisterUser`, and `code.users.HandleUserRegistration` shown above.
    *   **You act:** Review and refine the `detailed_behavior` to perfect the business logic.

3.  **Implement with Certainty (Phases 2 & 3):**
    *   **You say:** `DSpec: Implement Code Spec code.users.HandleUserRegistration`
    *   **Agent acts (Step 1):** The agent's internal modules run. For example, the `DataFlowSecurityAnalyzer` verifies that the `pii_category: "ContactInfo"` on the `email` field is handled correctly according to security policies. Let's assume it passes.
    *   **Agent acts (Step 2):** Now that analysis has passed, it generates a `test` spec for your approval. `[PAUSE] Please review and ACCEPT the proposed test cases for successful registration and duplicate email handling.`
    *   **You act:** `ACCEPT`.
    *   **Agent acts (Step 3):** Generates the final TypeScript code for `handleRegistration` in `registration_handler.ts`, guaranteed to be covered by the approved tests.

4.  **Evolve the System (Phase 5):**
    *   **You notice:** The logic for checking email uniqueness is used in registration, password reset, and account updates.
    *   **You say:** `DSpec: Distill Pattern from code.users.HandleUserRegistration` (pointing to the uniqueness check logic).
    *   **Agent acts:** Initiates a dialogue to create a new `pattern CheckEmailUniqueness(...)` and adds it to the project's `Architectural Profile`.
    *   **You act:** Refactor the other `code` specs to use this new, clean, reusable abstraction.

---
## Appendix A: Common Pitfalls & Best Practices

*   **Pitfall: Ambiguous Prompts.**
    *   **Problem:** Vague commands like "fix the checkout" will be rejected by the v4.1 agent.
    *   **Avoid By:** Be specific. Use the agent's commands. Reference artifacts by their full `QualifiedName` (e.g., `code.users.HandleUserRegistration`). Your goal is to reduce uncertainty.
*   **Pitfall: Ignoring a `[PAUSE]`.**
    *   **Problem:** The agent pauses for a reason—it has found a verifiable problem. Overriding it or ignoring the recommendation leads to technical debt or bugs.
    *   **Avoid By:** Treat every `[PAUSE]` as a critical integration point. Read the `[Fact]` and `[RECOMMENDATION]` carefully. Engage in the dialogue.
*   **Pitfall: Code-in-Spec.**
    *   **Problem:** Writing TypeScript in `detailed_behavior` instead of abstract keywords.
    *   **Avoid By:** Keep `detailed_behavior` focused on *business logic flow*. Use `CALL`, `PERSIST`, etc. Let the `Architectural Profile`'s `pattern`s handle the implementation details.
*   **Pitfall: Blind Trust.**
    *   **Problem:** Accepting AI-generated specs or analysis without review.
    *   **Avoid By:** The agent is a partner that provides verifiable, fact-based output. Your job is to be the human expert who reviews those facts and makes the final judgment. ALWAYS review generated artifacts.
*   **Pitfall: Logic in Directives.**
    *   **Problem:** Putting complex business logic (`IF/ELSE`) inside a `directive` template.
    *   **Avoid By:** Business rules belong in `code.detailed_behavior`. Directives are for *translating* abstract steps into technology-specific implementation, not for defining the steps themselves.

---
## Appendix B: Thinking with the AI: Effective Interaction

*   **Be a Verifier:** Your primary role is to ensure the specs you write are clear, complete, and correct. The better your input, the better the agent's output.
*   **Use the Full Command Set:** Don't just use `Implement Code Spec`. Leverage the strategic commands (`Explore Idea`, `Analyze Impact`) to make better decisions *earlier* in the process.
*   **Embrace the `[PAUSE]`:** A pause isn't a failure; it's the system working correctly. It's an opportunity to catch an error before it becomes code.
*   **Drive Evolution:** Use `Distill Pattern from...` actively. When you see repeated logic, make it a priority to teach the agent how to abstract it. This is how you collectively improve the entire system's quality and maintainability.
*   **When in Doubt, Ask for Analysis:** If you're unsure about a spec's quality or impact, ask the agent to analyze it. Use `Analyze Spec Quality...` or `Generate Diagram...` to build your own understanding.