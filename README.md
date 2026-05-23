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

---

## 🗺️ Visão Geral das Aulas

```
AULA 1                    AULA 2                      AULA 3
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Fundamentos Laravel       Dados & Simulação           API & Dashboard
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• Instalação              • Migrations                • Routes & Controllers
• Estrutura MVC           • Models & Relations        • API Resources (JSON)
• Artisan básico          • Seeders & Factories       • Form Requests
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

## ✅ Aula 1 — Fundamentos Laravel (Concluída)

### 🎯 Objetivo

> Instalar o Laravel, entender a estrutura MVC e se familiarizar com o ecossistema e o Artisan CLI.

### O que foi feito

- [x] Instalação do Laravel via Composer
- [x] Configuração do ambiente (`.env`, SQLite)
- [x] Exploração da estrutura MVC
- [x] Primeiros comandos Artisan (`serve`, `make:model`, `make:migration`)
- [x] Conceito de rotas, controllers e views
- [x] Entendimento do ciclo de vida de uma request no Laravel

### Conceitos-chave abordados

| Conceito           | Descrição                                                         |
|--------------------|-------------------------------------------------------------------|
| **MVC**            | Model-View-Controller — separação de responsabilidades            |
| **Artisan**        | CLI nativo do Laravel para automação de tarefas                   |
| **Composer**       | Gerenciador de dependências PHP                                   |
| **`.env`**        | Arquivo de variáveis de ambiente (nunca comitar!)                 |
| **Service Container** | Motor de injeção de dependências do Laravel                    |
| **Routes**         | Mapeamento URL → Controller → Response                            |

---

## ✅ Aula 2 — Dados & Simulação (Concluída)

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

---

### 📦 Bloco 1 — Migrations

#### Conceito

> Migrations são o **controle de versão do banco de dados**. Cada migration é um arquivo PHP que descreve uma alteração na estrutura do banco. Permite que qualquer dev da equipe reproduza o banco exato com um único comando.

#### Comandos utilizados

```bash
php artisan make:migration create_maquinas_table
php artisan make:migration create_sensores_table
php artisan make:migration create_pulsos_table
php artisan migrate
php artisan migrate:fresh --seed   # Rebuild completo
```

#### Migration: `create_maquinas_table`

```php
Schema::create('maquinas', function (Blueprint $table) {
    $table->id();
    $table->string('nome');
    $table->string('setor');
    $table->timestamps();
});
```

#### Migration: `create_sensores_table`

```php
Schema::create('sensores', function (Blueprint $table) {
    $table->id();
    $table->foreignId('maquina_id')->constrained('maquinas')->onDelete('cascade');
    $table->string('identificador');
    $table->string('porta');
    $table->timestamps();
});
```

#### Migration: `create_pulsos_table`

```php
Schema::create('pulsos', function (Blueprint $table) {
    $table->id();
    $table->foreignId('maquina_id')->constrained('maquinas')->onDelete('cascade');
    $table->decimal('voltagem', 8, 2);
    $table->decimal('ruido', 8, 2);
    $table->timestamp('registrado_em')->useCurrent();
    $table->timestamps();
});
```

#### Analogia

> **Migration = planta baixa de um prédio.** Você descreve exatamente o que cada andar (tabela) vai ter. Se precisar reformar, cria uma nova migration — nunca edita a planta original.

---

### 📦 Bloco 2 — Models & Relacionamentos Eloquent

#### Conceito

> O Eloquent é o ORM do Laravel. Cada Model representa uma tabela do banco. Os relacionamentos (`hasOne`, `hasMany`, `belongsTo`) permitem navegar entre tabelas de forma fluente, sem escrever SQL manual.

#### Model: `Maquina`

```php
// app/Models/Maquina.php
class Maquina extends Model
{
    use HasFactory;

    protected $fillable = ['nome', 'setor'];

