# 🏭 Sensor Legacy — Sistema de Monitoramento Industrial

> Projeto didático desenvolvido durante o curso de **Laravel** na escola **Flexxo** (Caxias do Sul/RS).
> Simula o monitoramento de sensores industriais em chão de fábrica, com coleta de pulsos (voltagem e ruído), API REST e dashboard web.

---

## 📚 Sobre o Curso

| Item              | Detalhe                                      |
|-------------------|----------------------------------------------|
| **Escola**        | Flexxo — Polo Caxias do Sul                  |
| **Formato**       | Mentoria 1:1                                 |
| **Carga horária** | 12 horas (3 aulas de 4h)                     |
| **Framework**     | Laravel 11.x                                 |
| **PHP**           | 8.2+                                         |
| **Banco**         | SQLite (dev) / MySQL (produção)              |
| **Status**        | ✅ **CONCLUÍDO**                              |

---

## 🗺️ Visão Geral das Aulas

```
AULA 1 ✅                 AULA 2 ✅                   AULA 3 ✅
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Fundamentos Laravel       Dados & Simulação           API & Dashboard
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• Instalação              • Migrations                • Form Requests
• Estrutura MVC           • Models & Relations        • API Resources (JSON)
• Artisan básico          • Seeders & Factories       • Resource Controllers
• Configuração .env       • Comando Artisan Custom    • Dashboard Blade
                          • Simulação de Pulsos       • Filtros & Queries
                          • Progress Bar + Resumo     • Revisão & Boas Práticas
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        BANCO                   LÓGICA                    ENTREGA
```

---

## 🧱 Estrutura do Banco de Dados

### Tabelas

```
maquinas
├── id
├── nome
├── setor
├── created_at
└── updated_at

sensores
├── id
├── maquina_id (FK → maquinas.id)
├── identificador
├── porta
├── created_at
└── updated_at

pulsos
├── id
├── maquina_id (FK → maquinas.id)
├── voltagem
├── ruido
├── registrado_em
├── created_at
└── updated_at
```

### Relacionamentos

```
Maquina  ──hasOne──▶  Sensor
Maquina  ──hasMany──▶ Pulso
Sensor   ──belongsTo──▶ Maquina
Pulso    ──belongsTo──▶ Maquina
```

---

## ✅ Aula 1 — Fundamentos Laravel

### 🎯 Objetivo

> Instalar o Laravel, entender a estrutura MVC e se familiarizar com o ecossistema e o Artisan CLI.

### O que foi feito

- [x] Instalação do Laravel via Composer
- [x] Configuração do ambiente (`.env`, SQLite)
- [x] Exploração da estrutura MVC
- [x] Primeiros comandos Artisan (`serve`, `make:model`, `make:migration`)
- [x] Conceito de rotas, controllers e views
- [x] Entendimento do ciclo de vida de uma request no Laravel

### Conceitos-chave

| Conceito           | Descrição                                                         |
|--------------------|-------------------------------------------------------------------|
| **MVC**            | Model-View-Controller — separação de responsabilidades            |
| **Artisan**        | CLI nativo do Laravel para automação de tarefas                   |
| **Composer**       | Gerenciador de dependências PHP                                   |
| **`.env`**       | Arquivo de variáveis de ambiente (nunca comitar!)                 |
| **Service Container** | Motor de injeção de dependências do Laravel                    |
| **Routes**         | Mapeamento URL → Controller → Response                            |

### 🔧 Analogia Industrial — Aula 1

```
┌─────────────────────────────────────────────────────┐
│                CHÃO DE FÁBRICA                      │
│                                                     │
│  Composer    = Almoxarifado (fornece as peças)      │
│  Laravel     = Máquina principal (framework)        │
│  Artisan     = Painel de controle do operador       │
│  .env        = Chave do quadro elétrico             │
│  MVC         = Separar quem opera, processa, exibe  │
│  Routes      = Esteira → direciona cada peça        │
└─────────────────────────────────────────────────────┘
```

