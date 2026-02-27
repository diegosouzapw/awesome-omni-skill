---
name: budget-ui-components
description: Componentes de UI para orçamentos seguindo o padrão de components do Easy Budget.
---

# Componentes de UI para Orçamentos

Esta skill define os componentes Blade específicos para a gestão de orçamentos no Easy Budget, seguindo o padrão de components estabelecido no sistema.

## Estrutura de Components

```
resources/views/components/
├── budget/
│   ├── budget-card.blade.php          # Card resumido de orçamento
│   ├── budget-details.blade.php      # Detalhes completos do orçamento
│   ├── budget-actions.blade.php      # Ações disponíveis para orçamento
│   ├── budget-status.blade.php       # Badge de status do orçamento
│   ├── budget-totals.blade.php       # Totais e valores do orçamento
│   ├── budget-items.blade.php        # Lista de itens/serviços do orçamento
│   ├── budget-filters.blade.php      # Filtros específicos para orçamentos
│   └── budget-quick-stats.blade.php  # Estatísticas rápidas de orçamentos
└── ...
```

## 1. Budget Card Component

Componente para exibição resumida de orçamentos em listas e dashboards.

### Uso Básico

```blade
<x-budget.budget-card :budget="$budget" :showCustomer="true" :showActions="true" />
```

### Parâmetros

| Parâmetro | Tipo | Descrição | Padrão |
|-----------|------|-----------|--------|
| `budget` | `Budget` | Modelo do orçamento | Obrigatório |
| `showCustomer` | `bool` | Exibir informações do cliente | `true` |
| `showActions` | `bool` | Exibir botões de ação | `true` |
| `variant` | `string` | Estilo do card (primary, secondary, etc.) | `primary` |

### Estrutura

```blade
@props([
    'budget',
    'showCustomer' => true,
    'showActions' => true,
    'variant' => 'primary'
])

<div class="card border-0 shadow-sm h-100">
    <div class="card-body">
        <div class="d-flex justify-content-between align-items-start mb-3">
            <div class="d-flex align-items-center gap-3">
                <div class="avatar-circle bg-{{ $variant }} bg-gradient">
                    <i class="bi bi-file-earmark-text text-white"></i>
                </div>
                <div>
                    <h6 class="mb-1 fw-bold">{{ $budget->code }}</h6>
                    <small class="text-muted">{{ $budget->created_at->format('d/m/Y') }}</small>
                </div>
            </div>
            <x-budget.budget-status :budget="$budget" />
        </div>

        @if($showCustomer && $budget->customer)
            <div class="mb-3">
                <small class="text-muted">Cliente:</small>
                <div class="fw-semibold">{{ $budget->customer->display_name }}</div>
            </div>
        @endif

        <div class="d-flex justify-content-between align-items-center">
            <div>
                <small class="text-muted">Valor Total</small>
                <div class="h6 mb-0 fw-bold text-{{ $variant }}">
                    R$ {{ number_format($budget->total, 2, ',', '.') }}
                </div>
            </div>

            @if($showActions)
                <div class="btn-group btn-group-sm" role="group">
                    <a href="{{ route('provider.budgets.show', $budget) }}" class="btn btn-outline-primary">
                        <i class="bi bi-eye"></i> Ver
                    </a>
                    <a href="{{ route('provider.budgets.edit', $budget) }}" class="btn btn-outline-secondary">
                        <i class="bi bi-pencil"></i> Editar
                    </a>
                </div>
            @endif
        </div>
    </div>
</div>
```

## 2. Budget Details Component

Componente para exibição detalhada de informações do orçamento.

### Uso Básico

```blade
<x-budget.budget-details :budget="$budget" :showServices="true" :showTotals="true" />
```

### Parâmetros

| Parâmetro | Tipo | Descrição | Padrão |
|-----------|------|-----------|--------|
| `budget` | `Budget` | Modelo do orçamento | Obrigatório |
| `showServices` | `bool` | Exibir serviços vinculados | `true` |
| `showTotals` | `bool` | Exibir totais detalhados | `true` |
| `collapsible` | `bool` | Permitir colapsar seções | `false` |

### Estrutura

