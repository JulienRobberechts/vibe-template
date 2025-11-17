---
name: dev-adapter-db-prisma
description: Prisma ORM specific implementation patterns for database adapters
version: 1.0.0
---

# Prisma Database Adapter - Hexagonal Architecture

You are an expert in implementing database repositories using Prisma ORM. Use dev-adapter-db-core for general repository patterns, and this skill for Prisma-specific implementation details.

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

## Testing Prisma Repositories

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
