---
name: dev-adapter-db-core
description: Core repository patterns and practices for database adapters (technology-agnostic)
version: 1.0.0
---

# Database Adapter Core Patterns - Hexagonal Architecture

You are an expert in core database adapter patterns. This skill covers technology-agnostic repository patterns, entity mapping, transactions, and error handling. Use technology-specific skills (prisma, drizzle, raw) for ORM-specific details.


# Database Output Adapter Expert - Hexagonal Architecture

You are an expert in developing database output adapters in hexagonal architecture. You implement the repository pattern to provide clean database access while maintaining strict separation between domain logic and persistence concerns.

## Architectural Position

**Database Adapters as Output Adapters in Hexagonal Architecture:**
- Database repositories are **output adapters** in the infrastructure layer
- They implement repository **ports** (interfaces) defined in the application layer
- They translate domain entities to database models and vice versa
- They handle all database-specific concerns (connections, queries, transactions)
- They never expose database details to the domain/application layers
- They convert database errors to domain errors

**Key Principle**: Repository adapters should:
1. Implement repository ports (interfaces) from application layer
2. Map between domain entities and database models
3. Handle database connections and transactions
4. Convert database errors to domain exceptions
5. Never leak database concerns to upper layers

## Implementation Order: Fake First, Real Second

**CRITICAL: Always implement a fake adapter BEFORE the real adapter!**

When implementing database adapters, follow this order:
1. **First**: Create a fake (in-memory) repository implementation
2. **Then**: Create the real database repository implementation

### Why Fake Adapters First?

**Fake adapters enable:**
- ✅ Testing use cases immediately without database setup
- ✅ Fast, reliable tests with no external dependencies
- ✅ Easy simulation of edge cases and failures
- ✅ TDD workflow - test domain logic before infrastructure
- ✅ Development without database configuration

**Example workflow:**
1. Define repository port (interface) in application layer
2. **Implement fake repository with in-memory Map**
3. Write and test use cases using the fake
4. Implement real repository with database
5. Run integration tests with real database

See `/examples/fake-adapters/` for complete examples of fake repository implementations.

## Folder Structure

Typical database adapter structure:
```
infrastructure/
├── adapters/
│   └── database/
│       ├── repositories/           # Real repository implementations
│       │   ├── user.repository.ts
│       │   ├── order.repository.ts
│       │   └── payment.repository.ts
│       ├── fake/                   # Fake implementations for testing
│       │   ├── fake-user.repository.ts
│       │   ├── fake-order.repository.ts
│       │   └── fake-payment.repository.ts
│       ├── models/                 # Database models/schemas
│       │   ├── user.model.ts
│       │   ├── order.model.ts
│       │   └── payment.model.ts
│       ├── mappers/                # Entity ↔ Model mappers
│       │   ├── user.mapper.ts
│       │   ├── order.mapper.ts
│       │   └── payment.mapper.ts
│       ├── migrations/             # Database migrations
│       │   ├── 001_create_users.ts
│       │   └── 002_create_orders.ts
│       ├── seeds/                  # Seed data
│       │   └── users.seed.ts
│       ├── config/                 # Database configuration
│       │   └── database.config.ts
│       └── connection.ts           # Connection management
└── config/
    └── database.ts
```

## Repository Pattern

**Port (Interface) in Application Layer:**
```typescript
// application/ports/user-repository.port.ts
import { User } from '@/domain/entities/user';

export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  findAll(): Promise<User[]>;
  save(user: User): Promise<User>;
  update(user: User): Promise<User>;
  delete(id: string): Promise<void>;
  exists(id: string): Promise<boolean>;
}
```

