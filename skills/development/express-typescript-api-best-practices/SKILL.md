---
name: express-typescript-api-best-practices
description: Professional-grade REST API architecture with Express.js and TypeScript following SOLID principles, layered architecture, transaction management, JWT authentication with role-based authorization (RBAC), input validation with Zod, OpenAPI/Swagger documentation, standardized response format, and production-ready patterns. Use when building or refactoring REST APIs with Express + TypeScript that require enterprise-level code quality, maintainability, scalability, and security.
---

# Express + TypeScript REST API Best Practices

Professional skill for building production-ready REST APIs with Express.js and TypeScript, following SOLID principles and enterprise-grade architectural patterns.

## Core Principles

### 1. **Layered Architecture (Separation of Concerns)**

Follow strict layer separation to achieve **Single Responsibility Principle (SRP)** and **Dependency Inversion Principle (DIP)**:

```
Routes → Controllers → Services → Models → Database
```

**Layer Responsibilities:**

- **Routes** (`src/v1/routes/`): Define HTTP endpoints, apply middleware
- **Controllers** (`src/controllers/`): Handle HTTP request/response, format data
- **Services** (`src/services/`): Contain business logic, orchestrate transactions
- **Models** (`src/models/`): Define data structures (ORM models)
- **Middlewares** (`src/middlewares/`): Cross-cutting concerns (auth, validation)

**Implementation Rules:**

```typescript
// ❌ BAD: Business logic in controller
export const createUser = async (req: Request, res: Response) => {
  const hashedPassword = await argon2.hash(req.body.password);
  const user = await User.create({ ...req.body, password: hashedPassword });
  res.json(user);
};

// ✅ GOOD: Controller delegates to service
export const createUser = async (req: Request, res: Response) => {
  try {
    const user = await crearUsuario(req.body);
    res.status(201).json({
      status: "ok",
      message: "Usuario creado correctamente",
      data: user
    });
  } catch (error: any) {
    res.status(500).json({
      status: "error",
      message: "Error al crear usuario",
      error: error.message
    });
  }
};
```

### 2. **Standardized API Response Format**

ALL endpoints MUST return consistent response structure:

```typescript
// Success response
{
  status: "ok",
  message: string,
  data?: any  // Optional
}

// Error response
{
  status: "error",
  message: string,
  error?: string,     // Detailed error
  code?: string,      // Error code (e.g., "TOKEN_INVALID")
  detalles?: object   // Additional context
}
```

**HTTP Status Codes:**
- `200` - Success (GET, PUT, DELETE)
- `201` - Resource created (POST)
- `400` - Validation error
- `401` - Unauthorized (missing/invalid token)
- `403` - Forbidden (insufficient permissions)
- `404` - Resource not found
- `500` - Server error

### 3. **Transaction Management Pattern**

For operations involving multiple database changes, ALWAYS use transactions:

```typescript
export const crearDetalleVenta = async (detalleVentaData: any[]) => {
  const transaction = await sequelize.transaction();

  try {
    // 1. Validate input
    if (!Array.isArray(detalleVentaData) || detalleVentaData.length === 0) {
      throw new Error('detalleVentaData debe ser un array no vacío');
    }

    // 2. Batch load related data (optimization)
    const productoIds = [...new Set(detalleVentaData.map(i => i.producto_id))];
    const productos = await Producto.findAll({
      where: { id: productoIds },
      transaction
    });
    const productoMap = new Map(productos.map(p => [p.id, p]));

    // 3. Process each item with business rules
    for (const item of detalleVentaData) {
      const producto = productoMap.get(item.producto_id);
      if (!producto) throw new Error(`Producto ${item.producto_id} no encontrado`);

      // Auto-fill fields
      item.precio_unitario = item.precio_unitario ?? Number(producto.precio_minorista);
      item.sub_total = item.sub_total ?? parseFloat((item.precio_unitario * item.cantidad).toFixed(2));

      // Create related records (stock movement)
      const movimiento = await crearMovimiento({
        tipo: "salida",
        producto_id: item.producto_id,
        almacen_id: item.almacen_id,
        cantidad: item.cantidad,
        precio_unitario: item.precio_unitario,
        descripcion: `Salida por venta ID: ${item.venta_id}`
      }, transaction);

      item.movimiento_id = movimiento.id;
    }

    // 4. Bulk insert (performance)
    const detalles = await DetalleVenta.bulkCreate(detalleVentaData, { transaction });

    // 5. Update parent record with locking
    const venta = await Venta.findOne({
      where: { id: detalleVentaData[0].venta_id },
      transaction,
      lock: transaction.LOCK.UPDATE  // Prevent race conditions
    });

    if (!venta) throw new Error('Venta no encontrada');

    const totalSubtotales = detalleVentaData.reduce((sum, d) => sum + d.sub_total, 0);
    if (venta.total === 0 || venta.total < totalSubtotales) {
      venta.total = totalSubtotales;
      await venta.save({ transaction });
    }

    await transaction.commit();
    return detalles;
  } catch (error) {
    await transaction.rollback();
    console.error('Error en transacción:', error);
    throw error;
  }
};
```