---

## ✅ Aula 2 — Dados & Simulação

### 🎯 Objetivo

> Criar a camada de dados completa e simular o comportamento real de sensores industriais via terminal.

### O que foi feito

- [x] Criação das Migrations (`maquinas`, `sensores`, `pulsos`)
- [x] Criação dos Models com relacionamentos Eloquent
- [x] Seeders para popular máquinas de chão de fábrica
- [x] Factories para geração de dados realistas
- [x] Comando Artisan customizado `simular:pulsos`
- [x] Flags `--quantidade` e `--intervalo`
- [x] Progress bar visual no terminal
- [x] Log colorido por porta/sensor
- [x] Tabela de resumo final com métricas agregadas

### Migrations

```php
// create_maquinas_table
Schema::create('maquinas', function (Blueprint $table) {
    $table->id();
    $table->string('nome');
    $table->string('setor');
    $table->timestamps();
});

// create_sensores_table
Schema::create('sensores', function (Blueprint $table) {
    $table->id();
    $table->foreignId('maquina_id')->constrained('maquinas')->onDelete('cascade');
    $table->string('identificador');
    $table->string('porta');
    $table->timestamps();
});

// create_pulsos_table
Schema::create('pulsos', function (Blueprint $table) {
    $table->id();
    $table->foreignId('maquina_id')->constrained('maquinas')->onDelete('cascade');
    $table->decimal('voltagem', 8, 2);
    $table->decimal('ruido', 8, 2);
    $table->timestamp('registrado_em')->useCurrent();
    $table->timestamps();
});
```

### Models

```php
// Maquina.php
class Maquina extends Model {
    use HasFactory;
    protected $fillable = ['nome', 'setor'];

    public function sensor()  { return $this->hasOne(Sensor::class); }
    public function pulsos()  { return $this->hasMany(Pulso::class); }
}

// Sensor.php
class Sensor extends Model {
    use HasFactory;
    protected $table = 'sensores';
    protected $fillable = ['maquina_id', 'identificador', 'porta'];

    public function maquina() { return $this->belongsTo(Maquina::class); }
}

// Pulso.php
class Pulso extends Model {
    use HasFactory;
    protected $fillable = ['maquina_id', 'voltagem', 'ruido', 'registrado_em'];
    protected $casts = [
        'voltagem' => 'decimal:2',
        'ruido' => 'decimal:2',
        'registrado_em' => 'datetime',
    ];

    public function maquina() { return $this->belongsTo(Maquina::class); }
}
```

### Seeder

```php
// MaquinaSeeder.php — 8 máquinas industriais + sensor automático
$maquinas = [
    ['nome' => 'Encartuchadeira L1',   'setor' => 'Linha 1'],
    ['nome' => 'Blistadeira L1',       'setor' => 'Linha 1'],
    ['nome' => 'Encartuchadeira L2',   'setor' => 'Linha 2'],
    ['nome' => 'Rotuladeira L2',       'setor' => 'Linha 2'],
    ['nome' => 'Torno CNC',            'setor' => 'Usinagem'],
    ['nome' => 'Prensa Hidráulica',    'setor' => 'Estamparia'],
    ['nome' => 'Injetora Plástica',    'setor' => 'Injeção'],
    ['nome' => 'Compressor Central',   'setor' => 'Utilidades'],
];
```

### Comando Artisan: `simular:pulsos`

```bash
# Padrão: 10 pulsos, 2s de intervalo
php artisan simular:pulsos

# Customizado
php artisan simular:pulsos --quantidade=50 --intervalo=0
```

Saída com progress bar + tabela de resumo com métricas por máquina.

### 🔧 Analogia Industrial — Aula 2

