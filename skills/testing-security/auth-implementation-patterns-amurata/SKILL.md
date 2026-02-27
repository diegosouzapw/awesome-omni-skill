---
name: auth-implementation-patterns
description: JWT、OAuth2、セッション管理、RBACを含む認証・認可パターンをマスターし、安全でスケーラブルなアクセス制御システムを構築。認証システムの実装、APIの保護、セキュリティ問題のデバッグ時に使用。
---

> **[English](../../../../../../plugins/developer-essentials/skills/auth-implementation-patterns/SKILL.md)** | **日本語**

# 認証・認可実装パターン

業界標準のパターンと最新のベストプラクティスを使用して、安全でスケーラブルな認証・認可システムを構築します。

## このスキルを使用するタイミング

- ユーザー認証システムの実装
- REST または GraphQL API の保護
- OAuth2/ソーシャルログインの追加
- ロールベースアクセス制御（RBAC）の実装
- セッション管理の設計
- 認証システムの移行
- 認証問題のデバッグ
- SSO またはマルチテナンシーの実装

## コア概念

### 1. 認証 vs 認可

**認証（Authentication, AuthN）**：あなたは誰ですか？
- 身元の検証（ユーザー名/パスワード、OAuth、生体認証）
- 資格情報の発行（セッション、トークン）
- ログイン/ログアウトの管理

**認可（Authorization, AuthZ）**：あなたは何ができますか？
- 権限チェック
- ロールベースアクセス制御（RBAC）
- リソース所有権の検証
- ポリシー適用

### 2. 認証戦略

**セッションベース：**
- サーバーがセッション状態を保存
- Cookie内のセッションID
- 従来型、シンプル、ステートフル

**トークンベース（JWT）：**
- ステートレス、自己完結型
- 水平スケーリング可能
- クレームを保存可能

**OAuth2/OpenID Connect：**
- 認証の委譲
- ソーシャルログイン（Google、GitHub）
- エンタープライズSSO

## JWT認証

### パターン1：JWT実装

```typescript
// JWT構造：header.payload.signature
import jwt from 'jsonwebtoken';
import { Request, Response, NextFunction } from 'express';

interface JWTPayload {
    userId: string;
    email: string;
    role: string;
    iat: number;
    exp: number;
}

// JWTを生成
function generateTokens(userId: string, email: string, role: string) {
    const accessToken = jwt.sign(
        { userId, email, role },
        process.env.JWT_SECRET!,
        { expiresIn: '15m' }  // 短命
    );

    const refreshToken = jwt.sign(
        { userId },
        process.env.JWT_REFRESH_SECRET!,
        { expiresIn: '7d' }  // 長命
    );

    return { accessToken, refreshToken };
}

// JWTを検証
function verifyToken(token: string): JWTPayload {
    try {
        return jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload;
    } catch (error) {
        if (error instanceof jwt.TokenExpiredError) {
            throw new Error('トークンが期限切れです');
        }
        if (error instanceof jwt.JsonWebTokenError) {
            throw new Error('無効なトークンです');
        }
        throw error;
    }
}

// ミドルウェア
function authenticate(req: Request, res: Response, next: NextFunction) {
    const authHeader = req.headers.authorization;
    if (!authHeader?.startsWith('Bearer ')) {
        return res.status(401).json({ error: 'トークンが提供されていません' });
    }

    const token = authHeader.substring(7);
    try {
        const payload = verifyToken(token);
        req.user = payload;  // リクエストにユーザーを付加
        next();
    } catch (error) {
        return res.status(401).json({ error: '無効なトークンです' });
    }
}

// 使用例
app.get('/api/profile', authenticate, (req, res) => {
    res.json({ user: req.user });
});
```

### パターン2：リフレッシュトークンフロー

