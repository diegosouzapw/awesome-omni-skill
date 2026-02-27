---
name: custom-plugin-flutter-skill-backend
description: 1500+ lines of backend integration mastery - REST APIs, GraphQL, WebSockets, authentication, Firebase, error handling with production-ready code examples.
sasmp_version: "1.3.0"
bonded_agent: 01-flutter-ui-development
bond_type: PRIMARY_BOND
---

# custom-plugin-flutter: Backend Integration Skill

## Quick Start - Dio Client

```dart
final dio = Dio(BaseOptions(
  baseUrl: 'https://api.example.com',
  connectTimeout: Duration(seconds: 10),
  receiveTimeout: Duration(seconds: 10),
));

// Add logging interceptor
dio.interceptors.add(LoggingInterceptor());

// Make request
Future<User> getUser(String id) async {
  try {
    final response = await dio.get('/users/$id');
    return User.fromJson(response.data);
  } on DioException catch (e) {
    throw ApiException(e.message ?? 'Unknown error');
  }
}
```

## 1. HTTP Clients

**HTTP Package (Lightweight):**
```dart
import 'package:http/http.dart' as http;

Future<User> getUser(String id) async {
  final response = await http.get(
    Uri.parse('https://api.example.com/users/$id'),
    headers: {'Authorization': 'Bearer $token'},
  );

  if (response.statusCode == 200) {
    return User.fromJson(jsonDecode(response.body));
  } else {
    throw Exception('Failed to load user');
  }
}
```

**Dio Framework (Production):**
```dart
final dio = Dio(BaseOptions(
  baseUrl: 'https://api.example.com',
  connectTimeout: Duration(seconds: 10),
  receiveTimeout: Duration(seconds: 10),
));

// Add interceptors
dio.interceptors.add(
  InterceptorsWrapper(
    onRequest: (options, handler) async {
      options.headers['Authorization'] = 'Bearer $token';
      return handler.next(options);
    },
    onError: (error, handler) async {
      if (error.response?.statusCode == 401) {
        await _refreshToken();
      }
      return handler.next(error);
    },
  ),
);

// Use Dio
final response = await dio.get('/users/123');
```

## 2. REST API Patterns

**Repository Pattern:**
```dart
abstract class UserRepository {
  Future<User> getUser(String id);
  Future<List<User>> getAllUsers();
  Future<void> createUser(User user);
}

class UserRepositoryImpl implements UserRepository {
  final Dio _dio;

  UserRepositoryImpl(this._dio);

  @override
  Future<User> getUser(String id) async {
    try {
      final response = await _dio.get('/users/$id');
      return User.fromJson(response.data);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  @override
  Future<List<User>> getAllUsers() async {
    try {
      final response = await _dio.get('/users');
      return (response.data as List)
          .map((u) => User.fromJson(u))
          .toList();
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  Exception _handleError(DioException e) {
    if (e.type == DioExceptionType.connectionTimeout) {
      return TimeoutException('Connection timeout');
    } else if (e.response?.statusCode == 401) {
      return UnauthorizedException('Unauthorized');
    } else if (e.response?.statusCode == 500) {
      return ServerException('Server error');
    }
    return ApiException('Unknown error');
  }
}
```

## 3. GraphQL Integration

```dart
import 'package:graphql/client.dart';

final graphQLClient = GraphQLClient(
  cache: GraphQLCache(),
  link: HttpLink('https://api.example.com/graphql'),
);

// Query
Future<User> getUser(String id) async {
  final result = await graphQLClient.query(
    QueryOptions(
      document: gql('''
        query GetUser(\$id: ID!) {
          user(id: \$id) {
            id
            name
            email
          }
        }
      '''),
      variables: {'id': id},
    ),
  );

  if (result.hasException) {
    throw Exception(result.exception);
  }

  return User.fromJson(result.data!['user']);
}

// Mutation
Future<void> updateUser(User user) async {
  final result = await graphQLClient.mutate(
    MutationOptions(
      document: gql('''
        mutation UpdateUser(\$id: ID!, \$name: String!) {
          updateUser(id: \$id, name: \$name) {
            id
            name
          }
        }
      '''),
      variables: {'id': user.id, 'name': user.name},
    ),
  );

  if (result.hasException) {
    throw Exception(result.exception);
  }
}
```

