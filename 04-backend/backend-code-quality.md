# Backend Code Quality and File Boundaries

## File-size rule

1. **MUST NOT** allow a production Java/Kotlin source file to exceed 800 physical lines, excluding generated code.
2. **MUST** split a file before it becomes hard to understand; 800 lines is a hard upper limit, not a target.
3. **MUST** split by responsibility, not by arbitrary line ranges.
4. **MUST** keep related split files in the same module/package and use explicit types/interfaces so their relationship is easy to find.
5. **MUST NOT** create vague `Helper`, `Util`, `Manager`, `CommonService`, or `Part1` files merely to pass the limit.

## Good split examples

```text
booking/application/
  CreateBookingUseCase.java
  CancelBookingUseCase.java
  BookingAvailabilityService.java
  BookingPolicyValidator.java

booking/api/
  BookingController.java
  BookingRequestMapper.java
  BookingResponseMapper.java
```

```text
identity_access/application/
  LoginService.java
  RefreshSessionService.java
  PasswordResetService.java
  EmailVerificationService.java
```

The calling relationship is clear through constructor injection/interfaces, package placement, and tests. For example `CreateBookingUseCase` depends on `BookingAvailabilityService` and `BookingPolicyValidator`; it does not become a 1,200-line service.

## Additional maintainability rules

1. **MUST** use constructor injection; field injection is prohibited.
2. **MUST** keep controller/request/response DTOs separate from JPA entities.
3. **MUST NOT** expose JPA entities directly through API responses.
4. **MUST** keep one public top-level type per source file, except tightly coupled small package-private types where the language/project convention permits it.
5. **SHOULD** keep methods focused; extract a named method/service when a method has multiple independent responsibilities or deeply nested branches.
6. **MUST** enforce formatting and static analysis in the build/CI pipeline; complexity/file-size checks should warn early and fail at the hard limit.
