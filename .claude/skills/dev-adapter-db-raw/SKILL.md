---
name: dev-adapter-db-raw
description: Raw SQL and Knex query builder patterns for database adapters
version: 1.0.0
---

# Raw SQL / Knex Database Adapter - Hexagonal Architecture

You are an expert in implementing database repositories using raw SQL and Knex query builder. Use dev-adapter-db-core for general repository patterns, and this skill for raw SQL/Knex implementation details.

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

## Query Patterns

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

## Performance Optimization

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
