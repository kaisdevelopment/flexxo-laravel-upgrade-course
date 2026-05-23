# 🚀 Aula 02 — Speed Run: Upgrade Laravel 9 → 10 → 11 → 12

## Módulo: Laravel Avançado | Flexxo - Polo Caxias do Sul
**Data:** 23 de Maio de 2026
**Instrutor:** Wiliam
**Aluno:** Juares (Mentoria 1:1)
**Modalidade:** Remoto

---

## 📌 Pré-requisito

- ✅ Upgrade Laravel 8 → 9 concluído (Aula 01)
- ✅ PHP 8.3 instalado
- ✅ Composer 2.x instalado

---

# 🏁 SALTO 1 — Laravel 9 → 10

**Tempo estimado:** ~10 minutos
**PHP mínimo:** 8.1 (✅ temos 8.3)

## O que muda de importante:

- PHP 8.1 obrigatório
- Composer 2.2 obrigatório
- `spatie/laravel-ignition` sobe de `^1.0` para `^2.0`
- `doctrine/dbal` sobe para `^3.0` (se estiver usando)
- Monolog 3 (mudança nos handlers de log)
- Property `$dates` dos Models foi **deprecated** → usar `$casts` ao invés

---

### Passo 1.1 — Editar `composer.json`

Alterar as seguintes versões:

```json
{
    "require": {
        "laravel/framework": "^10.0",
        "spatie/laravel-ignition": "^2.0"
    },
    "require-dev": {
        "nunomaduro/collision": "^7.0",
        "phpunit/phpunit": "^10.0"
    }
}
```

> **⚠️ Atenção:** Se tiver `laravel/sanctum`, atualize para `^3.2`
> Se tiver `doctrine/dbal`, atualize para `^3.0`

### Passo 1.2 — Executar o upgrade

```bash
composer update -W
```

### Passo 1.3 — Verificar a versão

```bash
php artisan --version
# Esperado: Laravel Framework 10.x.x
```

### Passo 1.4 — Mudança nos Models (se aplicável)

Se algum Model usa a propriedade `$dates` (deprecated), migrar para `$casts`:

```php
// ❌ ANTES (Laravel 9 - deprecated)
protected $dates = ['ultimo_heartbeat', 'created_at'];

// ✅ DEPOIS (Laravel 10+)
protected $casts = [
    'ultimo_heartbeat' => 'datetime',
    'created_at' => 'datetime',
];
```

### Passo 1.5 — Minimum Stability

Verificar no `composer.json` se o campo `minimum-stability` está como `stable`:

```json
"minimum-stability": "stable"
```

### ✅ Checklist Salto 9→10

- [ ] `composer.json` editado
- [ ] `composer update -W` sem erros
- [ ] `php artisan --version` mostra 10.x
- [ ] Propriedade `$dates` substituída por `$casts` (se existia)
- [ ] Aplicação roda (`php artisan serve` funciona)

---

# 🏁 SALTO 2 — Laravel 10 → 11

**Tempo estimado:** ~15 minutos
**PHP mínimo:** 8.2 (✅ temos 8.3)

## O que muda de importante:

- PHP 8.2 obrigatório
- **Nova estrutura de aplicação** (mais enxuta, menos arquivos)
- `doctrine/dbal` foi **removido** do core (usar migrations nativas)
- Sanctum sobe para `^4.0`
- SQLite 3.35+ obrigatório
- `$casts` vira **método** ao invés de propriedade (opcional, mas recomendado)
- Carbon 3 suportado

> **💡 Nota Importante:** O Laravel 11 mudou drasticamente a estrutura de
> diretórios padrão (removeu `Http/Kernel.php`, `Providers/`, etc.).
> **MAS** — projetos que fazem upgrade NÃO precisam adotar a nova estrutura!
> O Laravel 11 é retrocompatível com a estrutura antiga.

---

### Passo 2.1 — Editar `composer.json`

```json
{
    "require": {
        "laravel/framework": "^11.0"
    },
    "require-dev": {
        "nunomaduro/collision": "^8.1"
    }
}
```