**STEP 1: Fake Adapter (In-Memory Implementation for Testing):**
```typescript
// infrastructure/adapters/database/fake/fake-user.repository.ts
import { IUserRepository } from '@/application/ports/user-repository.port';
import { User } from '@/domain/entities/user';

export class FakeUserRepository implements IUserRepository {
  private users = new Map<string, User>();
  private emailIndex = new Map<string, string>(); // email -> id

  async findById(id: string): Promise<User | null> {
    return this.users.get(id) ?? null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const userId = this.emailIndex.get(email);
    if (!userId) return null;
    return this.users.get(userId) ?? null;
  }

  async findAll(): Promise<User[]> {
    return Array.from(this.users.values());
  }

  async save(user: User): Promise<User> {
    // Simulate unique constraint on email
    const existingUserId = this.emailIndex.get(user.email);
    if (existingUserId && existingUserId !== user.id) {
      throw new Error(`User with email ${user.email} already exists`);
    }

    this.users.set(user.id, user);
    this.emailIndex.set(user.email, user.id);
    return user;
  }

  async update(user: User): Promise<User> {
    if (!this.users.has(user.id)) {
      throw new Error(`User with id ${user.id} not found`);
    }

    const oldUser = this.users.get(user.id)!;
    if (oldUser.email !== user.email) {
      this.emailIndex.delete(oldUser.email);
      this.emailIndex.set(user.email, user.id);
    }

    this.users.set(user.id, user);
    return user;
  }

  async delete(id: string): Promise<void> {
    const user = this.users.get(id);
    if (user) {
      this.emailIndex.delete(user.email);
      this.users.delete(id);
    }
  }

  async exists(id: string): Promise<boolean> {
    return this.users.has(id);
  }

  // Test helper methods (not part of port interface)
  clear(): void {
    this.users.clear();
    this.emailIndex.clear();
  }

  size(): number {
    return this.users.size;
  }
}
```

**STEP 2: Real Adapter (Database Implementation):**
```typescript
// infrastructure/adapters/database/repositories/user.repository.ts
import { IUserRepository } from '@/application/ports/user-repository.port';
import { User } from '@/domain/entities/user';
import { UserModel } from '../models/user.model';
import { UserMapper } from '../mappers/user.mapper';
import { DatabaseError } from '@/domain/errors/database.error';

export class UserRepository implements IUserRepository {
  constructor(private readonly db: DatabaseConnection) {}

  async findById(id: string): Promise<User | null> {
    try {
      const userModel = await this.db.users.findUnique({
        where: { id },
      });

      if (!userModel) {
        return null;
      }

      return UserMapper.toDomain(userModel);
    } catch (error) {
      throw new DatabaseError('Failed to find user', { cause: error });
    }
  }

  async findByEmail(email: string): Promise<User | null> {
    try {
      const userModel = await this.db.users.findUnique({
        where: { email },
      });

      if (!userModel) {
        return null;
      }

      return UserMapper.toDomain(userModel);
    } catch (error) {
      throw new DatabaseError('Failed to find user by email', { cause: error });
    }
  }

  async findAll(): Promise<User[]> {
    try {
      const userModels = await this.db.users.findMany();
      return userModels.map(UserMapper.toDomain);
    } catch (error) {
      throw new DatabaseError('Failed to fetch users', { cause: error });
    }
  }

  async save(user: User): Promise<User> {
    try {
      const userModel = UserMapper.toPersistence(user);

      const savedModel = await this.db.users.create({
        data: userModel,
      });

      return UserMapper.toDomain(savedModel);
    } catch (error) {
      throw new DatabaseError('Failed to save user', { cause: error });
    }
  }

  async update(user: User): Promise<User> {
    try {
      const userModel = UserMapper.toPersistence(user);

      const updatedModel = await this.db.users.update({
        where: { id: user.id },
        data: userModel,
      });

      return UserMapper.toDomain(updatedModel);
    } catch (error) {
      throw new DatabaseError('Failed to update user', { cause: error });
    }
  }

  async delete(id: string): Promise<void> {
    try {
      await this.db.users.delete({
        where: { id },
      });
    } catch (error) {
      throw new DatabaseError('Failed to delete user', { cause: error });
    }
  }

  async exists(id: string): Promise<boolean> {
    try {
      const count = await this.db.users.count({
        where: { id },
      });
      return count > 0;
    } catch (error) {
      throw new DatabaseError('Failed to check user existence', { cause: error });
    }
  }
}
```

## Entity ↔ Model Mapping

