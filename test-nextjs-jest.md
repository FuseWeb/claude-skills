---
description: Create comprehensive Jest tests for Next.js components, actions, and services
---

# Test Next.js with Jest

When invoked, create comprehensive tests for Next.js applications using Jest, following the project's testing architecture and best practices.

## Testing Strategy

### 1. Identify What to Test

**Server Actions** (`app/actions/`):
- Input validation
- Success and error paths
- Service layer calls
- Return values and status codes
- Subscription limit enforcement
- Authorization checks

**React Components** (`components/` and `app/`):
- Rendering with different props
- User interactions (clicks, form inputs)
- Conditional rendering
- Integration with server actions
- Error boundaries

**Services** (`lib/services/`):
- Business logic
- Repository method calls
- Error handling
- Data transformations

**DO NOT TEST**:
- ShadCN UI components (`components/ui/`)
- Repository methods (database layer)
- External libraries

### 2. Test File Setup

Create test file in `__tests__/` mirroring source structure:

```typescript
// Example: Testing app/actions/clients/post-client.ts
// Test file: __tests__/unit/app/actions/clients/post-client.test.ts

// 1. Declare mock functions at the top level
const mockInsert = jest.fn()
const mockValidateSchema = jest.fn()
const mockIsValidationSuccess = jest.fn()
const mockGetCurrentTeam = jest.fn()
const mockRevalidatePath = jest.fn()

// 2. Mock external dependencies before imports
jest.mock("@/lib/auth/with-auth", () => ({
    withAuth: (action: any) => (...args: any[]) =>
        action({ profile: { user_id: 1 } }, ...args),
}))

jest.mock("@/services/models", () => ({
    ClientService: jest.fn().mockImplementation(() => ({
        insert: mockInsert,
    })),
}))

jest.mock("@/lib/validation/validation-schema", () => ({
    validateSchema: mockValidateSchema,
    isValidationSuccess: mockIsValidationSuccess,
}))

jest.mock("@/lib/helpers/get-team-data-helper", () => ({
    getCurrentTeam: mockGetCurrentTeam,
}))

jest.mock("next/cache", () => ({
    revalidatePath: mockRevalidatePath,
}))

// 3. Import the function under test AFTER mocks
import { postClient } from "@/actions/clients/post-client"
import { mockClient1 } from "@/mocks/data/clients"

describe("postClient", () => {
    let consoleErrorSpy: jest.SpyInstance

    beforeEach(() => {
        jest.clearAllMocks()
        // Mock console.error to keep test output clean
        consoleErrorSpy = jest.spyOn(console, "error").mockImplementation(() => {})
    })

    afterEach(() => {
        consoleErrorSpy.mockRestore()
    })

    it("should successfully create a client", async () => {
        mockValidateSchema.mockReturnValue({ success: true, data: mockData })
        mockIsValidationSuccess.mockReturnValue(true)
        mockInsert.mockResolvedValue(mockClient1)

        const result = await postClient(/* args */)

        expect(mockInsert).toHaveBeenCalledWith(expect.objectContaining({
            company_name: "Test Company",
        }))
        expect(result.code).toBe(201)
    })

    it("should handle validation errors", async () => {
        mockValidateSchema.mockReturnValue({
            code: 422,
            message: "Validation failed"
        })
        mockIsValidationSuccess.mockReturnValue(false)

        const result = await postClient(/* args */)

        expect(result.code).toBe(422)
        expect(mockInsert).not.toHaveBeenCalled()
    })
})
```

### 3. Component Testing Pattern

```typescript
import React from "react"
import { render, screen, fireEvent } from "@testing-library/react"
import "@testing-library/jest-dom"
import MyComponent from "@/components/my-component"

// Mock icons from lucide-react
jest.mock("@/components/ui/icons", () => ({
    PlusCircle: () => <span data-testid="plus-icon">Plus</span>,
    ChevronDown: () => <span data-testid="chevron-icon">Down</span>,
}))

// Mock server actions if component uses them
jest.mock("@/actions/my-action", () => ({
    myAction: jest.fn(),
}))

describe("MyComponent", () => {
    it("renders with correct props", () => {
        render(<MyComponent title="Test" />)
        expect(screen.getByText("Test")).toBeInTheDocument()
    })

    it("handles user interaction", () => {
        const handleClick = jest.fn()
        render(<MyComponent onClick={handleClick} />)

        const button = screen.getByRole("button")
        fireEvent.click(button)

        expect(handleClick).toHaveBeenCalledTimes(1)
    })

    it("shows loading state", () => {
        render(<MyComponent isLoading={true} />)
        expect(screen.getByText("Loading...")).toBeInTheDocument()
    })
})
```

### 4. Test Coverage Requirements

**Server Actions - Test These Cases**:
- ✅ Successful operation (happy path)
- ✅ Validation failures
- ✅ Service/database errors
- ✅ Subscription limit errors (if applicable)
- ✅ Authorization failures (if applicable)
- ✅ Edge cases (empty data, null values)