> **Se tiver estes pacotes, atualize também:**

```json
"laravel/sanctum": "^4.0",
"laravel/breeze": "^2.0",
"laravel/dusk": "^8.0",
"laravel/passport": "^12.0",
"laravel/telescope": "^5.0"
```

### Passo 2.2 — Remover doctrine/dbal (se existir)

```bash
# Remover — Laravel 11 não depende mais dele
composer remove doctrine/dbal --no-update
```

### Passo 2.3 — Executar o upgrade

```bash
composer update -W
```

### Passo 2.4 — Verificar a versão

```bash
php artisan --version
# Esperado: Laravel Framework 11.x.x
```

### Passo 2.5 — (Opcional mas recomendado) Migrar `$casts` para método

O Laravel 11+ recomenda que `$casts` seja um **método** ao invés de propriedade.
Isso permite casts dinâmicos e casts encriptados.

```php
// ❌ ANTES (Laravel 10 - funciona mas é o jeito antigo)
protected $casts = [
    'ultimo_heartbeat' => 'datetime',
    'metadata' => 'array',
];

// ✅ DEPOIS (Laravel 11+ - jeito moderno)
protected function casts(): array
{
    return [
        'ultimo_heartbeat' => 'datetime',
        'metadata' => 'array',
    ];
}
```

### ✅ Checklist Salto 10→11

- [ ] `composer.json` editado
- [ ] `doctrine/dbal` removido (se existia)
- [ ] `composer update -W` sem erros
- [ ] `php artisan --version` mostra 11.x
- [ ] `$casts` migrado para método (opcional)
- [ ] Aplicação roda (`php artisan serve` funciona)

---

# 🏁 SALTO 3 — Laravel 11 → 12

**Tempo estimado:** ~5 minutos
**PHP mínimo:** 8.2 (✅ temos 8.3)

## O que muda de importante:

- Este é o **salto mais suave** de todos
- Carbon 2 removido (somente Carbon 3)
- UUIDv7 para Models (se usar UUIDs)
- Validação de imagem agora exclui SVGs
- Mix totalmente removido (somente Vite)

---

### Passo 3.1 — Editar `composer.json`

```json
{
    "require": {
        "laravel/framework": "^12.0"
    },
    "require-dev": {
        "phpunit/phpunit": "^11.0"
    }
}
```

### Passo 3.2 — Executar o upgrade

```bash
composer update -W
```

### Passo 3.3 — Verificar a versão

```bash
php artisan --version
# Esperado: Laravel Framework 12.x.x 🎉
```

### ✅ Checklist Salto 11→12

- [ ] `composer.json` editado
- [ ] `composer update -W` sem erros
- [ ] `php artisan --version` mostra 12.x
- [ ] Aplicação roda (`php artisan serve` funciona)
- [ ] 🎉 **UPGRADE COMPLETO!**

---

# 📊 Resumo Visual do Speed Run

```
Laravel 8 ──(Aula 01)──► Laravel 9 ──(hoje)──► Laravel 10 ──► Laravel 11 ──► Laravel 12
   │                        │                      │               │              │
   │ facade/ignition        │ spatie ^1.0           │ spatie ^2.0   │ casts()     │ 🎉
   │ PHP 7.3+               │ PHP 8.0+              │ PHP 8.1+      │ PHP 8.2+    │ PHP 8.2+
   │ ~30 min                │ ✅ FEITO              │ ~10 min       │ ~15 min     │ ~5 min
```

---

# 🧪 Comandos Rápidos para Cada Salto

## Referência: copie e cole bloco a bloco

### Salto 9→10 (rodar na ordem)

```bash
# 1. Editar composer.json (use sed ou nano)
sed -i 's/"laravel\/framework": "\^9.0"/"laravel\/framework": "\^10.0"/' composer.json
sed -i 's/"spatie\/laravel-ignition": "\^1.0"/"spatie\/laravel-ignition": "\^2.0"/' composer.json

# 2. Rodar update
composer update -W

# 3. Confirmar
php artisan --version
```