```typescript
interface StoredRefreshToken {
    token: string;
    userId: string;
    expiresAt: Date;
    createdAt: Date;
}

class RefreshTokenService {
    // リフレッシュトークンをデータベースに保存
    async storeRefreshToken(userId: string, refreshToken: string) {
        const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000);
        await db.refreshTokens.create({
            token: await hash(refreshToken),  // 保存前にハッシュ化
            userId,
            expiresAt,
        });
    }

    // アクセストークンをリフレッシュ
    async refreshAccessToken(refreshToken: string) {
        // リフレッシュトークンを検証
        let payload;
        try {
            payload = jwt.verify(
                refreshToken,
                process.env.JWT_REFRESH_SECRET!
            ) as { userId: string };
        } catch {
            throw new Error('無効なリフレッシュトークンです');
        }

        // データベースにトークンが存在するか確認
        const storedToken = await db.refreshTokens.findOne({
            where: {
                token: await hash(refreshToken),
                userId: payload.userId,
                expiresAt: { $gt: new Date() },
            },
        });

        if (!storedToken) {
            throw new Error('リフレッシュトークンが見つからないか期限切れです');
        }

        // ユーザーを取得
        const user = await db.users.findById(payload.userId);
        if (!user) {
            throw new Error('ユーザーが見つかりません');
        }

        // 新しいアクセストークンを生成
        const accessToken = jwt.sign(
            { userId: user.id, email: user.email, role: user.role },
            process.env.JWT_SECRET!,
            { expiresIn: '15m' }
        );

        return { accessToken };
    }

    // リフレッシュトークンを無効化（ログアウト）
    async revokeRefreshToken(refreshToken: string) {
        await db.refreshTokens.deleteOne({
            token: await hash(refreshToken),
        });
    }

    // すべてのユーザートークンを無効化（全デバイスからログアウト）
    async revokeAllUserTokens(userId: string) {
        await db.refreshTokens.deleteMany({ userId });
    }
}

// APIエンドポイント
app.post('/api/auth/refresh', async (req, res) => {
    const { refreshToken } = req.body;
    try {
        const { accessToken } = await refreshTokenService
            .refreshAccessToken(refreshToken);
        res.json({ accessToken });
    } catch (error) {
        res.status(401).json({ error: '無効なリフレッシュトークンです' });
    }
});

app.post('/api/auth/logout', authenticate, async (req, res) => {
    const { refreshToken } = req.body;
    await refreshTokenService.revokeRefreshToken(refreshToken);
    res.json({ message: 'ログアウトしました' });
});
```

## セッションベース認証

### パターン1：Express Session

```typescript
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';

// セッションストレージ用にRedisをセットアップ
const redisClient = createClient({
    url: process.env.REDIS_URL,
});
await redisClient.connect();

app.use(
    session({
        store: new RedisStore({ client: redisClient }),
        secret: process.env.SESSION_SECRET!,
        resave: false,
        saveUninitialized: false,
        cookie: {
            secure: process.env.NODE_ENV === 'production',  // HTTPSのみ
            httpOnly: true,  // JavaScriptからのアクセス不可
            maxAge: 24 * 60 * 60 * 1000,  // 24時間
            sameSite: 'strict',  // CSRF保護
        },
    })
);

// ログイン
app.post('/api/auth/login', async (req, res) => {
    const { email, password } = req.body;

    const user = await db.users.findOne({ email });
    if (!user || !(await verifyPassword(password, user.passwordHash))) {
        return res.status(401).json({ error: '無効な認証情報です' });
    }

    // セッションにユーザーを保存
    req.session.userId = user.id;
    req.session.role = user.role;

    res.json({ user: { id: user.id, email: user.email, role: user.role } });
});

// セッションミドルウェア
function requireAuth(req: Request, res: Response, next: NextFunction) {
    if (!req.session.userId) {
        return res.status(401).json({ error: '認証されていません' });
    }
    next();
}

// 保護されたルート
app.get('/api/profile', requireAuth, async (req, res) => {
    const user = await db.users.findById(req.session.userId);
    res.json({ user });
});

// ログアウト
app.post('/api/auth/logout', (req, res) => {
    req.session.destroy((err) => {
        if (err) {
            return res.status(500).json({ error: 'ログアウトに失敗しました' });
        }
        res.clearCookie('connect.sid');
        res.json({ message: 'ログアウトしました' });
    });
});
```

## OAuth2 / ソーシャルログイン

### パターン1：Passport.jsを使用したOAuth2

```typescript
import passport from 'passport';
import { Strategy as GoogleStrategy } from 'passport-google-oauth20';
import { Strategy as GitHubStrategy } from 'passport-github2';

// Google OAuth
passport.use(
    new GoogleStrategy(
        {
            clientID: process.env.GOOGLE_CLIENT_ID!,
            clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
            callbackURL: '/api/auth/google/callback',
        },
        async (accessToken, refreshToken, profile, done) => {
            try {
                // ユーザーを検索または作成
                let user = await db.users.findOne({
                    googleId: profile.id,
                });

                if (!user) {
                    user = await db.users.create({
                        googleId: profile.id,
                        email: profile.emails?.[0]?.value,
                        name: profile.displayName,
                        avatar: profile.photos?.[0]?.value,
                    });
                }

                return done(null, user);
            } catch (error) {
                return done(error, undefined);
            }
        }
    )
);

// ルート
app.get('/api/auth/google', passport.authenticate('google', {
    scope: ['profile', 'email'],
}));

app.get(
    '/api/auth/google/callback',
    passport.authenticate('google', { session: false }),
    (req, res) => {
        // JWTを生成
        const tokens = generateTokens(req.user.id, req.user.email, req.user.role);
        // トークンと共にフロントエンドにリダイレクト
        res.redirect(`${process.env.FRONTEND_URL}/auth/callback?token=${tokens.accessToken}`);
    }
);
```