```
┌─────────────────────────────────────────────────────┐
│                SUPERVISÓRIO                         │
│                                                     │
│  Migrations  = Planta baixa do prédio               │
│  Eloquent    = Ficha de cadastro inteligente         │
│  Seeders     = Cadastro inicial do supervisório      │
│  Factories   = Gerador de amostras para teste        │
│  Cmd Custom  = Macro no CLP (automatiza ação)        │
│  hasMany     = Máquina tem vários pulsos             │
│  belongsTo   = Cada pulso pertence a uma máquina     │
└─────────────────────────────────────────────────────┘
```

---

## ✅ Aula 3 — API REST & Dashboard Web

### 🎯 Objetivo

> Construir a camada de API REST e um Dashboard Web, fechando o ciclo completo de uma aplicação Laravel profissional.

### O que foi feito

- [x] Form Request com validação e mensagens PT-BR (`StorePulsoRequest`)
- [x] API Resources para formatação JSON (`MaquinaResource`, `PulsoResource`)
- [x] Resource Controllers para API (`MaquinaController`, `PulsoController`)
- [x] Rotas API com `apiResource`
- [x] Dashboard Web com Blade (tema dark industrial)
- [x] Cards de resumo (máquinas, pulsos, última leitura, ativas)
- [x] Filtro por setor via dropdown
- [x] Status automático: 🟢 Ativa | 🔴 Inativa | 🟡 Alerta (ruído > 75 dB)
- [x] Paginação nos endpoints da API

### Rotas da API

```
GET    /api/maquinas           → Lista máquinas + métricas agregadas
GET    /api/maquinas/{id}      → Detalhe de uma máquina + últimos 50 pulsos
GET    /api/pulsos             → Lista pulsos (filtros + paginação)
POST   /api/pulsos             → Registra novo pulso (validado)
GET    /api/pulsos/{id}        → Detalhe de um pulso
GET    /dashboard              → Painel web de monitoramento
```

### Filtros disponíveis na API

```
GET /api/maquinas?setor=Linha 1
GET /api/maquinas?nome=CNC
GET /api/pulsos?maquina_id=3
GET /api/pulsos?data=2026-05-30
GET /api/pulsos?voltagem_min=100
GET /api/pulsos?voltagem_max=200
GET /api/pulsos?ruido_min=80
GET /api/pulsos?per_page=50
```

### Form Request — Validação

```php
// StorePulsoRequest.php
class StorePulsoRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'maquina_id'    => 'required|integer|exists:maquinas,id',
            'voltagem'      => 'required|numeric|min:0|max:500',
            'ruido'         => 'required|numeric|min:0|max:150',
            'registrado_em' => 'nullable|date',
        ];
    }

    public function messages(): array
    {
        return [
            'maquina_id.required' => 'A máquina é obrigatória.',
            'maquina_id.exists'   => 'Máquina não encontrada no cadastro.',
            'voltagem.required'   => 'A voltagem é obrigatória.',
            'voltagem.max'        => 'Voltagem máxima permitida: 500V.',
            'ruido.required'      => 'O ruído é obrigatório.',
            'ruido.max'           => 'Ruído máximo permitido: 150 dB.',
        ];
    }
}
```

### API Resources

```php
// MaquinaResource.php
class MaquinaResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'     => $this->id,
            'nome'   => $this->nome,
            'setor'  => $this->setor,
            'sensor' => [
                'identificador' => $this->sensor?->identificador,
                'porta'         => $this->sensor?->porta,
            ],
            'metricas' => [
                'total_pulsos'    => $this->pulsos_count ?? $this->pulsos->count(),
                'voltagem_media'  => round($this->pulsos->avg('voltagem'), 2),
                'ruido_medio'     => round($this->pulsos->avg('ruido'), 2),
                'ultimo_registro' => $this->pulsos->sortByDesc('registrado_em')
                                          ->first()?->registrado_em
                                          ?->format('d/m/Y H:i:s'),
            ],
            'status' => $this->resolverStatus(),
        ];
    }

    private function resolverStatus(): string
    {
        if ($this->pulsos->isEmpty()) return '🔴 Inativa';

        $ultimoRuido = $this->pulsos->sortByDesc('registrado_em')
                            ->first()?->ruido;

        return $ultimoRuido > 75 ? '🟡 Alerta' : '🟢 Ativa';
    }
}

// PulsoResource.php
class PulsoResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'             => $this->id,
            'maquina'        => $this->maquina->nome,
            'setor'          => $this->maquina->setor,
            'voltagem'       => (float) $this->voltagem,
            'ruido'          => (float) $this->ruido,
            'registrado_em'  => $this->registrado_em->format('d/m/Y H:i:s'),
        ];
    }
}
```