**Transaction Best Practices:**
- Pass transaction to ALL database operations within the scope
- Use optimistic locking (`lock: transaction.LOCK.UPDATE`) for concurrent updates
- Batch load related data BEFORE the loop (N+1 query prevention)
- Always rollback on error, commit on success
- Validate ALL input at the beginning

### 4. **JWT Authentication & Role-Based Authorization (RBAC)**

**Authentication Middleware (`autenticarToken`):**

```typescript
export const autenticarToken = (req: AuthRequest, res: Response, next: NextFunction): void => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN

  if (!token) {
    res.status(401).json({
      status: "error",
      message: "Acceso no autorizado",
      error: "No se proporcionó un token de autenticación",
      code: "TOKEN_NOT_PROVIDED"
    });
    return;
  }

  try {
    const payload = verificarToken(token);
    req.usuario = payload;  // Attach user to request
    next();
  } catch (error) {
    res.status(403).json({
      status: "error",
      message: "Token inválido",
      error: "El token ha expirado o es inválido",
      code: "TOKEN_INVALID"
    });
  }
};
```

**Authorization Middleware (`verificarRol`):**

```typescript
export const verificarRol = (...rolesPermitidos: string[]) => {
  return async (req: AuthRequest, res: Response, next: NextFunction): Promise<void> => {
    if (!req.usuario) {
      res.status(401).json({
        status: "error",
        message: "Autenticación requerida",
        code: "AUTHENTICATION_REQUIRED"
      });
      return;
    }

    // Cache roles in request to avoid multiple DB queries
    if (!req.usuario.roles) {
      const rolesAsignados = await UsuarioRol.findAll({
        where: { usuario_id: req.usuario.id },
        include: [{ model: Rol, as: "rol" }]
      });
      req.usuario.roles = rolesAsignados.map((ur: any) => ur.rol.nombre);
    }

    const tienePermiso = (req.usuario.roles || []).some(rol =>
      rolesPermitidos.includes(rol)
    );

    if (!tienePermiso) {
      res.status(403).json({
        status: "error",
        message: "Permisos insuficientes",
        error: `Requiere rol: ${formatearRoles(rolesPermitidos)}`,
        code: "INSUFFICIENT_PERMISSIONS",
        detalles: {
          rolesRequeridos: rolesPermitidos,
          usuario: req.usuario.username
        }
      });
      return;
    }

    next();
  };
};
```

**Usage in Routes:**

```typescript
// Apply globally to all routes
app.use("/v1/usuarios", autenticarToken, usuariosRouter);

// Apply per-route with role checking
router.post("/", verificarRol("admin"), createUsuario);
router.get("/", verificarRol("admin", "gerente"), getUsuarios);
```

**Many-to-Many Role System:**
- Users can have multiple roles (flexibility)
- Roles cached in request object (performance)
- Database-driven (no hardcoded roles)

### 5. **Input Validation with Zod**

Define schemas in `src/schemas/`:

```typescript
import { z } from "zod";

export const createUsuarioSchema = z.object({
  body: z.object({
    username: z.string()
      .min(3, "Username debe tener al menos 3 caracteres")
      .max(50, "Username no puede exceder 50 caracteres"),
    email: z.string()
      .email("Email inválido")
      .max(100),
    password: z.string()
      .min(8, "Contraseña debe tener al menos 8 caracteres")
      .max(100),
    rol_id: z.number().int().positive().optional(),
    mustChangePassword: z.boolean().optional()
  }),
  params: z.object({}),
  query: z.object({})
});

export type CreateUsuarioInput = z.infer<typeof createUsuarioSchema>;
```

**Validation Middleware:**

```typescript
const validate = (schema: ZodObject<any, any>) => {
  return async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const result = await schema.safeParseAsync({
        body: req.body,
        query: req.query,
        params: req.params
      });

      if (!result.success) {
        const formattedErrors = result.error.issues.map(error => ({
          field: error.path[1] || error.path[0],
          message: error.message
        }));

        res.status(400).json({
          status: "error",
          message: "Error de validación",
          errors: formattedErrors
        });
        return;
      }

      // Replace with validated data
      if (result.data.body) req.body = result.data.body;
      if (result.data.query) req.query = result.data.query as any;
      if (result.data.params) req.params = result.data.params as any;

      next();
    } catch (error) {
      res.status(500).json({
        status: "error",
        message: "Error interno en validación"
      });
    }
  };
};
```

**Usage:**