## 認可パターン

### パターン1：ロールベースアクセス制御（RBAC）

```typescript
enum Role {
    USER = 'user',
    MODERATOR = 'moderator',
    ADMIN = 'admin',
}

const roleHierarchy: Record<Role, Role[]> = {
    [Role.ADMIN]: [Role.ADMIN, Role.MODERATOR, Role.USER],
    [Role.MODERATOR]: [Role.MODERATOR, Role.USER],
    [Role.USER]: [Role.USER],
};

function hasRole(userRole: Role, requiredRole: Role): boolean {
    return roleHierarchy[userRole].includes(requiredRole);
}

// ミドルウェア
function requireRole(...roles: Role[]) {
    return (req: Request, res: Response, next: NextFunction) => {
        if (!req.user) {
            return res.status(401).json({ error: '認証されていません' });
        }

        if (!roles.some(role => hasRole(req.user.role, role))) {
            return res.status(403).json({ error: '権限が不十分です' });
        }

        next();
    };
}

// 使用例
app.delete('/api/users/:id',
    authenticate,
    requireRole(Role.ADMIN),
    async (req, res) => {
        // 管理者のみがユーザーを削除可能
        await db.users.delete(req.params.id);
        res.json({ message: 'ユーザーを削除しました' });
    }
);
```

### パターン2：権限ベースアクセス制御

```typescript
enum Permission {
    READ_USERS = 'read:users',
    WRITE_USERS = 'write:users',
    DELETE_USERS = 'delete:users',
    READ_POSTS = 'read:posts',
    WRITE_POSTS = 'write:posts',
}

const rolePermissions: Record<Role, Permission[]> = {
    [Role.USER]: [Permission.READ_POSTS, Permission.WRITE_POSTS],
    [Role.MODERATOR]: [
        Permission.READ_POSTS,
        Permission.WRITE_POSTS,
        Permission.READ_USERS,
    ],
    [Role.ADMIN]: Object.values(Permission),
};

function hasPermission(userRole: Role, permission: Permission): boolean {
    return rolePermissions[userRole]?.includes(permission) ?? false;
}

function requirePermission(...permissions: Permission[]) {
    return (req: Request, res: Response, next: NextFunction) => {
        if (!req.user) {
            return res.status(401).json({ error: '認証されていません' });
        }

        const hasAllPermissions = permissions.every(permission =>
            hasPermission(req.user.role, permission)
        );

        if (!hasAllPermissions) {
            return res.status(403).json({ error: '権限が不十分です' });
        }

        next();
    };
}

// 使用例
app.get('/api/users',
    authenticate,
    requirePermission(Permission.READ_USERS),
    async (req, res) => {
        const users = await db.users.findAll();
        res.json({ users });
    }
);
```

### パターン3：リソース所有権

```typescript
// ユーザーがリソースを所有しているか確認
async function requireOwnership(
    resourceType: 'post' | 'comment',
    resourceIdParam: string = 'id'
) {
    return async (req: Request, res: Response, next: NextFunction) => {
        if (!req.user) {
            return res.status(401).json({ error: '認証されていません' });
        }

        const resourceId = req.params[resourceIdParam];

        // 管理者はすべてにアクセス可能
        if (req.user.role === Role.ADMIN) {
            return next();
        }

        // 所有権を確認
        let resource;
        if (resourceType === 'post') {
            resource = await db.posts.findById(resourceId);
        } else if (resourceType === 'comment') {
            resource = await db.comments.findById(resourceId);
        }

        if (!resource) {
            return res.status(404).json({ error: 'リソースが見つかりません' });
        }

        if (resource.userId !== req.user.userId) {
            return res.status(403).json({ error: '権限がありません' });
        }

        next();
    };
}

// 使用例
app.put('/api/posts/:id',
    authenticate,
    requireOwnership('post'),
    async (req, res) => {
        // ユーザーは自分の投稿のみ更新可能
        const post = await db.posts.update(req.params.id, req.body);
        res.json({ post });
    }
);
```

