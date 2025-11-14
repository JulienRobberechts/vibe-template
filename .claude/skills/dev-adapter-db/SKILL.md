---
name: dev-adapter-db
description: Expert in developing database output adapters in hexagonal architecture. Covers repository pattern, database libraries, connection management, queries, transactions, migrations, and testing.
version: 1.0.0
---

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

### 1. Prisma (Recommended for TypeScript)

**Strengths:**
- Type-safe query builder
- Auto-generated types from schema
- Excellent DX with migrations
- Built-in connection pooling
- Multi-database support

**Schema Definition:**
```prisma
// prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  role      Role     @default(USER)
  orders    Order[]
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("users")
}

enum Role {
  ADMIN
  USER
  GUEST
}

model Order {
  id        String   @id @default(uuid())
  userId    String   @map("user_id")
  user      User     @relation(fields: [userId], references: [id])
  total     Decimal  @db.Decimal(10, 2)
  status    String
  createdAt DateTime @default(now()) @map("created_at")

  @@map("orders")
}
```

**Repository Implementation:**
```typescript
import { PrismaClient } from '@prisma/client';
import { IUserRepository } from '@/application/ports/user-repository.port';
import { User } from '@/domain/entities/user';
import { UserMapper } from '../mappers/user.mapper';

export class PrismaUserRepository implements IUserRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    const user = await this.prisma.user.findUnique({
      where: { id },
      include: { orders: true }, // Include relations if needed
    });

    return user ? UserMapper.toDomain(user) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const user = await this.prisma.user.findUnique({
      where: { email },
    });

    return user ? UserMapper.toDomain(user) : null;
  }

  async findAll(): Promise<User[]> {
    const users = await this.prisma.user.findMany({
      orderBy: { createdAt: 'desc' },
    });

    return users.map(UserMapper.toDomain);
  }

  async save(user: User): Promise<User> {
    const data = UserMapper.toPersistence(user);

    const created = await this.prisma.user.create({
      data,
    });

    return UserMapper.toDomain(created);
  }

  async update(user: User): Promise<User> {
    const data = UserMapper.toPersistence(user);

    const updated = await this.prisma.user.update({
      where: { id: user.id },
      data,
    });

    return UserMapper.toDomain(updated);
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({
      where: { id },
    });
  }
}
```

### 2. Drizzle ORM (Modern, Type-Safe)

**Strengths:**
- Thin layer over SQL
- Excellent TypeScript inference
- SQL-like query builder
- Zero dependencies
- Great performance

**Schema Definition:**
```typescript
// infrastructure/adapters/database/schema.ts
import { pgTable, uuid, varchar, timestamp, pgEnum } from 'drizzle-orm/pg-core';

export const roleEnum = pgEnum('role', ['ADMIN', 'USER', 'GUEST']);

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  name: varchar('name', { length: 255 }).notNull(),
  role: roleEnum('role').notNull().default('USER'),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
});

export const orders = pgTable('orders', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull().references(() => users.id),
  total: varchar('total', { length: 20 }).notNull(),
  status: varchar('status', { length: 50 }).notNull(),
  createdAt: timestamp('created_at').notNull().defaultNow(),
});
```

**Repository Implementation:**
```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { eq } from 'drizzle-orm';
import { users } from '../schema';
import { IUserRepository } from '@/application/ports/user-repository.port';
import { User } from '@/domain/entities/user';
import { UserMapper } from '../mappers/user.mapper';

export class DrizzleUserRepository implements IUserRepository {
  constructor(private readonly db: ReturnType<typeof drizzle>) {}

  async findById(id: string): Promise<User | null> {
    const [user] = await this.db
      .select()
      .from(users)
      .where(eq(users.id, id))
      .limit(1);

    return user ? UserMapper.toDomain(user) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const [user] = await this.db
      .select()
      .from(users)
      .where(eq(users.email, email))
      .limit(1);

    return user ? UserMapper.toDomain(user) : null;
  }

  async findAll(): Promise<User[]> {
    const result = await this.db.select().from(users);
    return result.map(UserMapper.toDomain);
  }

  async save(user: User): Promise<User> {
    const data = UserMapper.toPersistence(user);

    const [created] = await this.db
      .insert(users)
      .values(data)
      .returning();

    return UserMapper.toDomain(created);
  }

  async update(user: User): Promise<User> {
    const data = UserMapper.toPersistence(user);

    const [updated] = await this.db
      .update(users)
      .set(data)
      .where(eq(users.id, user.id))
      .returning();

    return UserMapper.toDomain(updated);
  }

  async delete(id: string): Promise<void> {
    await this.db.delete(users).where(eq(users.id, id));
  }
}
```