## 4. Authentication

**JWT Token Management:**
```dart
class AuthService {
  final Dio _dio;
  String? _accessToken;
  String? _refreshToken;

  AuthService(this._dio);

  Future<void> login(String email, String password) async {
    try {
      final response = await _dio.post('/auth/login', data: {
        'email': email,
        'password': password,
      });

      _accessToken = response.data['accessToken'];
      _refreshToken = response.data['refreshToken'];

      await _secureStorage.write(
        key: 'accessToken',
        value: _accessToken!,
      );
      await _secureStorage.write(
        key: 'refreshToken',
        value: _refreshToken!,
      );
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  Future<String> getValidToken() async {
    if (_isTokenExpired(_accessToken)) {
      await _refreshAccessToken();
    }
    return _accessToken!;
  }

  Future<void> _refreshAccessToken() async {
    try {
      final response = await _dio.post(
        '/auth/refresh',
        data: {'refreshToken': _refreshToken},
      );

      _accessToken = response.data['accessToken'];
      await _secureStorage.write(
        key: 'accessToken',
        value: _accessToken!,
      );
    } catch (e) {
      await logout();
      rethrow;
    }
  }

  bool _isTokenExpired(String? token) {
    if (token == null) return true;
    final decoded = jwt.decode(token);
    final expiration = DateTime.fromMillisecondsSinceEpoch(
      decoded['exp'] * 1000,
    );
    return DateTime.now().isAfter(expiration);
  }

  Future<void> logout() async {
    _accessToken = null;
    _refreshToken = null;
    await _secureStorage.deleteAll();
  }
}
```

## 5. Error Handling

**Custom Exceptions:**
```dart
abstract class ApiException implements Exception {
  final String message;
  ApiException(this.message);
}

class NetworkException extends ApiException {
  NetworkException(String message) : super('Network Error: $message');
}

class UnauthorizedException extends ApiException {
  UnauthorizedException(String message) : super('Unauthorized: $message');
}

class ServerException extends ApiException {
  ServerException(String message) : super('Server Error: $message');
}

class ValidationException extends ApiException {
  final Map<String, String> errors;
  ValidationException(this.errors) : super('Validation Error');
}
```

## 6. Firebase Integration

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

class FirebaseUserRepository {
  final firestore = FirebaseFirestore.instance;

  Future<User?> getUser(String id) async {
    try {
      final doc = await firestore.collection('users').doc(id).get();
      if (!doc.exists) return null;
      return User.fromJson(doc.data()!);
    } catch (e) {
      throw Exception('Failed to get user: $e');
    }
  }

  Stream<User?> watchUser(String id) {
    return firestore
        .collection('users')
        .doc(id)
        .snapshots()
        .map((doc) => doc.exists ? User.fromJson(doc.data()!) : null);
  }

  Future<void> updateUser(User user) async {
    await firestore.collection('users').doc(user.id).update(
      user.toJson(),
    );
  }

  Future<void> createUser(User user) async {
    await firestore.collection('users').doc(user.id).set(
      user.toJson(),
    );
  }
}
```

## 7. Retry Logic

```dart
Future<T> withRetry<T>(
  Future<T> Function() operation, {
  int maxRetries = 3,
  Duration initialDelay = const Duration(milliseconds: 100),
}) async {
  int retries = 0;
  Duration delay = initialDelay;

  while (true) {
    try {
      return await operation();
    } catch (e) {
      if (retries >= maxRetries || !_isRetryable(e)) {
        rethrow;
      }

      await Future.delayed(delay);
      delay = Duration(milliseconds: (delay.inMilliseconds * 2).toInt());
      retries++;
    }
  }
}

bool _isRetryable(dynamic error) {
  if (error is! DioException) return false;
  return error.type == DioExceptionType.connectionTimeout ||
      error.type == DioExceptionType.receiveTimeout ||
      (error.response?.statusCode ?? 0) >= 500;
}
```

---

**Master backend integration for robust Flutter applications.**
