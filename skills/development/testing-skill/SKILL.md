---
name: Testing Skill
description: Automatiza pruebas y diagnósticos del sistema SmartK et sin perder tiempo
---

# Testing Skill - SmartKet ERP

## 🎯 Propósito

Esta skill facilita la **creación, ejecución y mantenimiento de pruebas** en SmartKet, integrando los scripts existentes de `/Pruebas` y proporcionando templates para tests automatizados.

## 🧪 Estrategia de Testing

### Pirámide de Testing

```
         /\
        /E2E\         ← Pocos, críticos (Dusk, flow completo)
       /------\
      /  API  \       ← Moderados (Feature tests, endpoints)
     /----------\
    /   UNIT     \    ← Muchos, rápidos (Services, Models)
   /--------------\
```

**Regla**: 70% Unit, 20% API, 10% E2E

### Tipos de Prueba en SmartKet

1. **Unit Tests** - Lógica de Services y Modelos
2. **Feature Tests** - Endpoints API completos
3. **Database Tests** - Migraciones y seeders
4. **Frontend Tests** - Componentes Vue (Vitest)
5. **Integration Tests** - Flujos multi-tenant

---

## 📋 Checklist de Testing

### Antes de Commit
- [ ] Tests de la funcionalidad nueva/modificada pasan
- [ ] Tests existentes no se rompieron
- [ ] Coverage mínimo: 60% en Services

### Antes de Deploy
- [ ] Suite completa ejecutada (`test-all.ps1`)
- [ ] Tests de integración multi-tenant pasados
- [ ] Diagnósticos de API (`api-tester.php`) OK
- [ ] Tests de navegador críticos (login, registro) OK

---

## 🛠️ Scripts de Testing

### 1. Ejecutar Suite Completa
```bash
.\. agent\skills\testing\scripts\test-runner.ps1
```

**Qué hace**: 
- Tests PHPUnit (backend)
- Tests Vitest (frontend)
- Diagnosis de API
- Diagnosis de DB

### 2. Probar Endpoints API
```bash
cd smartket-api
php ..\.agent\skills\testing\scripts\api-tester.php
```

**Qué hace**: Prueba endpoints críticos con datos reales

### 3. Diagnóstico de Base de Datos
```bash
cd smartket-api
php ..\.agent\skills\testing\scripts\db-diagnostics.php
```

**Qué hace**: Verifica esquemas landlord y tenant

---

## 📚 Templates de Tests

### Feature Test Template
Ver: `.agent/skills/testing/examples/feature-test-example.php`

```php
// Test de endpoint API
test('can create product', function () {
    $response = $this->postJson('/api/products', [
        'name' => 'Test Product',
        'price' => 99.99
    ]);
    
    $response->assertStatus(201)
             ->assertJsonStructure(['id', 'name', 'price']);
});
```

### Unit Test Template
Ver: `.agent/skills/testing/examples/unit-test-example.php`

```php
// Test de Service
test('ProductService creates product correctly', function () {
    $service = new ProductService();
    $product = $service->create(['name' => 'Test', 'price' => 50]);
    
    expect($product)->toBeInstanceOf(Product::class)
                    ->and($product->name)->toBe('Test');
});
```

---

## 🚨 Tests Críticos Obligatorios

### Backend
1. **Auth**: Login, logout, token refresh
2. **Multi-Tenant**: Aislamiento de datos
3. **RBAC**: Permisos granulares
4. **Business Logic**: Cálculos de ventas, stock

### Frontend
1. **Login Flow**: Desde landing hasta app
2. **Dashboard**: Carga correcta de datos
3. **Components**: Buttons, modals, forms

---

## 📖 Integración con /Pruebas

Los scripts de `/Pruebas` existentes están integrados:

- `diagnostico-usuario.php` → Verificar usuarios'
- `simular-login.php` → Test manual de auth
- `test-cors.ps1` → Validar CORS
- `README.md` → Documentación de uso

Usar estos para debugging manual cuando los tests automatizados fallen.

---

## 🎓 Cuándo Usar Esta Skill

✅ **Usar cuando**:
- Agregues nueva funcionalidad (TDD)
- Refactorices código existente
- Debuggees error de producción
- Prepares deployment

❌ **No reemplaza**:
- Testing manual de UX
- Code review humano

---

## 💡 Best Practices

1. **Tests rápidos**: Unit tests < 100ms
2. **Tests aislados**: No depender de orden
3. **Data factories**: Usa factories para datos de prueba
4. **Cleanup**: Tests deben limpiar lo que crean
5. **Descriptivos**: Nombre debe describir qué prueba

```php
// ❌ Malo
test('test 1', function () { ... });

// ✅ Bueno
test('customer can purchase product when in stock', function () { ... });
```