### 3. TypeORM (Mature, Feature-Rich)

**Strengths:**
- Mature ecosystem
- Active Record & Data Mapper patterns
- Migrations and seeding
- Multiple database support
- Decorators for entities

**Repository Implementation:**
```typescript
import { Repository } from 'typeorm';
import { UserModel } from '../models/user.model';
import { IUserRepository } from '@/application/ports/user-repository.port';
import { User } from '@/domain/entities/user';
import { UserMapper } from '../mappers/user.mapper';

export class TypeORMUserRepository implements IUserRepository {
  constructor(private readonly repository: Repository<UserModel>) {}

  async findById(id: string): Promise<User | null> {
    const user = await this.repository.findOne({
      where: { id },
    });

    return user ? UserMapper.toDomain(user) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const user = await this.repository.findOne({
      where: { email },
    });

    return user ? UserMapper.toDomain(user) : null;
  }

  async findAll(): Promise<User[]> {
    const users = await this.repository.find({
      order: { createdAt: 'DESC' },
    });

    return users.map(UserMapper.toDomain);
  }

  async save(user: User): Promise<User> {
    const data = UserMapper.toPersistence(user);
    const entity = this.repository.create(data);
    const saved = await this.repository.save(entity);
    return UserMapper.toDomain(saved);
  }

  async update(user: User): Promise<User> {
    const data = UserMapper.toPersistence(user);
    await this.repository.update(user.id, data);
    const updated = await this.repository.findOneOrFail({
      where: { id: user.id },
    });
    return UserMapper.toDomain(updated);
  }

  async delete(id: string): Promise<void> {
    await this.repository.delete(id);
  }
}
```

### 4. Knex (SQL Query Builder)

**Strengths:**
- Raw SQL control
- Migration system
- Transaction support
- Multiple database drivers

**Repository Implementation:**
```typescript
import { Knex } from 'knex';
import { IUserRepository } from '@/application/ports/user-repository.port';
import { User } from '@/domain/entities/user';
import { UserMapper } from '../mappers/user.mapper';

export class KnexUserRepository implements IUserRepository {
  private readonly tableName = 'users';

  constructor(private readonly knex: Knex) {}

  async findById(id: string): Promise<User | null> {
    const user = await this.knex(this.tableName)
      .where({ id })
      .first();

    return user ? UserMapper.toDomain(user) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const user = await this.knex(this.tableName)
      .where({ email })
      .first();

    return user ? UserMapper.toDomain(user) : null;
  }

  async findAll(): Promise<User[]> {
    const users = await this.knex(this.tableName)
      .select('*')
      .orderBy('created_at', 'desc');

    return users.map(UserMapper.toDomain);
  }

  async save(user: User): Promise<User> {
    const data = UserMapper.toPersistence(user);

    const [created] = await this.knex(this.tableName)
      .insert(data)
      .returning('*');

    return UserMapper.toDomain(created);
  }

  async update(user: User): Promise<User> {
    const data = UserMapper.toPersistence(user);

    const [updated] = await this.knex(this.tableName)
      .where({ id: user.id })
      .update(data)
      .returning('*');

    return UserMapper.toDomain(updated);
  }

  async delete(id: string): Promise<void> {
    await this.knex(this.tableName).where({ id }).delete();
  }
}
```

### 5. node-postgres (Raw SQL)

**Strengths:**
- Maximum control
- No ORM overhead
- Direct SQL queries
- Best performance