```blade
@props([
    'budget',
    'showServices' => true,
    'showTotals' => true,
    'collapsible' => false
])

<div class="budget-details">
    <!-- Informações Básicas -->
    <div class="row g-3 mb-3">
        <div class="col-md-3">
            <div class="info-item">
                <label class="text-muted small">Código</label>
                <div class="fw-bold">{{ $budget->code }}</div>
            </div>
        </div>
        <div class="col-md-3">
            <div class="info-item">
                <label class="text-muted small">Data</label>
                <div class="fw-bold">{{ $budget->created_at->format('d/m/Y H:i') }}</div>
            </div>
        </div>
        <div class="col-md-3">
            <div class="info-item">
                <label class="text-muted small">Validade</label>
                <div class="fw-bold">{{ $budget->due_date?->format('d/m/Y') ?? 'Indeterminado' }}</div>
            </div>
        </div>
        <div class="col-md-3">
            <div class="info-item">
                <label class="text-muted small">Status</label>
                <x-budget.budget-status :budget="$budget" />
            </div>
        </div>
    </div>

    <!-- Descrição -->
    @if($budget->description)
        <div class="row mb-3">
            <div class="col-12">
                <div class="card border-0">
                    <div class="card-body">
                        <label class="text-muted small">Descrição</label>
                        <p class="mb-0">{{ $budget->description }}</p>
                    </div>
                </div>
            </div>
        </div>
    @endif

    <!-- Serviços -->
    @if($showServices && $budget->services->isNotEmpty())
        <div class="row mb-3">
            <div class="col-12">
                <div class="card border-0">
                    <div class="card-header bg-transparent border-0">
                        <h6 class="mb-0">Serviços Vinculados</h6>
                    </div>
                    <div class="card-body p-0">
                        <div class="table-responsive">
                            <table class="table table-hover mb-0">
                                <thead>
                                    <tr>
                                        <th>Serviço</th>
                                        <th>Categoria</th>
                                        <th class="text-end">Valor</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    @foreach($budget->services as $service)
                                        <tr>
                                            <td>{{ $service->description ?? $service->category->name }}</td>
                                            <td>{{ $service->category->name }}</td>
                                            <td class="text-end fw-bold">
                                                R$ {{ number_format($service->total, 2, ',', '.') }}
                                            </td>
                                        </tr>
                                    @endforeach
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    @endif

    <!-- Totais -->
    @if($showTotals)
        <x-budget.budget-totals :budget="$budget" />
    @endif
</div>
```

## 3. Budget Status Component

Componente para exibição do status do orçamento com cores e ícones apropriados.

### Uso Básico

```blade
<x-budget.budget-status :budget="$budget" :showIcon="true" />
```

### Parâmetros

| Parâmetro | Tipo | Descrição | Padrão |
|-----------|------|-----------|--------|
| `budget` | `Budget` | Modelo do orçamento | Obrigatório |
| `showIcon` | `bool` | Exibir ícone ao lado do status | `true` |
| `size` | `string` | Tamanho do badge (sm, md, lg) | `md` |

### Estrutura

```blade
@props([
    'budget',
    'showIcon' => true,
    'size' => 'md'
])

@php
    $status = $budget->status;
    $metadata = $status->getMetadata();
    $color = $metadata['color'] ?? '#6c757d';
    $icon = $metadata['icon'] ?? 'circle';
    $description = $metadata['description'] ?? $status->value;

    $sizeClass = match($size) {
        'sm' => 'badge-sm',
        'lg' => 'badge-lg',
        default => 'badge-md'
    };
@endphp

<span class="badge modern-badge {{ $sizeClass }}"
      style="background-color: {{ $color }}20; color: {{ $color }}; border: 1px solid {{ $color }}40;">
    @if($showIcon)
        <i class="bi {{ $icon }} me-1"></i>
    @endif
    {{ $description }}
</span>
```

## 4. Budget Totals Component

Componente para exibição detalhada dos totais e valores do orçamento.

### Uso Básico

```blade
<x-budget.budget-totals :budget="$budget" :showBreakdown="true" />
```

### Parâmetros

| Parâmetro | Tipo | Descrição | Padrão |
|-----------|------|-----------|--------|
| `budget` | `Budget` | Modelo do orçamento | Obrigatório |
| `showBreakdown` | `bool` | Exibir detalhamento dos valores | `true` |
| `variant` | `string` | Estilo visual (primary, success, etc.) | `primary` |

### Estrutura