```typescript
router.post("/", validate(createUsuarioSchema), createUsuario);
```

### 6. **Dynamic Data Inclusion Pattern**

Support flexible data loading via query parameters:

```typescript
// Controller: Parse query params
function construirInclusions(include: string) {
  return String(include || "")
    .split(",")
    .map(s => s.trim())
    .filter(Boolean);
}

export const getUsuarios = async (req: Request, res: Response) => {
  const includeList = construirInclusions(req.query.include as string);
  const usuarios = await obtenerUsuarios({ include: includeList });
  res.json({ status: "ok", data: usuarios });
};

// Service: Build Sequelize includes
type Opts = {
  include?: string[];
}

const construirInclusions = (opts: Opts = {}): any[] => {
  const include: any[] = [];

  if (opts.include?.includes("roles")) {
    include.push({
      model: Rol,
      as: "roles",
      through: { attributes: [] }  // Hide join table
    });
  }

  if (opts.include?.includes("almacen")) {
    include.push({ model: Almacen, as: "almacen" });
  }

  return include;
};

export const obtenerUsuarios = async (opts: Opts = {}) => {
  const include = construirInclusions(opts);
  return await Usuario.findAll({ include });
};
```

**Usage:**
```
GET /v1/usuarios?include=roles
GET /v1/detalle_ventas?include=producto,almacen
```

### 7. **OpenAPI/Swagger Documentation**

Configure comprehensive API documentation:

```typescript
// src/config/swagger.ts
const swaggerDefinition = {
  openapi: '3.0.0',
  info: {
    title: 'API Sistema',
    version: '1.0.0',
    description: 'REST API con autenticación JWT y RBAC'
  },
  servers: [
    { url: `http://${SERVER_HOST}:${SERVER_PORT}`, description: 'Desarrollo' }
  ],
  components: {
    securitySchemes: {
      BearerAuth: {
        type: 'http',
        scheme: 'bearer',
        bearerFormat: 'JWT'
      }
    },
    schemas: {
      SuccessResponse: {
        type: 'object',
        properties: {
          status: { type: 'string', enum: ['ok'] },
          message: { type: 'string' },
          data: { type: 'object' }
        }
      },
      ErrorResponse: {
        type: 'object',
        properties: {
          status: { type: 'string', enum: ['error'] },
          message: { type: 'string' },
          error: { type: 'string' },
          code: { type: 'string' }
        }
      }
    }
  },
  paths: {
    '/v1/usuarios': {
      get: {
        tags: ['Usuarios'],
        security: [{ BearerAuth: [] }],
        responses: {
          200: { description: 'Success' },
          401: { $ref: '#/components/responses/UnauthorizedError' }
        }
      }
    }
  }
};
```

**Mount in Express:**

```typescript
import swaggerUi from "swagger-ui-express";
import { swaggerSpec } from "./config/swagger";

app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(swaggerSpec, {
  customSiteTitle: "API Documentation",
  customCss: '.swagger-ui .topbar { display: none }'
}));
```

### 8. **Security Best Practices**

**Password Hashing (Argon2):**

```typescript
import argon2 from "argon2";

// Hashing
const hashedPassword = await argon2.hash(password);

// Verification
const isValid = await argon2.verify(hashedPassword, password);
```

**Environment Variables:**

```typescript
// src/config/config.ts
import dotenv from "dotenv";
dotenv.config();

export const SERVER_HOST = process.env.SERVER_HOST || "localhost";
export const SERVER_PORT = Number(process.env.SERVER_PORT) || 3000;
export const DB_HOST = process.env.DB_HOST || "localhost";
export const DB_USER = process.env.DB_USER!;
export const DB_PASS = process.env.DB_PASS!;
export const DB_NAME = process.env.DB_NAME!;
```

**JWT Configuration:**

```typescript
// src/config/jwt.ts
import jwt from "jsonwebtoken";

const SECRET_KEY = process.env.JWT_SECRET || "your-secret-key";

export const generarToken = (payload: any): string => {
  return jwt.sign(payload, SECRET_KEY, { expiresIn: "1h" });
};