**Repository Implementation:**
```typescript
import { Pool } from 'pg';
import { IUserRepository } from '@/application/ports/user-repository.port';
import { User } from '@/domain/entities/user';
import { UserMapper } from '../mappers/user.mapper';

export class PostgresUserRepository implements IUserRepository {
  constructor(private readonly pool: Pool) {}

  async findById(id: string): Promise<User | null> {
    const result = await this.pool.query(
      'SELECT * FROM users WHERE id = $1',
      [id]
    );

    if (result.rows.length === 0) {
      return null;
    }

    return UserMapper.toDomain(result.rows[0]);
  }

  async findByEmail(email: string): Promise<User | null> {
    const result = await this.pool.query(
      'SELECT * FROM users WHERE email = $1',
      [email]
    );

    if (result.rows.length === 0) {
      return null;
    }

    return UserMapper.toDomain(result.rows[0]);
  }

  async findAll(): Promise<User[]> {
    const result = await this.pool.query(
      'SELECT * FROM users ORDER BY created_at DESC'
    );

    return result.rows.map(UserMapper.toDomain);
  }

  async save(user: User): Promise<User> {
    const data = UserMapper.toPersistence(user);

    const result = await this.pool.query(
      `INSERT INTO users (id, email, name, role)
       VALUES ($1, $2, $3, $4)
       RETURNING *`,
      [data.id, data.email, data.name, data.role]
    );

    return UserMapper.toDomain(result.rows[0]);
  }

  async update(user: User): Promise<User> {
    const data = UserMapper.toPersistence(user);

    const result = await this.pool.query(
      `UPDATE users
       SET email = $2, name = $3, role = $4, updated_at = NOW()
       WHERE id = $1
       RETURNING *`,
      [data.id, data.email, data.name, data.role]
    );

    return UserMapper.toDomain(result.rows[0]);
  }

  async delete(id: string): Promise<void> {
    await this.pool.query('DELETE FROM users WHERE id = $1', [id]);
  }
}
```

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

**Singleton Connection:**
```typescript
// infrastructure/adapters/database/connection.ts
import { PrismaClient } from '@prisma/client';

let prisma: PrismaClient | null = null;

export function getPrismaClient(): PrismaClient {
  if (!prisma) {
    prisma = new PrismaClient({
      log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
    });
  }

  return prisma;
}

export async function disconnectDatabase(): Promise<void> {
  if (prisma) {
    await prisma.$disconnect();
    prisma = null;
  }
}

// Graceful shutdown
process.on('beforeExit', async () => {
  await disconnectDatabase();
});
```

**Connection Pool Configuration:**
```typescript
import { Pool } from 'pg';

const pool = new Pool({
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT),
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20, // Maximum connections in pool
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Connection health check
export async function checkDatabaseConnection(): Promise<boolean> {
  try {
    const client = await pool.connect();
    await client.query('SELECT 1');
    client.release();
    return true;
  } catch (error) {
    return false;
  }
}
```

## Query Patterns

**Filtering & Pagination:**
```typescript
interface FindUsersOptions {
  role?: string;
  search?: string;
  page?: number;
  limit?: number;
}

async findUsers(options: FindUsersOptions): Promise<{ users: User[]; total: number }> {
  const { role, search, page = 1, limit = 20 } = options;

  const where: any = {};

  if (role) {
    where.role = role;
  }

  if (search) {
    where.OR = [
      { name: { contains: search, mode: 'insensitive' } },
      { email: { contains: search, mode: 'insensitive' } },
    ];
  }

  const [users, total] = await Promise.all([
    this.prisma.user.findMany({
      where,
      skip: (page - 1) * limit,
      take: limit,
      orderBy: { createdAt: 'desc' },
    }),
    this.prisma.user.count({ where }),
  ]);

  return {
    users: users.map(UserMapper.toDomain),
    total,
  };
}
```

**Relations Loading:**
```typescript
async findUserWithOrders(id: string): Promise<UserWithOrders | null> {
  const user = await this.prisma.user.findUnique({
    where: { id },
    include: {
      orders: {
        orderBy: { createdAt: 'desc' },
        take: 10,
      },
    },
  });

  if (!user) {
    return null;
  }

  return {
    ...UserMapper.toDomain(user),
    orders: user.orders.map(OrderMapper.toDomain),
  };
}
```

**Aggregations:**
```typescript
async getUserStats(userId: string): Promise<UserStats> {
  const stats = await this.prisma.order.aggregate({
    where: { userId },
    _count: { id: true },
    _sum: { total: true },
    _avg: { total: true },
  });

  return {
    orderCount: stats._count.id,
    totalSpent: stats._sum.total ?? 0,
    averageOrder: stats._avg.total ?? 0,
  };
}
```