```blade
@props([
    'budget',
    'showBreakdown' => true,
    'variant' => 'primary'
])

<div class="budget-totals">
    @if($showBreakdown)
        <div class="row g-3 mb-3">
            <div class="col-md-4">
                <div class="card border-0">
                    <div class="card-body text-center">
                        <div class="text-muted small">Subtotal</div>
                        <div class="h5 mb-0 fw-bold">
                            R$ {{ number_format($budget->services->sum('total'), 2, ',', '.') }}
                        </div>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="card border-0">
                    <div class="card-body text-center">
                        <div class="text-muted small">Desconto</div>
                        <div class="h5 mb-0 fw-bold text-danger">
                            - R$ {{ number_format($budget->discount, 2, ',', '.') }}
                        </div>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="card border-0 bg-{{ $variant }} bg-gradient text-white">
                    <div class="card-body text-center">
                        <div class="small">Valor Total</div>
                        <div class="h4 mb-0 fw-bold">
                            R$ {{ number_format($budget->total, 2, ',', '.') }}
                        </div>
                    </div>
                </div>
            </div>
        </div>
    @else
        <div class="card border-0 bg-{{ $variant }} bg-gradient text-white">
            <div class="card-body">
                <div class="row align-items-center">
                    <div class="col-md-8">
                        <div class="small">Valor Total do Orçamento</div>
                        <div class="h3 mb-0 fw-bold">
                            R$ {{ number_format($budget->total, 2, ',', '.') }}
                        </div>
                    </div>
                    <div class="col-md-4 text-end">
                        <x-budget.budget-status :budget="$budget" />
                    </div>
                </div>
            </div>
        </div>
    @endif
</div>
```

## 5. Budget Actions Component

Componente para exibição de ações disponíveis para um orçamento.

### Uso Básico

```blade
<x-budget.budget-actions :budget="$budget" :showShare="true" :showDelete="false" />
```

### Parâmetros

| Parâmetro | Tipo | Descrição | Padrão |
|-----------|------|-----------|--------|
| `budget` | `Budget` | Modelo do orçamento | Obrigatório |
| `showShare` | `bool` | Exibir botão de compartilhamento | `true` |
| `showDelete` | `bool` | Exibir botão de exclusão | `false` |
| `showPrint` | `bool` | Exibir botão de impressão | `true` |

### Estrutura

```blade
@props([
    'budget',
    'showShare' => true,
    'showDelete' => false,
    'showPrint' => true
])

<div class="budget-actions btn-group" role="group">
    <!-- Visualizar -->
    <a href="{{ route('provider.budgets.show', $budget) }}"
       class="btn btn-outline-primary"
       title="Visualizar Orçamento">
        <i class="bi bi-eye"></i> Visualizar
    </a>

    <!-- Editar -->
    @if($budget->status->isEditable())
        <a href="{{ route('provider.budgets.edit', $budget) }}"
           class="btn btn-outline-secondary"
           title="Editar Orçamento">
            <i class="bi bi-pencil"></i> Editar
        </a>
    @endif

    <!-- Compartilhar -->
    @if($showShare)
        <button type="button"
                class="btn btn-outline-info"
                data-bs-toggle="modal"
                data-bs-target="#shareModal-{{ $budget->id }}"
                title="Compartilhar Orçamento">
            <i class="bi bi-share"></i> Compartilhar
        </button>
    @endif

    <!-- Imprimir -->
    @if($showPrint)
        <a href="{{ route('provider.budgets.pdf', $budget) }}"
           class="btn btn-outline-warning"
           target="_blank"
           title="Imprimir Orçamento">
            <i class="bi bi-printer"></i> Imprimir
        </a>
    @endif

    <!-- Enviar por E-mail -->
    <button type="button"
            class="btn btn-outline-success"
            data-bs-toggle="modal"
            data-bs-target="#emailModal-{{ $budget->id }}"
            title="Enviar por E-mail">
        <i class="bi bi-envelope"></i> Enviar
    </button>

    <!-- Excluir -->
    @if($showDelete && $budget->canDelete())
        <button type="button"
                class="btn btn-outline-danger"
                data-bs-toggle="modal"
                data-bs-target="#deleteModal-{{ $budget->id }}"
                title="Excluir Orçamento">
            <i class="bi bi-trash"></i> Excluir
        </button>
    @endif
</div>
```

## 6. Budget Filters Component

Componente para filtros específicos de listagem de orçamentos.

### Uso Básico

```blade
<x-budget.budget-filters :filters="$filters" :showDateRange="true" />
```

### Parâmetros

| Parâmetro | Tipo | Descrição | Padrão |
|-----------|------|-----------|--------|
| `filters` | `array` | Filtros atuais | `[]` |
| `showDateRange` | `bool` | Exibir filtro de período | `true` |
| `showStatus` | `bool` | Exibir filtro por status | `true` |
| `showCustomer` | `bool` | Exibir filtro por cliente | `true` |

### Estrutura