    public function sensor()
    {
        return $this->hasOne(Sensor::class);
    }

    public function pulsos()
    {
        return $this->hasMany(Pulso::class);
    }
}
```

#### Model: `Sensor`

```php
// app/Models/Sensor.php
class Sensor extends Model
{
    use HasFactory;

    protected $table = 'sensores';
    protected $fillable = ['maquina_id', 'identificador', 'porta'];

    public function maquina()
    {
        return $this->belongsTo(Maquina::class);
    }
}
```

#### Model: `Pulso`

```php
// app/Models/Pulso.php
class Pulso extends Model
{
    use HasFactory;

    protected $fillable = ['maquina_id', 'voltagem', 'ruido', 'registrado_em'];

    protected $casts = [
        'voltagem'      => 'decimal:2',
        'ruido'         => 'decimal:2',
        'registrado_em' => 'datetime',
    ];

    public function maquina()
    {
        return $this->belongsTo(Maquina::class);
    }
}
```

#### Mapa de relacionamentos

```
┌──────────┐    hasOne     ┌──────────┐
│ Maquina  │──────────────▶│  Sensor  │
│          │               │          │
│          │◀──────────────│belongsTo │
│          │    hasMany    ┌──────────┐
│          │──────────────▶│  Pulso   │
│          │               │          │
│          │◀──────────────│belongsTo │
└──────────┘               └──────────┘
```

#### Analogia

> **Model = ficha de cadastro inteligente.** Ao invés de ir no arquivo morto (banco) e procurar manualmente, você pergunta pro Model: *"Me traga todos os pulsos desta máquina"* — e ele faz o SQL pra você.

---

### 📦 Bloco 3 — Seeders

#### Conceito

> Seeders populam o banco com dados iniciais. Ideal para dados fixos (máquinas do chão de fábrica) e para testes.

#### Comando

```bash
php artisan make:seeder MaquinaSeeder
php artisan db:seed
```

#### Seeder: `MaquinaSeeder`

```php
// database/seeders/MaquinaSeeder.php
class MaquinaSeeder extends Seeder
{
    public function run(): void
    {
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

        foreach ($maquinas as $m) {
            $maquina = Maquina::create($m);

            Sensor::create([
                'maquina_id'    => $maquina->id,
                'identificador' => 'SNS-' . str_pad($maquina->id, 3, '0', STR_PAD_LEFT),
                'porta'         => '/dev/ttyUSB' . ($maquina->id - 1),
            ]);
        }
    }
}
```

#### DatabaseSeeder

```php
// database/seeders/DatabaseSeeder.php
class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            MaquinaSeeder::class,
        ]);
    }
}
```

#### Analogia

> **Seeder = cadastro inicial do sistema.** É como o técnico que, antes de ligar a fábrica, registra todas as máquinas no sistema supervisório.

---

### 📦 Bloco 4 — Factories

#### Conceito

> Factories geram dados falsos (mas realistas) automaticamente. Usam a lib Faker. Essenciais para testes e carga de dados.

#### Comando

```bash
php artisan make:factory PulsoFactory --model=Pulso
```

#### Factory: `PulsoFactory`

```php
// database/factories/PulsoFactory.php
class PulsoFactory extends Factory
{
    protected $model = Pulso::class;

    public function definition(): array
    {
        return [
            'maquina_id'    => Maquina::inRandomOrder()->first()->id,
            'voltagem'      => fake()->randomFloat(2, 24, 220),
            'ruido'         => fake()->randomFloat(2, 0, 100),
            'registrado_em' => now(),
        ];
    }
}
```

#### Analogia

> **Factory = gerador de amostras.** Como um simulador que cria leituras fictícias mas dentro dos parâmetros reais da operação.

---

### 📦 Bloco 5 — Comando Artisan Customizado

#### Conceito

> O Artisan permite criar comandos CLI personalizados. Aqui criamos o `simular:pulsos`, que gera leituras de sensores como se fossem dados chegando de dispositivos reais.

#### Comando de criação

```bash
php artisan make:command SimularPulsos
```

#### Código completo: `SimularPulsos.php`

```php
// app/Console/Commands/SimularPulsos.php
namespace App\Console\Commands;