**Bulk Operations:**
```typescript
async createMany(users: User[]): Promise<void> {
  const data = users.map(UserMapper.toPersistence);

  await this.prisma.user.createMany({
    data,
    skipDuplicates: true,
  });
}

async updateMany(userIds: string[], updates: Partial<User>): Promise<void> {
  await this.prisma.user.updateMany({
    where: { id: { in: userIds } },
    data: UserMapper.toPersistence(updates as User),
  });
}
```

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

**Prisma Migrations:**
```bash
# Create migration
npx prisma migrate dev --name add_user_table

# Apply migrations
npx prisma migrate deploy

# Reset database
npx prisma migrate reset
```

**Drizzle Migrations:**
```typescript
// drizzle.config.ts
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  schema: './src/infrastructure/adapters/database/schema.ts',
  out: './migrations',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

```bash
# Generate migration
npx drizzle-kit generate

# Apply migration
npx drizzle-kit migrate
```

**Knex Migrations:**
```typescript
// migrations/20240115_create_users_table.ts
import { Knex } from 'knex';

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable('users', (table) => {
    table.uuid('id').primary().defaultTo(knex.raw('gen_random_uuid()'));
    table.string('email', 255).notNullable().unique();
    table.string('name', 255).notNullable();
    table.enum('role', ['admin', 'user', 'guest']).notNullable().defaultTo('user');
    table.timestamps(true, true);
  });
}

export async function down(knex: Knex): Promise<void> {
  await knex.schema.dropTable('users');
}
```

```bash
# Run migrations
npx knex migrate:latest

# Rollback
npx knex migrate:rollback
```

**Raw SQL Migrations:**
```sql
-- migrations/001_create_users_table.sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL UNIQUE,
  name VARCHAR(255) NOT NULL,
  role VARCHAR(20) NOT NULL DEFAULT 'user',
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

## Testing Database Adapters

**Unit Tests (Mocked Database):**
```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { PrismaUserRepository } from './user.repository';
import { User } from '@/domain/entities/user';
import { Email } from '@/domain/value-objects/email';

describe('PrismaUserRepository', () => {
  let repository: PrismaUserRepository;
  let mockPrisma: any;

  beforeEach(() => {
    mockPrisma = {
      user: {
        findUnique: vi.fn(),
        create: vi.fn(),
        update: vi.fn(),
        delete: vi.fn(),
      },
    };

    repository = new PrismaUserRepository(mockPrisma);
  });

  describe('findById', () => {
    it('should return user when found', async () => {
      const mockUser = {
        id: '123',
        email: 'test@example.com',
        name: 'Test User',
        role: 'user',
        createdAt: new Date(),
        updatedAt: new Date(),
      };

      mockPrisma.user.findUnique.mockResolvedValue(mockUser);

      const user = await repository.findById('123');

      expect(user).toBeInstanceOf(User);
      expect(user?.email.value).toBe('test@example.com');
      expect(mockPrisma.user.findUnique).toHaveBeenCalledWith({
        where: { id: '123' },
      });
    });

    it('should return null when user not found', async () => {
      mockPrisma.user.findUnique.mockResolvedValue(null);

      const user = await repository.findById('999');

      expect(user).toBeNull();
    });

    it('should throw DatabaseError on failure', async () => {
      mockPrisma.user.findUnique.mockRejectedValue(new Error('DB error'));

      await expect(repository.findById('123')).rejects.toThrow('Failed to find user');
    });
  });

  describe('save', () => {
    it('should create and return user', async () => {
      const user = new User(
        '123',
        new Email('test@example.com'),
        'Test User',
        'user',
        new Date(),
        new Date()
      );

      const mockCreated = {
        id: '123',
        email: 'test@example.com',
        name: 'Test User',
        role: 'user',
        createdAt: new Date(),
        updatedAt: new Date(),
      };

      mockPrisma.user.create.mockResolvedValue(mockCreated);

      const result = await repository.save(user);

      expect(result).toBeInstanceOf(User);
      expect(mockPrisma.user.create).toHaveBeenCalled();
    });
  });
});
```