## セキュリティベストプラクティス

### パターン1：パスワードセキュリティ

```typescript
import bcrypt from 'bcrypt';
import { z } from 'zod';

// パスワード検証スキーマ
const passwordSchema = z.string()
    .min(12, 'パスワードは12文字以上である必要があります')
    .regex(/[A-Z]/, 'パスワードには大文字を含める必要があります')
    .regex(/[a-z]/, 'パスワードには小文字を含める必要があります')
    .regex(/[0-9]/, 'パスワードには数字を含める必要があります')
    .regex(/[^A-Za-z0-9]/, 'パスワードには特殊文字を含める必要があります');

// パスワードをハッシュ化
async function hashPassword(password: string): Promise<string> {
    const saltRounds = 12;  // 2^12回の反復
    return bcrypt.hash(password, saltRounds);
}

// パスワードを検証
async function verifyPassword(
    password: string,
    hash: string
): Promise<boolean> {
    return bcrypt.compare(password, hash);
}

// パスワード検証付き登録
app.post('/api/auth/register', async (req, res) => {
    try {
        const { email, password } = req.body;

        // パスワードを検証
        passwordSchema.parse(password);

        // ユーザーが存在するか確認
        const existingUser = await db.users.findOne({ email });
        if (existingUser) {
            return res.status(400).json({ error: 'メールアドレスは既に登録されています' });
        }

        // パスワードをハッシュ化
        const passwordHash = await hashPassword(password);

        // ユーザーを作成
        const user = await db.users.create({
            email,
            passwordHash,
        });

        // トークンを生成
        const tokens = generateTokens(user.id, user.email, user.role);

        res.status(201).json({
            user: { id: user.id, email: user.email },
            ...tokens,
        });
    } catch (error) {
        if (error instanceof z.ZodError) {
            return res.status(400).json({ error: error.errors[0].message });
        }
        res.status(500).json({ error: '登録に失敗しました' });
    }
});
```

### パターン2：レート制限

```typescript
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';

// ログインレート制限
const loginLimiter = rateLimit({
    store: new RedisStore({ client: redisClient }),
    windowMs: 15 * 60 * 1000,  // 15分
    max: 5,  // 5回の試行
    message: 'ログイン試行回数が多すぎます。後でもう一度お試しください',
    standardHeaders: true,
    legacyHeaders: false,
});

// APIレート制限
const apiLimiter = rateLimit({
    windowMs: 60 * 1000,  // 1分
    max: 100,  // 1分あたり100リクエスト
    standardHeaders: true,
});

// ルートに適用
app.post('/api/auth/login', loginLimiter, async (req, res) => {
    // ログインロジック
});

app.use('/api/', apiLimiter);
```

## ベストプラクティス

1. **平文パスワードを保存しない**：常にbcrypt/argon2でハッシュ化
2. **HTTPSを使用**：転送中のデータを暗号化
3. **短命のアクセストークン**：最大15〜30分
4. **安全なCookie**：httpOnly、secure、sameSiteフラグ
5. **すべての入力を検証**：メール形式、パスワード強度
6. **認証エンドポイントにレート制限**：ブルートフォース攻撃を防止
7. **CSRF保護を実装**：セッションベース認証用
8. **シークレットを定期的にローテーション**：JWTシークレット、セッションシークレット
9. **セキュリティイベントをログ記録**：ログイン試行、認証失敗
10. **可能な限りMFAを使用**：追加のセキュリティ層

## よくある落とし穴

- **弱いパスワード**：強力なパスワードポリシーを適用
- **localStorageにJWT**：XSSに脆弱、httpOnly Cookieを使用
- **トークン有効期限なし**：トークンは期限切れにすべき
- **クライアント側の認証チェックのみ**：常にサーバー側で検証
- **安全でないパスワードリセット**：有効期限付きの安全なトークンを使用
- **レート制限なし**：ブルートフォース攻撃に脆弱
- **クライアントデータを信頼**：常にサーバーで検証

## リソース

- **references/jwt-best-practices.md**：JWT実装ガイド
- **references/oauth2-flows.md**：OAuth2フロー図と例
- **references/session-security.md**：安全なセッション管理
- **assets/auth-security-checklist.md**：セキュリティレビューチェックリスト
- **assets/password-policy-template.md**：パスワード要件テンプレート
- **scripts/token-validator.ts**：JWT検証ユーティリティ