**Domain Entity (Pure Business Logic):**
```typescript
// domain/entities/user.ts
import { Email } from '@/domain/value-objects/email';

export class User {
  constructor(
    public readonly id: string,
    public readonly email: Email,
    public readonly name: string,
    public readonly role: 'admin' | 'user' | 'guest',
    public readonly createdAt: Date,
    public readonly updatedAt: Date
  ) {}

  // Domain methods
  isAdmin(): boolean {
    return this.role === 'admin';
  }

  canAccessResource(resourceOwnerId: string): boolean {
    return this.isAdmin() || this.id === resourceOwnerId;
  }
}
```

**Database Model (Persistence):**
```typescript
// infrastructure/adapters/database/models/user.model.ts

// With Prisma (generated)
// See schema.prisma for definition

// With TypeORM
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn } from 'typeorm';

@Entity('users')
export class UserModel {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;

  @Column({ type: 'enum', enum: ['admin', 'user', 'guest'] })
  role: 'admin' | 'user' | 'guest';

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

// Plain object (for raw SQL)
export interface UserModel {
  id: string;
  email: string;
  name: string;
  role: 'admin' | 'user' | 'guest';
  created_at: Date;
  updated_at: Date;
}
```

**Mapper (Bidirectional Conversion):**
```typescript
// infrastructure/adapters/database/mappers/user.mapper.ts
import { User } from '@/domain/entities/user';
import { Email } from '@/domain/value-objects/email';
import { UserModel } from '../models/user.model';

export class UserMapper {
  /**
   * Convert database model to domain entity
   */
  static toDomain(model: UserModel): User {
    return new User(
      model.id,
      new Email(model.email),
      model.name,
      model.role,
      model.createdAt,
      model.updatedAt
    );
  }

  /**
   * Convert domain entity to database model
   */
  static toPersistence(entity: User): Omit<UserModel, 'createdAt' | 'updatedAt'> {
    return {
      id: entity.id,
      email: entity.email.value,
      name: entity.name,
      role: entity.role,
    };
  }

  /**
   * Batch conversion to domain
   */
  static toDomainList(models: UserModel[]): User[] {
    return models.map((model) => this.toDomain(model));
  }
}
```

## Modern Database Libraries
## Transaction Management

**Transaction Port (Interface):**
```typescript
// application/ports/transaction.port.ts
export interface ITransactionManager {
  execute<T>(callback: (trx: unknown) => Promise<T>): Promise<T>;
}
```

**Prisma Transaction:**
```typescript
export class PrismaTransactionManager implements ITransactionManager {
  constructor(private readonly prisma: PrismaClient) {}

  async execute<T>(callback: (trx: PrismaClient) => Promise<T>): Promise<T> {
    return this.prisma.$transaction(async (trx) => {
      return callback(trx);
    });
  }
}

// Usage in use case
class TransferMoneyUseCase {
  constructor(
    private readonly transactionManager: ITransactionManager,
    private readonly accountRepo: IAccountRepository
  ) {}

  async execute(fromId: string, toId: string, amount: number): Promise<void> {
    await this.transactionManager.execute(async (trx) => {
      const fromAccount = await this.accountRepo.findById(fromId, trx);
      const toAccount = await this.accountRepo.findById(toId, trx);

      fromAccount.withdraw(amount);
      toAccount.deposit(amount);

      await this.accountRepo.update(fromAccount, trx);
      await this.accountRepo.update(toAccount, trx);
    });
  }
}
```

**Drizzle Transaction:**
```typescript
export class DrizzleTransactionManager implements ITransactionManager {
  constructor(private readonly db: ReturnType<typeof drizzle>) {}

  async execute<T>(callback: (trx: any) => Promise<T>): Promise<T> {
    return this.db.transaction(async (trx) => {
      return callback(trx);
    });
  }
}
```

**Knex Transaction:**
```typescript
export class KnexTransactionManager implements ITransactionManager {
  constructor(private readonly knex: Knex) {}

  async execute<T>(callback: (trx: Knex.Transaction) => Promise<T>): Promise<T> {
    return this.knex.transaction(async (trx) => {
      return callback(trx);
    });
  }
}
```

**Raw SQL Transaction:**
```typescript
export class PostgresTransactionManager implements ITransactionManager {
  constructor(private readonly pool: Pool) {}

  async execute<T>(callback: (client: PoolClient) => Promise<T>): Promise<T> {
    const client = await this.pool.connect();

    try {
      await client.query('BEGIN');
      const result = await callback(client);
      await client.query('COMMIT');
      return result;
    } catch (error) {
      await client.query('ROLLBACK');
      throw error;
    } finally {
      client.release();
    }
  }
}
```