**Integration Tests (Test Database):**
```typescript
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import { PrismaClient } from '@prisma/client';
import { PrismaUserRepository } from './user.repository';
import { User } from '@/domain/entities/user';
import { Email } from '@/domain/value-objects/email';

describe('PrismaUserRepository Integration', () => {
  let prisma: PrismaClient;
  let repository: PrismaUserRepository;

  beforeAll(async () => {
    prisma = new PrismaClient({
      datasources: {
        db: {
          url: process.env.TEST_DATABASE_URL,
        },
      },
    });

    repository = new PrismaUserRepository(prisma);

    // Run migrations
    // await prisma.$executeRaw`...`;
  });

  afterAll(async () => {
    await prisma.$disconnect();
  });

  beforeEach(async () => {
    // Clean database before each test
    await prisma.user.deleteMany();
  });

  it('should save and retrieve user', async () => {
    const user = new User(
      crypto.randomUUID(),
      new Email('test@example.com'),
      'Test User',
      'user',
      new Date(),
      new Date()
    );

    await repository.save(user);

    const retrieved = await repository.findById(user.id);

    expect(retrieved).not.toBeNull();
    expect(retrieved?.email.value).toBe('test@example.com');
    expect(retrieved?.name).toBe('Test User');
  });

  it('should handle unique constraint violation', async () => {
    const user1 = new User(
      crypto.randomUUID(),
      new Email('test@example.com'),
      'User 1',
      'user',
      new Date(),
      new Date()
    );

    const user2 = new User(
      crypto.randomUUID(),
      new Email('test@example.com'), // Same email
      'User 2',
      'user',
      new Date(),
      new Date()
    );

    await repository.save(user1);

    await expect(repository.save(user2)).rejects.toThrow(UniqueConstraintError);
  });

  it('should update existing user', async () => {
    const user = new User(
      crypto.randomUUID(),
      new Email('test@example.com'),
      'Original Name',
      'user',
      new Date(),
      new Date()
    );

    await repository.save(user);

    const updatedUser = new User(
      user.id,
      user.email,
      'Updated Name',
      user.role,
      user.createdAt,
      new Date()
    );

    await repository.update(updatedUser);

    const retrieved = await repository.findById(user.id);

    expect(retrieved?.name).toBe('Updated Name');
  });
});
```

**Test Database Setup (Docker):**
```yaml
# docker-compose.test.yml
version: '3.8'
services:
  test-db:
    image: postgres:15
    environment:
      POSTGRES_DB: test_db
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: test_password
    ports:
      - '5433:5432'
    tmpfs:
      - /var/lib/postgresql/data
```

## Performance Optimization

**Connection Pooling:**
```typescript
// Prisma connection pooling
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: `${process.env.DATABASE_URL}?connection_limit=10&pool_timeout=20`,
    },
  },
});
```

**Query Optimization:**
```typescript
// ❌ N+1 Query Problem
async getUsersWithOrders(): Promise<UserWithOrders[]> {
  const users = await this.prisma.user.findMany();

  // This creates N additional queries!
  for (const user of users) {
    user.orders = await this.prisma.order.findMany({
      where: { userId: user.id },
    });
  }

  return users;
}

// ✅ Use eager loading
async getUsersWithOrders(): Promise<UserWithOrders[]> {
  const users = await this.prisma.user.findMany({
    include: {
      orders: true, // Single query with JOIN
    },
  });

  return users.map(UserWithOrdersMapper.toDomain);
}
```

**Indexing:**
```sql
-- Add indexes for frequently queried fields
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);

-- Composite index
CREATE INDEX idx_users_role_created_at ON users(role, created_at DESC);
```

**Caching Layer:**
```typescript
import { Redis } from 'ioredis';

export class CachedUserRepository implements IUserRepository {
  constructor(
    private readonly repository: IUserRepository,
    private readonly redis: Redis
  ) {}

  async findById(id: string): Promise<User | null> {
    // Check cache
    const cached = await this.redis.get(`user:${id}`);

    if (cached) {
      return JSON.parse(cached);
    }

    // Query database
    const user = await this.repository.findById(id);

    if (user) {
      // Cache for 5 minutes
      await this.redis.setex(`user:${id}`, 300, JSON.stringify(user));
    }

    return user;
  }

  async save(user: User): Promise<User> {
    const saved = await this.repository.save(user);

    // Invalidate cache
    await this.redis.del(`user:${saved.id}`);

    return saved;
  }
}
```

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