### Salto 10→11 (rodar na ordem)

```bash
# 1. Editar composer.json
sed -i 's/"laravel\/framework": "\^10.0"/"laravel\/framework": "\^11.0"/' composer.json

# 2. Remover doctrine/dbal se existir
composer remove doctrine/dbal --no-update 2>/dev/null

# 3. Rodar update
composer update -W

# 4. Confirmar
php artisan --version
```

### Salto 11→12 (rodar na ordem)

```bash
# 1. Editar composer.json
sed -i 's/"laravel\/framework": "\^11.0"/"laravel\/framework": "\^12.0"/' composer.json

# 2. Rodar update
composer update -W

# 3. Confirmar
php artisan --version
```

---

# 🔍 Tabela Comparativa: Evolução entre Versões

| Feature                    | Laravel 9        | Laravel 10       | Laravel 11        | Laravel 12       |
|----------------------------|------------------|------------------|-------------------|------------------|
| **PHP mínimo**             | 8.0              | 8.1              | 8.2               | 8.2              |
| **Ignition**               | spatie ^1.0      | spatie ^2.0      | spatie ^2.0       | spatie ^2.0      |
| **Casts**                  | $casts property | $casts property | casts() method    | casts() method   |
| **doctrine/dbal**          | Incluído         | Incluído         | Removido          | Removido         |
| **Estrutura Padrão**       | Clássica         | Clássica         | Slim (nova)       | Slim (nova)      |
| **Http/Kernel.php**        | Existe           | Existe           | Removido*         | Removido*        |
| **Providers/ pasta**       | Existe           | Existe           | Removido*         | Removido*        |
| **Frontend**               | Mix ou Vite      | Vite preferido   | Vite obrigatório  | Somente Vite     |
| **Carbon**                 | 2.x              | 2.x              | 2.x ou 3.x       | Somente 3.x     |
| **Sanctum**                | ^2.x             | ^3.2             | ^4.0              | ^4.0             |

> \* Projetos que fazem upgrade **mantêm** esses arquivos. Só projetos novos usam a estrutura slim.

---

# ⚠️ Troubleshooting — Erros Comuns em Cada Salto

## 9→10: `spatie/laravel-ignition` conflito

```bash
# Se esqueceu de atualizar o Ignition:
composer require spatie/laravel-ignition:"^2.0" --no-update
composer update -W
```

## 10→11: `doctrine/dbal` bloqueando

```bash
# Remover antes do update:
composer remove doctrine/dbal --no-update
composer update -W
```

## 11→12: Pacote de terceiro preso no `^11.0`

```bash
# Identificar qual pacote está bloqueando:
composer why-not laravel/framework 12.0

# Se for um pacote que já tem versão nova:
composer require pacote/nome:"^nova-versao" --no-update
composer update -W

# Se o pacote ainda não suporta L12 (raro):
# Opção A: Esperar update do mantenedor
# Opção B: Buscar alternativa
# Opção C: composer update -W --ignore-platform-req=ext-* (último recurso)
```

## Genérico: Limpar cache entre saltos

```bash
php artisan config:clear
php artisan cache:clear
php artisan route:clear
php artisan view:clear
composer dump-autoload
```

---

# 🏷️ Informações do Curso

| Item               | Detalhe                                    |
|--------------------|---------------------------------------------|
| **Escola**         | Flexxo — Polo Caxias do Sul                 |
| **Curso**          | Laravel Avançado (Mentoria)                  |
| **Carga Horária**  | 12 horas (3 aulas de 4h)                    |
| **Aula**           | 02 de 03                                     |
| **Formato**        | Mentoria Individual (1:1)                    |
| **Repositório**    | flexxo-laravel-upgrade-course                |

---

*Material de apoio gerado em 23/05/2026 — Flexxo, Polo Caxias do Sul*
*Instrutor: Wiliam | Todos os direitos reservados.*