export const verificarToken = (token: string): any => {
  return jwt.verify(token, SECRET_KEY);
};
```

### 9. **Project Structure**

```
src/
├── config/
│   ├── db.ts              # Database connection
│   ├── config.ts          # Environment variables
│   ├── jwt.ts             # JWT utilities
│   └── swagger.ts         # OpenAPI definition
├── models/
│   ├── usuarios/
│   │   ├── usuarios.ts
│   │   ├── roles.ts
│   │   ├── usuarios_roles.ts
│   │   └── associations.ts  # Model relationships
│   ├── productos/
│   └── ventas/
├── controllers/
│   ├── usuarios/
│   │   └── usuarios.controller.ts
│   └── ventas/
├── services/
│   ├── usuarios/
│   │   ├── usuarios.service.ts
│   │   └── roles.service.ts
│   └── ventas/
├── middlewares/
│   ├── auth.middleware.ts     # autenticarToken, verificarRol
│   └── validateResource.ts    # Zod validation
├── schemas/
│   ├── usuarios.schema.ts
│   └── ventas.schema.ts
├── v1/
│   └── routes/
│       ├── usuarios/
│       │   └── usuarios.route.ts
│       └── ventas/
├── scripts/
│   └── seed-admin.ts          # Database seeding
└── index.ts                   # App entry point
```

### 10. **Database Patterns**

**Model Associations (Sequelize):**

```typescript
// src/models/usuarios/associations.ts
import Usuario from "./usuarios";
import Rol from "./roles";
import UsuarioRol from "./usuarios_roles";

// Many-to-Many
Usuario.belongsToMany(Rol, {
  through: UsuarioRol,
  foreignKey: "usuario_id",
  otherKey: "rol_id",
  as: "roles"
});

Rol.belongsToMany(Usuario, {
  through: UsuarioRol,
  foreignKey: "rol_id",
  otherKey: "usuario_id",
  as: "usuarios"
});

// One-to-Many for join table
UsuarioRol.belongsTo(Usuario, { foreignKey: "usuario_id", as: "usuario" });
UsuarioRol.belongsTo(Rol, { foreignKey: "rol_id", as: "rol" });
```

**Database Initialization:**

```typescript
// src/index.ts
app.listen(PORT, HOST, async () => {
  try {
    await sequelize.authenticate();
    console.log("Database connected");

    // sync({ force: false }) preserves data
    // sync({ force: true }) drops and recreates (DANGER!)
    await sequelize.sync({ force: false });
    console.log("Models synchronized");
  } catch (error) {
    console.error("Database connection failed:", error);
  }
});
```

## SOLID Principles Application

### **S - Single Responsibility Principle**
- Each layer has ONE responsibility
- Controllers: HTTP handling
- Services: Business logic
- Models: Data structure

### **O - Open/Closed Principle**
- Middleware composition allows extension without modification
- Dynamic includes support new relations without changing core logic

### **L - Liskov Substitution Principle**
- Consistent response format allows interchangeable endpoints
- Middleware can be swapped without breaking the chain

### **I - Interface Segregation Principle**
- Zod schemas define minimal required fields
- Optional `include` parameter avoids forcing unnecessary data

### **D - Dependency Inversion Principle**
- Controllers depend on service abstractions, not concrete implementations
- Services use ORM models (abstraction) rather than direct SQL

## Quick Start Checklist

When creating a new REST API endpoint:

1. ✅ Define Zod schema in `src/schemas/`
2. ✅ Create service function in `src/services/` with business logic
3. ✅ Use transactions for multi-step operations
4. ✅ Create controller in `src/controllers/` that calls service
5. ✅ Return standardized response format
6. ✅ Define route in `src/v1/routes/`
7. ✅ Apply `autenticarToken` middleware if protected
8. ✅ Apply `verificarRol` middleware if role-restricted
9. ✅ Apply `validate(schema)` middleware for input validation
10. ✅ Document in Swagger (`src/config/swagger.ts`)

## Common Patterns

For detailed implementations and code examples, see:
- [Architecture Deep Dive](references/architecture.md) - Detailed layer explanations
- [Transaction Patterns](references/transactions.md) - Complex transaction scenarios
- [Authentication & RBAC](references/auth-rbac.md) - Complete auth implementation
- [Validation Strategies](references/validation.md) - Zod patterns and custom validators
- [Error Handling](references/error-handling.md) - Comprehensive error management
- [Testing Strategies](references/testing.md) - Unit and integration test patterns
- [Performance Optimization](references/performance.md) - Query optimization, caching

## Anti-Patterns to Avoid

❌ **DON'T:**
- Put business logic in controllers
- Make database calls from controllers
- Forget transactions for multi-step operations
- Hardcode role names in code (use database)
- Return raw error messages to clients
- Skip input validation
- Use `sync({ force: true })` in production
- Expose sensitive data in JWT payload
- Use `findAll()` without pagination (for large datasets)

✅ **DO:**
- Keep controllers thin (delegate to services)
- Wrap related operations in transactions
- Validate all input with Zod
- Return standardized response format
- Hash passwords with Argon2
- Cache user roles to avoid repeated DB queries
- Use environment variables for configuration
- Document all endpoints in Swagger

---

**Skill Version:** 1.0.0
**Author:** Miller Marru ([@MILLERMARRU](https://github.com/MILLERMARRU))
**Contact:** millermarru4@gmail.com
**Repository:** https://github.com/MILLERMARRU/express-typescript-api-best-practices
**License:** MIT
**Last Updated:** 2026-02-05