```blade
@props([
    'filters' => [],
    'showDateRange' => true,
    'showStatus' => true,
    'showCustomer' => true
])

<div class="budget-filters">
    <form action="{{ request()->url() }}" method="GET" class="row g-3">
        <div class="col-md-3">
            <label class="form-label small fw-bold text-muted text-uppercase">Código</label>
            <input type="text"
                   name="code"
                   class="form-control form-control-sm"
                   value="{{ $filters['code'] ?? '' }}"
                   placeholder="Código do orçamento">
        </div>

        @if($showCustomer)
            <div class="col-md-3">
                <label class="form-label small fw-bold text-muted text-uppercase">Cliente</label>
                <select name="customer_id" class="form-select form-select-sm">
                    <option value="">Todos os clientes</option>
                    @foreach($customers as $customer)
                        <option value="{{ $customer->id }}"
                                {{ ($filters['customer_id'] ?? '') == $customer->id ? 'selected' : '' }}>
                            {{ $customer->display_name }}
                        </option>
                    @endforeach
                </select>
            </div>
        @endif

        @if($showStatus)
            <div class="col-md-3">
                <label class="form-label small fw-bold text-muted text-uppercase">Status</label>
                <select name="status" class="form-select form-select-sm">
                    <option value="">Todos os status</option>
                    @foreach(\App\Enums\BudgetStatus::cases() as $status)
                        @php $metadata = $status->getMetadata(); @endphp
                        <option value="{{ $status->value }}"
                                {{ ($filters['status'] ?? '') == $status->value ? 'selected' : '' }}>
                            {{ $metadata['description'] }}
                        </option>
                    @endforeach
                </select>
            </div>
        @endif

        @if($showDateRange)
            <div class="col-md-3">
                <label class="form-label small fw-bold text-muted text-uppercase">Período</label>
                <div class="input-group input-group-sm">
                    <input type="date"
                           name="start_date"
                           class="form-control"
                           value="{{ $filters['start_date'] ?? '' }}">
                    <span class="input-group-text">até</span>
                    <input type="date"
                           name="end_date"
                           class="form-control"
                           value="{{ $filters['end_date'] ?? '' }}">
                </div>
            </div>
        @endif

        <div class="col-12">
            <div class="d-flex justify-content-between">
                <div class="btn-group" role="group">
                    <button type="submit" class="btn btn-primary btn-sm">
                        <i class="bi bi-search me-2"></i>Filtrar
                    </button>
                    <a href="{{ route('provider.budgets.index') }}" class="btn btn-outline-secondary btn-sm">
                        <i class="bi bi-x-circle me-2"></i>Limpar
                    </a>
                </div>

                <div class="btn-group" role="group">
                    <button type="button" class="btn btn-outline-success btn-sm" data-bs-toggle="modal" data-bs-target="#createModal">
                        <i class="bi bi-plus-circle me-2"></i>Novo Orçamento
                    </button>
                </div>
            </div>
        </div>
    </form>
</div>
```

## 7. Budget Quick Stats Component

Componente para exibição de estatísticas rápidas de orçamentos.

### Uso Básico

```blade
<x-budget.budget-quick-stats :stats="$stats" :showChart="true" />
```

### Parâmetros

| Parâmetro | Tipo | Descrição | Padrão |
|-----------|------|-----------|--------|
| `stats` | `array` | Dados estatísticos | Obrigatório |
| `showChart` | `bool` | Exibir gráfico rápido | `true` |
| `timeRange` | `string` | Período analisado | `month` |

### Estrutura