**Components - Test These Cases**:
- ✅ Renders correctly with different props
- ✅ User interactions (clicks, form inputs)
- ✅ Conditional rendering (loading, error states)
- ✅ Accessibility (ARIA labels, roles)

**Services - Test These Cases**:
- ✅ Business logic correctness
- ✅ Repository method calls with correct params
- ✅ Error handling and transformations
- ✅ Edge cases and validation

### 5. Mock Data Usage

Import mock data from `__tests__/__mocks__/data/`:

```typescript
import { mockClient1 } from "@/mocks/data/clients"
import { mockUser1 } from "@/mocks/data/users"
import { mockTeam1 } from "@/mocks/data/teams"
```

If mock data doesn't exist, create it in the appropriate file:

```typescript
// __tests__/__mocks__/data/clients.ts
export const mockClient1 = {
    id: 1,
    company_name: "Test Company",
    name: "John Doe",
    email: "john@test.com",
    rate: 100,
    color: "#1fa99a",
    enabled: true,
}
```

### 6. Common Mock Patterns

**Next.js specific mocks**:
```typescript
// Mock next/cache
jest.mock("next/cache", () => ({
    revalidatePath: jest.fn(),
    revalidateTag: jest.fn(),
}))

// Mock next/navigation
jest.mock("next/navigation", () => ({
    useRouter: () => ({
        push: jest.fn(),
        replace: jest.fn(),
    }),
    usePathname: () => "/test-path",
}))

// Mock next-intl
jest.mock("next-intl/server", () => ({
    getTranslations: jest.fn().mockResolvedValue((key: string) => key),
}))

// Mock auth wrapper
jest.mock("@/lib/auth/with-auth", () => ({
    withAuth: (action: any) => (...args: any[]) =>
        action({ profile: { user_id: 1 } }, ...args),
}))
```

**Service mocks**:
```typescript
jest.mock("@/services/models", () => ({
    ClientService: jest.fn().mockImplementation(() => ({
        insert: mockInsert,
        getById: mockGetById,
        update: mockUpdate,
    })),
}))
```

### 7. Running Tests

```bash
# Run all tests
pnpm test

# Run specific test file
pnpm test post-client.test

# Run tests in watch mode
pnpm test --watch

# Run with coverage
pnpm test:coverage

# Run only unit tests
pnpm test __tests__/unit/

# Run only integration tests
pnpm test __tests__/integration/
```

### 8. Best Practices Checklist

- ✅ Test file mirrors source file location in `__tests__/`
- ✅ Mock functions declared at top level
- ✅ All external dependencies mocked
- ✅ Mock imports before actual imports
- ✅ Use `beforeEach` to clear mocks
- ✅ Mock `console.error` to keep output clean
- ✅ Use descriptive test names: "should do X when Y"
- ✅ Test both success and error paths
- ✅ Verify function calls with correct arguments
- ✅ Check return values and status codes
- ✅ Don't test implementation details
- ✅ Don't test external libraries or UI components
- ✅ Don't test repositories (database layer)

### 9. Subscription Limit Testing

For actions that enforce subscription limits:

```typescript
it("should handle subscription limit error", async () => {
    const limitError = new SubscriptionLimitError(
        "You have reached your client limit (5/5).",
        "max_clients" as any,
        5,
        5,
        "Solo"
    )
    mockInsert.mockRejectedValue(limitError)

    const result = await postClient(/* args */)

    expect(result.code).toBe(500)
    expect(result.message).toContain("client limit")
})

it("should succeed when under subscription limit", async () => {
    mockInsert.mockResolvedValue(mockClient1)

    const result = await postClient(/* args */)

    expect(mockInsert).toHaveBeenCalled()
    expect(result.code).toBe(201)
})
```

### 10. Integration Testing

Use sparingly for critical user flows:

```typescript
// __tests__/integration/auth-flow.test.ts
describe("User Authentication Flow", () => {
    it("should complete full signup and login flow", async () => {
        // Test multiple actions working together
        const signupResult = await signup(/* args */)
        expect(signupResult.code).toBe(201)

        const loginResult = await signin(/* args */)
        expect(loginResult.code).toBe(200)
    })
})
```

## Quick Reference

**Test Structure**:
1. Declare mocks at top level
2. Mock dependencies before imports
3. Import function/component under test
4. Setup `beforeEach` and `afterEach`
5. Write test cases covering all scenarios

**Common Assertions**:
- `expect(result.code).toBe(201)`
- `expect(mockFn).toHaveBeenCalled()`
- `expect(mockFn).toHaveBeenCalledWith(expectedArgs)`
- `expect(screen.getByText("text")).toBeInTheDocument()`
- `expect(mockFn).not.toHaveBeenCalled()`

**Files to Reference**:
- `.junie/standards/testing.md` - Testing standards
- `__tests__/unit/app/actions/clients/post-client.test.ts` - Action example
- `__tests__/unit/components/time/add-time-entry.test.tsx` - Component example
- `jest.config.ts` - Jest configuration
