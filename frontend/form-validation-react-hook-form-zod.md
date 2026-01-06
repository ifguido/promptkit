# Advanced Form Validation - React Hook Form + Zod

## Objective
Build a comprehensive form validation system using React Hook Form and Zod schema validation, with complex validation rules, multi-step forms, and server-side validation integration.

## Technology Stack
- React 18+ with TypeScript
- react-hook-form v7+
- Zod for schema validation
- @hookform/resolvers for Zod integration
- Optional: react-hook-form-devtools for debugging

## Requirements

### 1. Core Features

**Schema-Based Validation:**
- Define validation schemas with Zod
- Type-safe form data
- Reusable validation schemas
- Complex nested object validation
- Array field validation
- Conditional validation rules

**Form Handling:**
- Real-time validation feedback
- Submit handling with loading states
- Server-side error integration
- Form reset functionality
- Dirty field tracking
- Touch field tracking

**UX Enhancements:**
- Show errors only after field blur or submit
- Inline error messages
- Summary of errors
- Success feedback
- Auto-focus first error field
- Accessible error announcements

### 2. Example Forms to Implement

**User Registration Form:**
```typescript
// Fields: email, password, confirmPassword, firstName, lastName, dateOfBirth, terms
// Validations: email format, password strength, matching passwords, age verification
```

**Multi-Step Checkout Form:**
```typescript
// Step 1: Shipping address
// Step 2: Payment details
// Step 3: Order review
// Preserve data between steps, validate each step independently
```

**Dynamic Form with Arrays:**
```typescript
// Add/remove form fields dynamically
// Example: Multiple phone numbers, work experience entries, etc.
```

**File Upload Form:**
```typescript
// File type validation
// File size limits
// Multiple file uploads
// Preview uploaded files
```

### 3. Zod Schemas

**Basic Validation Schema:**
```typescript
const registrationSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain uppercase letter')
    .regex(/[a-z]/, 'Password must contain lowercase letter')
    .regex(/[0-9]/, 'Password must contain number')
    .regex(/[^A-Za-z0-9]/, 'Password must contain special character'),
  confirmPassword: z.string(),
  firstName: z.string().min(2, 'First name required'),
  lastName: z.string().min(2, 'Last name required'),
  dateOfBirth: z.date().refine(
    (date) => {
      const age = differenceInYears(new Date(), date);
      return age >= 18;
    },
    { message: 'Must be 18 years or older' }
  ),
  terms: z.boolean().refine((val) => val === true, {
    message: 'You must accept terms and conditions',
  }),
}).refine((data) => data.password === data.confirmPassword, {
  message: "Passwords don't match",
  path: ['confirmPassword'],
});
```

**Nested Object Schema:**
```typescript
const addressSchema = z.object({
  street: z.string().min(1),
  city: z.string().min(1),
  state: z.string().length(2),
  zipCode: z.string().regex(/^\d{5}$/),
  country: z.string(),
});

const profileSchema = z.object({
  user: z.object({
    name: z.string(),
    email: z.string().email(),
  }),
  address: addressSchema,
  contacts: z.array(
    z.object({
      type: z.enum(['phone', 'email']),
      value: z.string(),
    })
  ).min(1),
});
```

### 4. Components

**FormField Component:**
```typescript
interface FormFieldProps {
  name: string;
  label: string;
  type?: 'text' | 'email' | 'password' | 'number' | 'date';
  placeholder?: string;
  helperText?: string;
  required?: boolean;
}
```

**FormSelect Component:**
```typescript
interface FormSelectProps {
  name: string;
  label: string;
  options: { value: string; label: string }[];
  placeholder?: string;
}
```

**FormCheckbox Component:**
**FormRadioGroup Component:**
**FormFileUpload Component:**
**FormArrayField Component:** (for dynamic fields)
**FormErrorSummary Component:**

### 5. Hooks

**useFormWithSchema:**
```typescript
function useFormWithSchema<T extends z.ZodType>(schema: T) {
  return useForm<z.infer<T>>({
    resolver: zodResolver(schema),
    mode: 'onBlur',
  });
}
```