```blade
@props([
    'stats',
    'showChart' => true,
    'timeRange' => 'month'
])

<div class="budget-quick-stats">
    <div class="row g-3">
        <!-- Total de Orçamentos -->
        <div class="col-md-3">
            <div class="card border-0 shadow-sm">
                <div class="card-body text-center">
                    <div class="avatar-circle bg-primary bg-gradient mb-2 mx-auto">
                        <i class="bi bi-file-earmark-text text-white"></i>
                    </div>
                    <div class="text-muted small">Total de Orçamentos</div>
                    <div class="h4 mb-0 fw-bold">{{ $stats['total'] ?? 0 }}</div>
                </div>
            </div>
        </div>

        <!-- Orçamentos Aprovados -->
        <div class="col-md-3">
            <div class="card border-0 shadow-sm">
                <div class="card-body text-center">
                    <div class="avatar-circle bg-success bg-gradient mb-2 mx-auto">
                        <i class="bi bi-check-circle text-white"></i>
                    </div>
                    <div class="text-muted small">Aprovados</div>
                    <div class="h4 mb-0 fw-bold">{{ $stats['approved'] ?? 0 }}</div>
                </div>
            </div>
        </div>

        <!-- Orçamentos Pendentes -->
        <div class="col-md-3">
            <div class="card border-0 shadow-sm">
                <div class="card-body text-center">
                    <div class="avatar-circle bg-warning bg-gradient mb-2 mx-auto">
                        <i class="bi bi-clock text-white"></i>
                    </div>
                    <div class="text-muted small">Pendentes</div>
                    <div class="h4 mb-0 fw-bold">{{ $stats['pending'] ?? 0 }}</div>
                </div>
            </div>
        </div>

        <!-- Valor Total -->
        <div class="col-md-3">
            <div class="card border-0 shadow-sm">
                <div class="card-body text-center">
                    <div class="avatar-circle bg-info bg-gradient mb-2 mx-auto">
                        <i class="bi bi-currency-dollar text-white"></i>
                    </div>
                    <div class="text-muted small">Valor Total</div>
                    <div class="h4 mb-0 fw-bold">
                        R$ {{ number_format($stats['total_value'] ?? 0, 2, ',', '.') }}
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Gráfico -->
    @if($showChart && isset($stats['chart_data']))
        <div class="row mt-3">
            <div class="col-12">
                <div class="card border-0 shadow-sm">
                    <div class="card-body">
                        <h6 class="card-title">Evolução de Orçamentos</h6>
                        <canvas id="budgetChart" width="400" height="100"></canvas>
                    </div>
                </div>
            </div>
        </div>

        @push('scripts')
        <script>
            document.addEventListener('DOMContentLoaded', function() {
                const ctx = document.getElementById('budgetChart').getContext('2d');
                new Chart(ctx, {
                    type: 'line',
                    data: @json($stats['chart_data']),
                    options: {
                        responsive: true,
                        plugins: {
                            legend: { display: false }
                        }
                    }
                });
            });
        </script>
        @endpush
    @endif
</div>
```

## 8. Integração com Padrões Existentes

### Uso em Views

```blade
{{-- Dashboard --}}
<x-budget.budget-quick-stats :stats="$budgetStats" />

{{-- Listagem --}}
<x-budget.budget-card :budget="$budget" />

{{-- Detalhes --}}
<x-budget.budget-details :budget="$budget" />

{{-- Formulários --}}
<x-budget.budget-actions :budget="$budget" />
```

### Estilos CSS

```css
/* Budget Components Styles */
.modern-badge {
    font-size: 0.75rem;
    font-weight: 600;
    padding: 4px 8px;
    border-radius: 6px;
    border: 1px solid transparent;
}

.avatar-circle {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-right: 10px;
}

.info-item {
    background: #f8f9fa;
    padding: 12px;
    border-radius: 8px;
    border: 1px solid #e9ecef;
}

.budget-actions .btn {
    transition: all 0.2s ease;
}

.budget-actions .btn:hover {
    transform: translateY(-1px);
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
```

## 9. JavaScript Interatividade

### Compartilhamento de Orçamentos

```javascript
// budget-actions.js
document.addEventListener('DOMContentLoaded', function() {
    // Modal de compartilhamento
    const shareModals = document.querySelectorAll('[data-bs-target^="#shareModal"]');
    shareModals.forEach(modal => {
        modal.addEventListener('click', function() {
            const budgetId = this.getAttribute('data-budget-id');
            generateShareLink(budgetId);
        });
    });

    // Geração de link público
    function generateShareLink(budgetId) {
        fetch(`/api/budgets/${budgetId}/share`, {
            method: 'POST',
            headers: {
                'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content
            }
        })
        .then(response => response.json())
        .then(data => {
            const shareUrl = data.share_url;
            // Atualizar modal com URL gerada
            updateShareModal(shareUrl);
        });
    }
});
```

## 10. Validação e Segurança

### Autorização

```php
// BudgetPolicy.php
public function view(User $user, Budget $budget)
{
    return $user->tenant_id === $budget->tenant_id;
}

public function update(User $user, Budget $budget)
{
    return $user->tenant_id === $budget->tenant_id &&
           $budget->status->isEditable();
}

public function delete(User $user, Budget $budget)
{
    return $user->tenant_id === $budget->tenant_id &&
           $budget->status->isDeletable();
}
```

### Validations

```blade
{{-- Budget Card com validação de permissões --}}
@can('view', $budget)
    <x-budget.budget-card :budget="$budget" />
@endcan

{{-- Budget Actions com validação de status --}}
@can('update', $budget)
    @if($budget->status->isEditable())
        <x-budget.budget-actions :budget="$budget" />
    @endif
@endcan
```

Este padrão de components para orçamentos garante consistência visual, reutilização de código e manutenibilidade, seguindo os mesmos princípios estabelecidos nos outros components do sistema.