### Resource Controllers

```php
// Api/MaquinaController.php
class MaquinaController extends Controller
{
    public function index(Request $request)
    {
        $query = Maquina::with(['sensor', 'pulsos']);

        if ($request->filled('setor')) {
            $query->where('setor', $request->setor);
        }

        if ($request->filled('nome')) {
            $query->where('nome', 'like', '%' . $request->nome . '%');
        }

        $maquinas = $query->withCount('pulsos')
                          ->paginate($request->get('per_page', 15));

        return MaquinaResource::collection($maquinas);
    }

    public function show(Maquina $maquina)
    {
        $maquina->load(['sensor', 'pulsos' => function ($q) {
            $q->orderByDesc('registrado_em')->take(50);
        }]);

        return new MaquinaResource($maquina);
    }
}

// Api/PulsoController.php
class PulsoController extends Controller
{
    public function index(Request $request)
    {
        $query = Pulso::with('maquina');

        if ($request->filled('maquina_id')) {
            $query->where('maquina_id', $request->maquina_id);
        }

        if ($request->filled('data')) {
            $query->whereDate('registrado_em', $request->data);
        }

        if ($request->filled('voltagem_min')) {
            $query->where('voltagem', '>=', $request->voltagem_min);
        }

        if ($request->filled('voltagem_max')) {
            $query->where('voltagem', '<=', $request->voltagem_max);
        }

        if ($request->filled('ruido_min')) {
            $query->where('ruido', '>=', $request->ruido_min);
        }

        $pulsos = $query->orderByDesc('registrado_em')
                        ->paginate($request->get('per_page', 15));

        return PulsoResource::collection($pulsos);
    }

    public function store(StorePulsoRequest $request)
    {
        $pulso = Pulso::create($request->validated());
        $pulso->load('maquina');

        return (new PulsoResource($pulso))
            ->response()
            ->setStatusCode(201);
    }

    public function show(Pulso $pulso)
    {
        $pulso->load('maquina');
        return new PulsoResource($pulso);
    }
}
```

### Dashboard Controller

```php
// DashboardController.php
class DashboardController extends Controller
{
    public function index(Request $request)
    {
        $query = Maquina::with(['sensor', 'pulsos']);

        if ($request->filled('setor')) {
            $query->where('setor', $request->setor);
        }

        $maquinas = $query->get();

        $setores = Maquina::select('setor')
                          ->distinct()
                          ->orderBy('setor')
                          ->pluck('setor');

        $totalMaquinas = $maquinas->count();

        $totalPulsosHoje = Pulso::whereDate('registrado_em', today())->count();

        $ultimoRegistro = Pulso::orderByDesc('registrado_em')
                               ->first()
                               ?->registrado_em
                               ?->format('d/m/Y H:i:s') ?? '—';

        $maquinasAtivas = $maquinas->filter(fn ($m) => $m->pulsos->isNotEmpty())->count();

        return view('dashboard', compact(
            'maquinas',
            'setores',
            'totalMaquinas',
            'totalPulsosHoje',
            'ultimoRegistro',
            'maquinasAtivas'
        ));
    }
}
```

### Rotas

```php
// routes/api.php
use App\Http\Controllers\Api\MaquinaController;
use App\Http\Controllers\Api\PulsoController;

Route::apiResource('maquinas', MaquinaController::class)->only(['index', 'show']);
Route::apiResource('pulsos', PulsoController::class)->only(['index', 'store', 'show']);

// routes/web.php
use App\Http\Controllers\DashboardController;

Route::get('/dashboard', [DashboardController::class, 'index']);
```