use App\Models\Maquina;
use App\Models\Pulso;
use Illuminate\Console\Command;

class SimularPulsos extends Command
{
    protected $signature = 'simular:pulsos
        {--quantidade=10 : Quantidade de pulsos a gerar}
        {--intervalo=2 : Intervalo em segundos entre pulsos}';

    protected $description = 'Simula pulsos de sensores industriais';

    public function handle()
    {
        $quantidade = (int) $this->option('quantidade');
        $intervalo  = (int) $this->option('intervalo');
        $maquinas   = Maquina::with('sensor')->get();

        if ($maquinas->isEmpty()) {
            $this->error('Nenhuma máquina cadastrada. Rode: php artisan db:seed');
            return 1;
        }

        $this->info('');
        $this->info('🏭 Simulando pulsos industriais...');
        $this->info('');

        $bar = $this->output->createProgressBar($quantidade);
        $bar->setFormat(' %current%/%max% [%bar%] %percent:3s%% ⏱  %elapsed%');
        $bar->start();

        for ($i = 0; $i < $quantidade; $i++) {
            $maquina  = $maquinas->random();
            $voltagem = round(mt_rand(2400, 22000) / 100, 2);
            $ruido    = round(mt_rand(0, 10000) / 100, 2);

            Pulso::create([
                'maquina_id'    => $maquina->id,
                'voltagem'      => $voltagem,
                'ruido'         => $ruido,
                'registrado_em' => now(),
            ]);

            $porta = $maquina->sensor->porta ?? 'N/A';
            $bar->advance();

            if ($intervalo > 0 && $i < $quantidade - 1) {
                sleep($intervalo);
            }
        }

        $bar->finish();
        $this->newLine(2);

        // ── Resumo do dia ──
        $this->info('✅ Simulação concluída! Resumo do dia:');
        $this->newLine();

        $resumo = Maquina::with(['sensor', 'pulsos' => function ($q) {
            $q->whereDate('created_at', today());
        }])->get();

        $rows = $resumo->map(function ($m) {
            return [
                $m->nome,
                $m->sensor->identificador ?? '–',
                $m->pulsos->count(),
                round($m->pulsos->avg('voltagem'), 2) . ' V',
                round($m->pulsos->avg('ruido'), 2) . ' dB',
            ];
        });

        $this->table(
            ['Máquina', 'Sensor', 'Pulsos Hoje', 'Voltagem Média', 'Ruído Médio'],
            $rows
        );

        return 0;
    }
}
```

#### Como usar

```bash
# Padrão: 10 pulsos, 2s de intervalo
php artisan simular:pulsos

# Customizado: 20 pulsos, 1s de intervalo
php artisan simular:pulsos --quantidade=20 --intervalo=1

# Rajada rápida: 50 pulsos, sem intervalo
php artisan simular:pulsos --quantidade=50 --intervalo=0
```

#### Saída esperada

```
🏭 Simulando pulsos industriais...

 20/20 [████████████████████████████████████████] 100% ⏱  0:22

✅ Simulação concluída! Resumo do dia:

