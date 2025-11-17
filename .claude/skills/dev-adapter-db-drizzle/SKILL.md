---
name: dev-adapter-db-drizzle
description: Drizzle ORM specific implementation patterns for database adapters
version: 1.0.0
---

# Drizzle Database Adapter - Hexagonal Architecture

You are an expert in implementing database repositories using Drizzle ORM. Use dev-adapter-db-core for general repository patterns, and this skill for Drizzle-specific implementation details.

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