## Connection Management

## Error Handling

## Error Handling

**Database Error Mapping:**
```typescript
import { Prisma } from '@prisma/client';
import {
  DatabaseError,
  UniqueConstraintError,
  NotFoundError,
  ForeignKeyError
} from '@/domain/errors';

export function handleDatabaseError(error: unknown): never {
  // Prisma errors
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    switch (error.code) {
      case 'P2002': // Unique constraint violation
        throw new UniqueConstraintError(
          `Duplicate value for field: ${error.meta?.target}`
        );
      case 'P2025': // Record not found
        throw new NotFoundError('Record not found');
      case 'P2003': // Foreign key constraint
        throw new ForeignKeyError('Foreign key constraint failed');
      default:
        throw new DatabaseError(`Database error: ${error.code}`, { cause: error });
    }
  }

  // PostgreSQL errors (raw SQL)
  if (error && typeof error === 'object' && 'code' in error) {
    const pgError = error as { code: string; detail?: string };

    switch (pgError.code) {
      case '23505': // unique_violation
        throw new UniqueConstraintError('Duplicate value');
      case '23503': // foreign_key_violation
        throw new ForeignKeyError('Foreign key constraint failed');
      case '23502': // not_null_violation
        throw new DatabaseError('Required field is missing');
      default:
        throw new DatabaseError('Database operation failed', { cause: error });
    }
  }

  throw new DatabaseError('Unknown database error', { cause: error });
}

// Usage in repository
async save(user: User): Promise<User> {
  try {
    const data = UserMapper.toPersistence(user);
    const created = await this.prisma.user.create({ data });
    return UserMapper.toDomain(created);
  } catch (error) {
    handleDatabaseError(error);
  }
}
```

## Migrations

## Checklist for New Repository

## Checklist for New Repository

When creating a new repository:
- [ ] Define repository port interface in application layer
- [ ] Implement adapter in infrastructure layer
- [ ] Create database model/schema
- [ ] Create bidirectional mapper (domain ↔ persistence)
- [ ] Handle all database errors and convert to domain errors
- [ ] Never expose database details to upper layers
- [ ] Support transactions when needed
- [ ] Add indexes for performance
- [ ] Write unit tests with mocked database
- [ ] Write integration tests with test database
- [ ] Document query patterns
- [ ] Consider caching for read-heavy operations

## Common Anti-Patterns to Avoid

**❌ Exposing database models to domain:**
```typescript
// BAD: Use case depends on Prisma model
class GetUserUseCase {
  execute(id: string): Promise<PrismaUser> {
    return this.userRepository.findById(id);
  }
}
```

**✅ Use domain entities:**
```typescript
// GOOD: Use case uses domain entity
class GetUserUseCase {
  execute(id: string): Promise<User> {
    return this.userRepository.findById(id);
  }
}
```

**❌ Business logic in repository:**
```typescript
// BAD: Repository contains business logic
async save(user: User): Promise<User> {
  if (user.age < 18) {
    throw new Error('Too young');
  }
  return this.prisma.user.create({ data: user });
}
```

**✅ Keep repository thin:**
```typescript
// GOOD: Repository only handles persistence
async save(user: User): Promise<User> {
  const data = UserMapper.toPersistence(user);
  const created = await this.prisma.user.create({ data });
  return UserMapper.toDomain(created);
}
```

**❌ Direct database access in use cases:**
```typescript
// BAD: Use case queries database directly
class GetUserUseCase {
  constructor(private prisma: PrismaClient) {}

  execute(id: string) {
    return this.prisma.user.findUnique({ where: { id } });
  }
}
```

**✅ Use repository abstraction:**
```typescript
// GOOD: Use case depends on repository interface
class GetUserUseCase {
  constructor(private userRepository: IUserRepository) {}

  execute(id: string): Promise<User | null> {
    return this.userRepository.findById(id);
  }
}
```

---

Your database adapters are critical for data persistence. Implement them as clean, testable repositories that strictly separate domain logic from database concerns. Keep the domain pure, and let adapters handle all the messy persistence details.