### Dashboard Blade — Painel de Monitoramento

```
┌──────────────────────────────────────────────────────────────┐
│  🏭 SENSOR LEGACY — Painel de Monitoramento                 │
├──────────────────────────────────────────────────────────────┤
│  📊 Cards: Máquinas | Pulsos Hoje | Último Registro | Ativas│
├──────────────────────────────────────────────────────────────┤
│  🔽 Filtro por setor (dropdown)                              │
├──────────────────────────────────────────────────────────────┤
│  Máquina │ Setor │ Sensor │ Porta │ Pulsos │ V │ dB │ Status│
│  ────────┼───────┼────────┼───────┼────────┼───┼────┼───────│
│  Enc. L1 │ L1    │SNS-001 │USB0   │   23   │127│ 42 │ 🟢   │
│  Torno   │ Usin. │SNS-005 │USB4   │    0   │ — │  — │ 🔴   │
│  Prensa  │ Est.  │SNS-006 │USB5   │   15   │198│ 82 │ 🟡   │
└──────────────────────────────────────────────────────────────┘
```

### 🔧 Analogia Industrial — Aula 3

```
┌─────────────────────────────────────────────────────┐
│                ENTREGA FINAL                        │
│                                                     │
│  Form Request   = Porteiro (valida antes de entrar) │
│  API Resource   = Painel supervisório (formata)     │
│  Controller     = Gerente (busca e entrega)         │
│  Blade          = Formulário pré-impresso           │
│  Rotas Web      = Tela do supervisório              │
│  Rotas API      = Interface serial RS-485           │
│  Filtros        = Seletor de linha no painel        │
│  Paginação      = Lote de peças no carrossel        │
│  Status 🟢🟡🔴  = Semáforo do CLP                   │
└─────────────────────────────────────────────────────┘
```

---

## 🚀 Como rodar o projeto

### Pré-requisitos

- PHP 8.2+
- Composer

### Instalação

```bash
git clone <url-do-repo>
cd sensor-legacy
composer install
cp .env.example .env
php artisan key:generate
touch database/database.sqlite
php artisan migrate --seed
```

### Executar

```bash
# Terminal 1 — Servidor
php artisan serve

# Terminal 2 — Simular dados
php artisan simular:pulsos --quantidade=50 --intervalo=0

# Acessar
# Dashboard: http://localhost:8000/dashboard
# API:       http://localhost:8000/api/maquinas
#            http://localhost:8000/api/pulsos
```

### Testar a API

```bash
# GET — Listar máquinas
curl -s http://localhost:8000/api/maquinas | python3 -m json.tool

# GET — Filtrar por setor
curl -s 'http://localhost:8000/api/maquinas?setor=Linha 1' | python3 -m json.tool

# GET — Pulsos filtrados por máquina
curl -s 'http://localhost:8000/api/pulsos?maquina_id=1' | python3 -m json.tool

# GET — Pulsos com filtro de voltagem
curl -s 'http://localhost:8000/api/pulsos?voltagem_min=100&voltagem_max=200' | python3 -m json.tool

# POST — Criar pulso (válido)
curl -s -X POST http://localhost:8000/api/pulsos \\
  -H 'Content-Type: application/json' \\
  -H 'Accept: application/json' \\
  -d '{"maquina_id": 1, "voltagem": 127.5, "ruido": 42.3}' | python3 -m json.tool

# POST — Criar pulso (inválido — testa validação)
curl -s -X POST http://localhost:8000/api/pulsos \\
  -H 'Content-Type: application/json' \\
  -H 'Accept: application/json' \\
  -d '{"maquina_id": 999, "voltagem": 600}' | python3 -m json.tool
```

---

## 📁 Estrutura Final do Projeto