┌──────────────────────┬─────────────┬─────────────┬────────────────┬─────────────┐
│ Máquina              │ Sensor      │ Pulsos Hoje │ Voltagem Média │ Ruído Médio │
├──────────────────────┼─────────────┼─────────────┼────────────────┼─────────────┤
│ Encartuchadeira L1   │ SNS-001     │ 3           │ 152.51 V       │ 28.03 dB    │
│ Blistadeira L1       │ SNS-002     │ 1           │ 213.55 V       │ 75.17 dB    │
│ Encartuchadeira L2   │ SNS-003     │ 3           │ 160.67 V       │ 51.02 dB    │
│ Rotuladeira L2       │ SNS-004     │ 6           │ 145.06 V       │ 24.66 dB    │
│ Torno CNC            │ SNS-005     │ 4           │ 123.85 V       │ 19.61 dB    │
│ Prensa Hidráulica    │ SNS-006     │ 2           │ 198.30 V       │ 62.44 dB    │
│ Injetora Plástica    │ SNS-007     │ 1           │ 176.20 V       │ 45.80 dB    │
│ Compressor Central   │ SNS-008     │ 0           │ 0.00 V         │ 0.00 dB     │
└──────────────────────┴─────────────┴─────────────┴────────────────┴─────────────┘
```

#### Analogia

> **Comando Artisan custom = macro no CLP.** Você programa uma rotina automática que roda sob demanda, sem precisar abrir o sistema inteiro.

---

### 📁 Arquivos criados/modificados na Aula 2

```
app/
├── Console/Commands/
│   └── SimularPulsos.php
├── Models/
│   ├── Maquina.php
│   ├── Sensor.php
│   └── Pulso.php
database/
├── factories/
│   ├── MaquinaFactory.php
│   ├── SensorFactory.php
│   └── PulsoFactory.php
├── migrations/
│   ├── xxxx_create_maquinas_table.php
│   ├── xxxx_create_sensores_table.php
│   └── xxxx_create_pulsos_table.php
└── seeders/
    ├── DatabaseSeeder.php
    └── MaquinaSeeder.php
```

---

## 🔜 Aula 3 — API REST & Dashboard Web (Próxima)

### 🎯 Objetivo

> Construir a camada de API REST e um Dashboard Web que consome os dados dos pulsos simulados, fechando o ciclo completo de uma aplicação Laravel profissional.

### Cronograma (4h)

| Bloco | Tema                          | Duração |
|-------|-------------------------------|---------|
| 1     | Routes & Resource Controllers | 45 min  |
| 2     | API Resources (JSON)          | 30 min  |
| 3     | Validação com Form Request    | 30 min  |
| 4     | Dashboard Web com Blade       | 45 min  |
| 5     | Filtros e Queries Avançadas   | 30 min  |
| 6     | Fechamento e Revisão          | 30 min  |

### Entregas previstas

- [ ] Rotas API (`/api/maquinas`, `/api/pulsos`)
- [ ] `MaquinaController` e `PulsoController` (Resource)
- [ ] `MaquinaResource` e `PulsoResource` (JSON formatado)
- [ ] `StorePulsoRequest` (validação)
- [ ] `DashboardController` + View Blade
- [ ] Filtros por máquina, data, voltagem mínima
- [ ] Paginação nos endpoints

### Rotas planejadas

```
GET    /api/maquinas           → Lista todas as máquinas + métricas
GET    /api/maquinas/{id}      → Detalhe de uma máquina
GET    /api/pulsos             → Lista pulsos (com filtros)
POST   /api/pulsos             → Registra novo pulso (validado)
GET    /dashboard              → Painel web de monitoramento
```

### Código planejado (preview)

#### Resource Controller

```php
// MaquinaController
public function index()
{
    $maquinas = Maquina::with(['sensor', 'pulsos'])
                       ->withCount('pulsos')
                       ->get();

    return MaquinaResource::collection($maquinas);
}
```

#### API Resource

```php
// MaquinaResource
public function toArray($request)
{
    return [
        'id'             => $this->id,
        'nome'           => $this->nome,
        'setor'          => $this->setor,
        'sensor'         => $this->sensor?->identificador,
        'total_pulsos'   => $this->pulsos_count,
        'voltagem_media' => round($this->pulsos->avg('voltagem'), 2),
        'ruido_medio'    => round($this->pulsos->avg('ruido'), 2),
        'status'         => $this->pulsos->avg('voltagem') > 0 ? 'ativa' : 'inativa',
    ];
}
```

#### Form Request

```php
// StorePulsoRequest
public function rules(): array
{
    return [
        'maquina_id' => 'required|exists:maquinas,id',
        'voltagem'   => 'required|numeric|min:0|max:500',
        'ruido'      => 'required|numeric|min:0|max:150',
    ];
}
```

#### Dashboard Blade

```html
<!-- Painel estilo industrial dark com tabela de monitoramento em tempo real -->
<!-- Exibe: Máquina | Setor | Sensor | Pulsos Hoje | Voltagem Média | Ruído Médio | Status -->
```

### Arquivos a serem criados

```
app/
├── Http/
│   ├── Controllers/
│   │   ├── Api/
│   │   │   ├── MaquinaController.php
│   │   │   └── PulsoController.php
│   │   └── DashboardController.php
│   ├── Requests/
│   │   └── StorePulsoRequest.php
│   └── Resources/
│       ├── MaquinaResource.php
│       └── PulsoResource.php
resources/
└── views/
    └── dashboard.blade.php
