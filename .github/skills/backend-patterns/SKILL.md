---
name: backend-patterns
description: NestJS and TypeScript patterns for backend-developer. Ensures consistency, testability, and clean separation of concerns across all backend code.
---

# Skill: Backend Patterns

## Purpose

NestJS + TypeScript patterns for the `backend-developer`. These ensure consistency, testability, and clean separation of concerns across all backend code.

---

## Module Structure

Every feature is a NestJS module. Modules are self-contained and expose only what other modules need.

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([UserEntity])],
  controllers: [UserController],
  providers: [
    CreateUserUseCase,
    GetUserUseCase,
    { provide: "IUserRepository", useClass: TypeOrmUserRepository },
  ],
  exports: [GetUserUseCase], // only export what other modules need
})
export class UserModule {}
```

---

## Controller Pattern

Controllers are thin adapters — parse input, call a use case, return output.

```typescript
@Controller("users")
export class UserController {
  constructor(private readonly createUser: CreateUserUseCase) {}

  @Post()
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    const user = await this.createUser.execute({
      email: dto.email,
      name: dto.name,
    });
    return UserResponseDto.fromDomain(user);
  }
}
```

Rules:

- No business logic in controllers
- Always use DTOs for input and output — never expose domain objects directly
- Handle exceptions at the controller level with exception filters

---

## Use Case Pattern

One use case per user action. Use cases orchestrate domain logic.

```typescript
@Injectable()
export class CreateUserUseCase {
  constructor(
    @Inject("IUserRepository")
    private readonly userRepository: IUserRepository,
  ) {}

  async execute(input: CreateUserInput): Promise<User> {
    const existing = await this.userRepository.findByEmail(input.email);
    if (existing) throw new UserAlreadyExistsError(input.email);

    const user = User.create({ email: input.email, name: input.name });
    await this.userRepository.save(user);
    return user;
  }
}
```

Rules:

- No HTTP concerns in use cases (no `Request`, `Response`, status codes)
- Input and output types are plain domain objects or simple DTOs
- One `execute()` method per use case

---

## Repository Pattern

Repositories are interfaces in the domain/application layer, implemented in infrastructure.

```typescript
// Application layer — interface (port)
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

// Infrastructure layer — implementation (adapter)
@Injectable()
export class TypeOrmUserRepository implements IUserRepository {
  constructor(
    @InjectRepository(UserEntity)
    private readonly repo: Repository<UserEntity>,
  ) {}

  async findById(id: string): Promise<User | null> {
    const entity = await this.repo.findOne({ where: { id } });
    return entity ? UserMapper.toDomain(entity) : null;
  }
  // ...
}
```

---

## DTO Validation

All input DTOs must use `class-validator` decorators:

```typescript
export class CreateUserDto {
  @IsEmail()
  @IsNotEmpty()
  email: string;

  @IsString()
  @MinLength(2)
  @MaxLength(100)
  name: string;
}
```

Enable global validation pipe in `main.ts`:

```typescript
app.useGlobalPipes(
  new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }),
);
```

---

## Exception Handling

Define domain exceptions and map them to HTTP errors via an exception filter:

```typescript
// Domain exception
export class UserNotFoundError extends Error {
  constructor(id: string) {
    super(`User ${id} not found`);
    this.name = "UserNotFoundError";
  }
}

// Exception filter
@Catch(UserNotFoundError)
export class UserNotFoundFilter implements ExceptionFilter {
  catch(exception: UserNotFoundError, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    ctx
      .getResponse()
      .status(404)
      .json({
        error: { code: "USER_NOT_FOUND", message: exception.message },
      });
  }
}
```

---

## Dependency Injection

- Always inject dependencies via constructor
- Use `@Inject('TOKEN')` for interface-based injection
- Never instantiate services with `new` inside other services
- Prefer `useClass` for single implementations; `useFactory` for complex setup

---

## Async / Transactions

- All repository methods are `async` — no blocking I/O
- Wrap multi-step operations in transactions
- Use `QueryRunner` for manual transaction control

---

## Authentication Guards

Protect endpoints using `JwtAuthGuard`. Authentication is opt-out, not opt-in — all endpoints are protected by default. Public endpoints must be explicitly marked:

```typescript
// Mark as public (auth skipped)
@Public()
@Get('health')
healthCheck() { ... }

// Protected endpoint (default)
@UseGuards(JwtAuthGuard)
@Get('me')
async getProfile(@GetUser() user: AuthUser): Promise<UserResponseDto> {
  return this.getOwnProfile.execute({ userId: user.id });
}
```

---

## Response Interceptors

Use a NestJS interceptor to apply the unified `{ data, error }` response envelope consistently across all endpoints, rather than constructing it manually in each controller:

```typescript
@Injectable()
export class ResponseEnvelopeInterceptor<T> implements NestInterceptor<
  T,
  ApiResponse<T>
> {
  intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Observable<ApiResponse<T>> {
    return next.handle().pipe(map((data) => ({ data, error: null })));
  }
}

// Register globally in main.ts
app.useGlobalInterceptors(new ResponseEnvelopeInterceptor());
```

Exception filters handle the `error` side of the envelope. Controllers return only the domain payload — the interceptor wraps it.

---

## Anti-Patterns to Avoid

- Business logic in controllers or repositories
- Use cases importing from infrastructure layer
- Shared mutable state in services
- Catching and swallowing errors silently
- Calling `save()` inside a domain entity constructor
- Using `any` in DTOs or domain objects