```
sensor-legacy/
├── app/
│   ├── Console/Commands/
│   │   └── SimularPulsos.php          # Comando de simulação
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Api/
│   │   │   │   ├── MaquinaController.php   # API: máquinas
│   │   │   │   └── PulsoController.php     # API: pulsos
│   │   │   └── DashboardController.php     # Web: dashboard
│   │   ├── Requests/
│   │   │   └── StorePulsoRequest.php       # Validação
│   │   └── Resources/
│   │       ├── MaquinaResource.php         # JSON: máquina
│   │       └── PulsoResource.php           # JSON: pulso
│   └── Models/
│       ├── Maquina.php
│       ├── Sensor.php
│       └── Pulso.php
├── database/
│   ├── factories/
│   │   └── PulsoFactory.php
│   ├── migrations/
│   │   ├── xxxx_create_maquinas_table.php
│   │   ├── xxxx_create_sensores_table.php
│   │   └── xxxx_create_pulsos_table.php
│   └── seeders/
│       ├── DatabaseSeeder.php
│       └── MaquinaSeeder.php
├── resources/views/
│   └── dashboard.blade.php             # Painel web
├── routes/
│   ├── api.php                         # Rotas API REST
│   └── web.php                         # Rota do dashboard
└── README.md
```

---

## 🛠️ Tecnologias

| Tecnologia          | Uso                                |
|---------------------|------------------------------------|
| **Laravel 11**      | Framework principal                |
| **Eloquent ORM**    | Models, Relations, Query Builder   |
| **Artisan CLI**     | Commands, Migrations, Seeders      |
| **Blade**           | Template engine (Dashboard)        |
| **API Resources**   | Formatação JSON                    |
| **Form Requests**   | Validação server-side              |
| **SQLite**          | Banco de dados (desenvolvimento)   |

---

## 🧠 Conceitos Aprendidos no Curso

| Aula | Conceito                | Analogia Industrial                               |
|------|-------------------------|---------------------------------------------------|
| 1    | MVC                     | Separar quem opera, quem processa, quem exibe      |
| 1    | Artisan CLI             | Painel de controle do operador                     |
| 1    | `.env`               | Chave do quadro elétrico (nunca compartilhar)      |
| 2    | Migrations              | Planta baixa do prédio                             |
| 2    | Eloquent ORM            | Ficha de cadastro inteligente                      |
| 2    | Seeders                 | Cadastro inicial do supervisório                   |
| 2    | Factories               | Gerador de amostras para teste                     |
| 2    | Comando Artisan custom  | Macro no CLP                                       |
| 3    | Form Request            | Porteiro da fábrica (valida antes de entrar)       |
| 3    | API Resource            | Painel do supervisório (formata dados pro display) |
| 3    | Resource Controller     | Gerente (busca dado e entrega pro painel)          |
| 3    | Blade Template          | Formulário pré-impresso (HTML + dados variáveis)   |
| 3    | Rotas Web vs API        | Tela do supervisório vs interface serial RS-485    |
| 3    | Filtros (Query String)  | Seletor de linha no painel de controle             |
| 3    | Paginação               | Lote de peças no carrossel de produção             |
| 3    | Status 🟢🟡🔴           | Semáforo de status do CLP                          |

---

## 📝 Histórico de Commits

| Aula | Commit                                                     |
|------|------------------------------------------------------------|
| 1    | `feat: scaffold inicial do projeto Laravel`              |
| 2    | `feat: models, migrations, seeders e comando de simulação`|
| 3    | `feat: API REST, resources, validação e dashboard web`   |
| 3    | `docs: README final do curso completo`                   |

---

## 🎓 Certificação

Curso concluído em **30/05/2026** na escola **Flexxo — Caxias do Sul/RS**.

---

## 👨‍🏫 Créditos

**Instrutor:** Wiliam — Flexxo Caxias do Sul
**Framework:** Laravel by Taylor Otwell
**Projeto:** Sensor Legacy — Monitoramento Industrial Didático

---

> _"O melhor jeito de aprender um framework é construindo algo que faça sentido no mundo real."_
