# 📘 Aula 01 — Upgrade de Projeto Laravel 8 para Laravel 12

## Módulo: Laravel Avançado | Flexxo - Polo Caxias do Sul
**Data:** 16 de Maio de 2026
**Instrutor:** Wiliam
**Aluno:** Juares (Mentoria 1:1)
**Modalidade:** Remoto

---

## 📌 Índice

1. [Objetivo da Aula](#1--objetivo-da-aula)
2. [Cenário Inicial do Projeto](#2--cenário-inicial-do-projeto)
3. [Estratégia de Upgrade — Por que Não Pular Versões?](#3--estratégia-de-upgrade--por-que-não-pular-versões)
4. [Roadmap Completo de Upgrade](#4--roadmap-completo-de-upgrade)
5. [Ferramentas e Fontes Oficiais](#5--ferramentas-e-fontes-oficiais)
6. [Prática — Upgrade Laravel 8 → 9](#6--prática--upgrade-laravel-8--9)
7. [Troubleshooting — Conflito de Dependências](#7--troubleshooting--conflito-de-dependências)
8. [Pacotes Deprecated vs Substitutos](#8--pacotes-deprecated-vs-substitutos)
9. [Comandos Composer Essenciais](#9--comandos-composer-essenciais)
10. [Próximos Passos](#10--próximos-passos)
11. [Glossário Rápido](#11--glossário-rápido)

---

## 1. 🎯 Objetivo da Aula

Planejar e iniciar a execução do **upgrade de um projeto real Laravel 8 para Laravel 12**,
passando por todas as versões intermediárias, seguindo as boas práticas da documentação oficial.

Ao final desta aula, o aluno será capaz de:

- Entender o ciclo de versionamento do Laravel (SemVer)
- Ler e interpretar erros do Composer
- Identificar pacotes descontinuados e seus substitutos
- Manipular o `composer.json` para realizar upgrades controlados
- Utilizar flags avançadas do Composer (`--no-update`, `-W`)

---

## 2. 🔍 Cenário Inicial do Projeto

Antes de qualquer alteração, mapeamos o estado atual:

| Item                | Valor Encontrado       |
|---------------------|------------------------|
| Framework           | Laravel 8.x            |
| PHP do Servidor     | 8.3.31                 |
| Gerenciador         | Composer 2.x           |
| Erro Handler        | `facade/ignition ^2.5`  |

> **⚠️ Ponto de Atenção:** O PHP 8.3 já é mais recente do que o Laravel 8 espera.
> Isso pode gerar deprecation warnings, mas funciona. O upgrade resolverá isso.

---

## 3. 🧠 Estratégia de Upgrade — Por que Não Pular Versões?

O Laravel segue o **Versionamento Semântico (SemVer)**:

```
MAJOR.MINOR.PATCH
  9  .  52  .  0
  │     │     └── Correções de bugs
  │     └──────── Novas funcionalidades (retrocompatíveis)
  └────────────── Breaking changes (pode quebrar seu código)
```

### Por que saltar versão a versão?

- Cada versão MAJOR tem seu **Upgrade Guide** oficial
- Pular de 8 para 12 diretamente causaria **centenas de conflitos simultâneos**
- Fazendo salto a salto, você resolve **poucos conflitos por vez**
- É a abordagem recomendada pelo próprio **Taylor Otwell** (criador do Laravel)

### Analogia Simples

> Imagine que você precisa subir do 1º ao 4º andar.
> Você pode subir de escada (andar por andar) ou tentar pular da janela do 1º direto pro 4º.
> O resultado é bem diferente. 😄

---

## 4. 🗺️ Roadmap Completo de Upgrade

| Salto     | Dificuldade    | Tempo Estimado | Principais Mudanças                          |
|-----------|----------------|----------------|-----------------------------------------------|
| 8 → 9    | 🟡 Média       | ~30 min        | PHP 8.0+, Symfony 6, Flysystem 3, Ignition   |
| 9 → 10   | 🟢 Fácil       | ~10 min        | PHP 8.1+, Processo nativo, Pest opcional      |
| 10 → 11  | 🟡 Média       | ~15 min        | PHP 8.2+, SQLite padrão, novo starter kit     |
| 11 → 12  | 🟢 Fácil       | ~5 min         | PHP 8.2+, refinamentos e melhorias            |

> **Tempo total estimado:** ~1 hora (sem contar testes da aplicação)

---

## 5. 🛠️ Ferramentas e Fontes Oficiais

### Documentação Oficial (Upgrade Guides)

| Versão | Link                                              |
|--------|---------------------------------------------------|
| 8 → 9  | https://laravel.com/docs/9.x/upgrade              |
| 9 → 10 | https://laravel.com/docs/10.x/upgrade             |
| 10 → 11| https://laravel.com/docs/11.x/upgrade             |
| 11 → 12| https://laravel.com/docs/12.x/upgrade             |

### Laravel Shift (Ferramenta Automatizada)

- **Site:** https://laravelshift.com
- **O que faz:** Automatiza o upgrade entre versões via Pull Request
- **Custo:** ~$29 a $39 por salto
- **Recomendado por:** Taylor Otwell (criador do Laravel)
- **Quando usar:** Projetos grandes com muitas dependências

---

## 6. 🔧 Prática — Upgrade Laravel 8 → 9

### Passo 1: Editar o `composer.json`

Alterar a versão do framework:

```json
// ANTES (Laravel 8)
"laravel/framework": "^8.0"

// DEPOIS (Laravel 9)
"laravel/framework": "^9.0"
```

### Passo 2: Tentar o Update

```bash
composer update --with-all-dependencies
```

### Passo 3: Analisar o Resultado

Neste ponto, o Composer **retornou um erro de conflito**.
Isso é **normal e esperado** em upgrades — o importante é saber diagnosticar.

---

## 7. 🔥 Troubleshooting — Conflito de Dependências

### O Erro Encontrado

O Composer informou que **todas as versões do Laravel 9 foram rejeitadas**.

### Diagnóstico

Analisando a árvore de dependências:

```
facade/ignition ^2.5
  └── requer illuminate/support ^7.0|^8.0
        └── CONFLITA com laravel/framework ^9.0 (que traz illuminate/support ^9.0)
```

### Causa Raiz

O pacote `facade/ignition` foi **descontinuado** após o Laravel 8.
Ele exige versões antigas do `illuminate/support` que:

- São incompatíveis com o Laravel 9+
- Requerem PHP ^7.3 (incompatível com nosso PHP 8.3)

### Solução Aplicada

```bash
# 1. Remover o pacote descontinuado
composer remove facade/ignition --no-update

# 2. Adicionar o substituto oficial (Spatie)
composer require spatie/laravel-ignition "^1.0" --no-update

# 3. Rodar o update completo
composer update --with-all-dependencies
```

> **💡 Dica Pro:** A flag `--no-update` permite alterar o `composer.json`
> sem executar o update imediatamente. Assim você faz várias alterações
> e roda um único update no final.

---

## 8. 📦 Pacotes Deprecated vs Substitutos

Este é um padrão comum no ecossistema Laravel. Pacotes mudam de mantenedor:

| Laravel     | Pacote de Error Handler          | Mantenedor |
|-------------|----------------------------------|------------|
| 6, 7, 8     | `facade/ignition`                | Facade     |
| 9, 10       | `spatie/laravel-ignition ^1.0`   | Spatie     |
| 11, 12      | `spatie/laravel-ignition ^2.0`   | Spatie     |

### Como Identificar Pacotes Deprecated?

1. **Leia o erro do Composer** — ele mostra a cadeia de conflitos
2. **Acesse o repositório no GitHub** — pacotes abandonados geralmente têm um aviso
3. **Consulte o Packagist** (https://packagist.org) — mostra se há substituto
4. **Consulte o Upgrade Guide** — lista as mudanças de pacotes obrigatórias

---

## 9. 🧰 Comandos Composer Essenciais

Referência rápida dos comandos mais usados durante upgrades:

```bash
# Atualizar todas as dependências
composer update

# Atualizar com resolução agressiva de dependências
composer update --with-all-dependencies
# ou forma curta:
composer update -W

# Remover pacote SEM rodar update
composer remove nome/pacote --no-update

# Adicionar pacote SEM rodar update
composer require nome/pacote "^1.0" --no-update

# Ver versão atual do Laravel
php artisan --version

# Ver todas as dependências instaladas
composer show

# Ver por que um pacote está instalado
composer depends nome/pacote

# Verificar problemas no composer.json
composer validate

# Limpar cache do Composer
composer clear-cache
```

---

## 10. 📋 Próximos Passos (Próxima Aula)

- [ ] Confirmar que o upgrade 8 → 9 finalizou com sucesso
- [ ] Testar a aplicação no Laravel 9 (rotas, migrations, funcionalidades)
- [ ] Resolver possíveis deprecation warnings do PHP 8.3
- [ ] Executar upgrade 9 → 10
- [ ] Executar upgrade 10 → 11
- [ ] Executar upgrade 11 → 12
- [ ] Validar o projeto 100% funcional no Laravel 12

---

## 11. 📖 Glossário Rápido

| Termo                  | Definição                                                                 |
|------------------------|---------------------------------------------------------------------------|
| **SemVer**             | Versionamento Semântico (MAJOR.MINOR.PATCH)                              |
| **Breaking Change**    | Mudança que pode quebrar código existente                                 |
| **Deprecated**         | Funcionalidade/pacote marcado para remoção futura                         |
| **Upgrade Guide**      | Documento oficial com todas as mudanças entre versões                     |
| **composer.json**      | Arquivo que declara as dependências do projeto                            |
| **composer.lock**      | Arquivo que trava as versões exatas instaladas                            |
| **Packagist**          | Repositório central de pacotes PHP/Composer                               |
| **illuminate/support** | Pacote core do Laravel (instalado automaticamente pelo framework)         |
| **--no-update**        | Flag do Composer que altera o JSON sem executar o update                  |
| **-W**                 | Atalho para --with-all-dependencies (permite downgrade/upgrade em cadeia) |

---

## 🏷️ Informações do Curso

| Item               | Detalhe                                    |
|--------------------|---------------------------------------------|
| **Escola**         | Flexxo — Polo Caxias do Sul                 |
| **Curso**          | Laravel Avançado (Mentoria)                  |
| **Carga Horária**  | 12 horas (3 aulas de 4h)                    |
| **Aula**           | 01 de 03                                     |
| **Formato**        | Mentoria Individual (1:1)                    |
| **Repositório**    | flexxo-laravel-upgrade-course                |

---

*Material de apoio gerado em 16/05/2026 — Flexxo, Polo Caxias do Sul*
*Instrutor: Wiliam | Todos os direitos reservados.*