routes/
├── api.php
└── web.php (atualizado)
```

---

## 🚀 Como rodar o projeto

### Pré-requisitos

- PHP 8.2+
- Composer
- Node.js (opcional, para assets)

### Instalação

```bash
# Clonar o repositório
git clone <url-do-repo>
cd sensor-legacy

# Instalar dependências
composer install

# Copiar .env
cp .env.example .env

# Gerar chave
php artisan key:generate

# Criar banco SQLite
touch database/database.sqlite

# Rodar migrations e seeders
php artisan migrate --seed
```

### Executar

```bash
# Subir o servidor
php artisan serve

# Simular pulsos (em outro terminal)
php artisan simular:pulsos --quantidade=50 --intervalo=1

# Acessar
# API:       http://localhost:8000/api/maquinas
# Dashboard: http://localhost:8000/dashboard
```

---

## 🛠️ Tecnologias

| Tecnologia      | Uso                              |
|-----------------|----------------------------------|
| **Laravel 11**  | Framework principal              |
| **Eloquent**    | ORM (Models, Relations, Queries) |
| **Artisan**     | CLI (Commands, Migrations, Seed) |
| **Blade**       | Template engine (Dashboard)      |
| **SQLite**      | Banco de dados (desenvolvimento) |
| **JSON API**    | Comunicação REST                 |

---

## 📁 Estrutura do Projeto (Visão Geral)

```
sensor-legacy/
├── app/
│   ├── Console/Commands/        # Comando de simulação
│   ├── Http/
│   │   ├── Controllers/         # Controllers Web e API
│   │   ├── Requests/            # Form Requests (validação)
│   │   └── Resources/           # API Resources (JSON)
│   └── Models/                  # Eloquent Models
├── database/
│   ├── factories/               # Factories para testes
│   ├── migrations/              # Estrutura do banco
│   └── seeders/                 # Dados iniciais
├── resources/views/             # Views Blade (Dashboard)
├── routes/
│   ├── api.php                  # Rotas da API REST
│   └── web.php                  # Rotas Web
└── README.md                    # ← Você está aqui
```

---

## 📝 Histórico de Commits

| Aula | Commit sugerido                                            |
|------|-----------------------------------------------------------|
| 1    | `feat: scaffold inicial do projeto Laravel`                |
| 2    | `feat: models, migrations, seeders e comando de simulação` |
| 3    | `feat: API REST, resources, validação e dashboard web`     |

---

## 👨‍🏫 Créditos

**Instrutor:** Wiliam — Flexxo Caxias do Sul
**Framework:** Laravel by Taylor Otwell
**Projeto:** Sensor Legacy — Monitoramento Industrial Didático

---

> _"O melhor jeito de aprender um framework é construindo algo que faça sentido no mundo real."_