**useMultiStepForm:**
```typescript
const {
  currentStep,
  nextStep,
  prevStep,
  goToStep,
  isFirstStep,
  isLastStep,
  formData,
  updateFormData,
} = useMultiStepForm(steps);
```

**useServerValidation:**
```typescript
const { setServerErrors } = useServerValidation(formMethods);
// Integrate backend validation errors into form
```

### 6. File Structure
```
src/
├── components/
│   ├── forms/
│   │   ├── FormField.tsx
│   │   ├── FormSelect.tsx
│   │   ├── FormCheckbox.tsx
│   │   ├── FormRadioGroup.tsx
│   │   ├── FormFileUpload.tsx
│   │   ├── FormArrayField.tsx
│   │   ├── FormErrorSummary.tsx
│   │   └── MultiStepForm.tsx
│   └── ui/
│       ├── Input.tsx
│       ├── Label.tsx
│       └── ErrorMessage.tsx
├── hooks/
│   ├── useFormWithSchema.ts
│   ├── useMultiStepForm.ts
│   └── useServerValidation.ts
├── schemas/
│   ├── auth.schemas.ts
│   ├── profile.schemas.ts
│   └── checkout.schemas.ts
├── utils/
│   ├── formHelpers.ts
│   └── validationMessages.ts
└── examples/
    ├── RegistrationForm.tsx
    ├── MultiStepCheckout.tsx
    └── DynamicForm.tsx
```

### 7. Advanced Validation Patterns

**Conditional Validation:**
```typescript
const schema = z.object({
  accountType: z.enum(['personal', 'business']),
  businessName: z.string().optional(),
  taxId: z.string().optional(),
}).refine(
  (data) => {
    if (data.accountType === 'business') {
      return !!data.businessName && !!data.taxId;
    }
    return true;
  },
  {
    message: 'Business name and tax ID required for business accounts',
    path: ['businessName'],
  }
);
```

**Async Validation (username availability):**
```typescript
const usernameSchema = z.string().refine(
  async (username) => {
    const available = await checkUsernameAvailability(username);
    return available;
  },
  { message: 'Username already taken' }
);
```

**Cross-Field Validation:**
```typescript
const dateRangeSchema = z.object({
  startDate: z.date(),
  endDate: z.date(),
}).refine((data) => data.endDate > data.startDate, {
  message: 'End date must be after start date',
  path: ['endDate'],
});
```

### 8. Features

**Form State Indicators:**
- isDirty: Form has been modified
- isValid: All validations pass
- isSubmitting: Form is being submitted
- isSubmitted: Form has been submitted
- touchedFields: Which fields user interacted with
- dirtyFields: Which fields have changed

**Performance:**
- Debounced validation for expensive checks
- Lazy validation (only validate on blur/submit)
- Memo heavy components
- Virtualize long forms

**Developer Experience:**
- TypeScript inference from Zod schemas
- React Hook Form DevTools integration
- Custom validation error messages
- Reusable schema composition

### 9. Server Integration

**Handle Server Errors:**
```typescript
try {
  await submitForm(data);
} catch (error) {
  if (error.validationErrors) {
    // Map server errors to form fields
    Object.entries(error.validationErrors).forEach(([field, message]) => {
      setError(field, { message });
    });
  }
}
```

**Optimistic Updates:**
```typescript
// Show success before server confirms
// Roll back on error
```

## Deliverables
- Complete form component library
- Zod validation schemas for common use cases
- Multi-step form implementation
- File upload with validation
- Dynamic array fields
- Server error integration
- Comprehensive TypeScript types
- Storybook examples
- Testing utilities

## Testing Checklist
- [ ] Required field validation works
- [ ] Email format validation works
- [ ] Password strength validation works
- [ ] Matching password validation works
- [ ] Custom validation rules work
- [ ] Nested object validation works
- [ ] Array field validation works
- [ ] Conditional validation works
- [ ] Async validation works (username check)
- [ ] Server errors display in form
- [ ] Multi-step form preserves data
- [ ] File upload validation works
- [ ] Form reset clears all fields
- [ ] Error messages are accessible
- [ ] Submit button disables during submission
- [ ] Form data is type-safe
