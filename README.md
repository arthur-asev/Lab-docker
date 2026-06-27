# Instrucao Docker Laravel

Projeto Laravel 12 com PHP 8.4, Filament 4, MySQL 8, Redis, Nginx, Mailpit, Queue, Scheduler, Composer, Node.js e Vite usando Docker Compose.

## Arquitetura

```text
Projeto
|
|-- Dockerfile
|-- docker-compose.yml
|-- docker-compose.prod.yml
|-- docker/
|   |-- nginx/default.conf
|   |-- php/php.ini
|   |-- php/www.conf
|   `-- mysql/my.cnf
|
`-- src/
    |-- app/
    |-- config/
    |-- database/
    |-- public/
    |-- resources/
    |-- routes/
    |-- tests/
    |-- composer.json
    |-- package.json
    `-- .env
```

Servicos principais:

- `app`: container PHP 8.4 FPM com Composer, extensoes PHP, Redis extension e Node.js.
- `nginx`: servidor HTTP que aponta para `src/public` e encaminha PHP para o container `app`.
- `mysql`: banco MySQL 8.4 com volume persistente.
- `redis`: usado para cache, session e queue.
- `queue`: executa `php artisan queue:work redis`.
- `scheduler`: executa `php artisan schedule:work`.
- `node`: executa Vite em modo desenvolvimento, usando o profile `dev`.
- `mailpit`: captura emails locais em uma interface web.

## Requisitos

- Docker
- Docker Compose
- Git

Composer, PHP e Node tambem existem dentro dos containers, entao nao precisam estar instalados na maquina para uso normal.

## Configuracao Inicial

1. Copie o arquivo de ambiente do Laravel:

```bash
cp src/.env.example src/.env
```

2. Ajuste `src/.env` para usar os servicos do Docker:

```dotenv
APP_NAME="Instrucao Docker"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8080
APP_TIMEZONE=America/Sao_Paulo

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=instrucao
DB_USERNAME=laravel
DB_PASSWORD=secret

CACHE_STORE=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

REDIS_CLIENT=phpredis
REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=hello@example.com
MAIL_FROM_NAME="${APP_NAME}"

VITE_APP_NAME="${APP_NAME}"
```

3. Suba os containers:

```bash
docker compose up -d --build
```

4. Instale dependencias PHP:

```bash
docker compose exec app composer install
```

5. Gere a chave da aplicacao:

```bash
docker compose exec app php artisan key:generate
```

6. Rode as migrations e seeders:

```bash
docker compose exec app php artisan migrate --seed
```

7. Instale dependencias frontend:

```bash
docker compose exec app npm install
```

## URLs

- Aplicacao: `http://localhost:8080`
- Filament Admin: `http://localhost:8080/admin`
- Mailpit: `http://localhost:8025`
- Vite: `http://localhost:5173`

## Comandos Docker

Subir ambiente:

```bash
docker compose up -d
```

Subir ambiente com Vite:

```bash
docker compose --profile dev up -d
```

Parar containers:

```bash
docker compose down
```

Reconstruir imagens:

```bash
docker compose build
```

Ver logs:

```bash
docker compose logs -f
```

Entrar no container PHP:

```bash
docker compose exec app bash
```

Ver status dos servicos:

```bash
docker compose ps
```

## Laravel

Comandos comuns:

```bash
docker compose exec app php artisan migrate
docker compose exec app php artisan migrate:fresh --seed
docker compose exec app php artisan optimize:clear
docker compose exec app php artisan config:cache
docker compose exec app php artisan route:cache
docker compose exec app php artisan view:cache
```

Criar classes:

```bash
docker compose exec app php artisan make:model Nome -mfs
docker compose exec app php artisan make:controller NomeController
docker compose exec app php artisan make:policy NomePolicy --model=Nome
docker compose exec app php artisan make:job NomeJob
docker compose exec app php artisan make:command NomeCommand
```

## Filament

O Filament esta instalado no projeto Laravel em `src/`.

Comandos uteis:

```bash
docker compose exec app php artisan filament:about
docker compose exec app php artisan filament:make-user
docker compose exec app php artisan filament:make-resource Nome --generate
docker compose exec app php artisan filament:optimize
docker compose exec app php artisan filament:optimize-clear
```

Para criar um usuario administrador:

```bash
docker compose exec app php artisan filament:make-user
```

## Queue e Scheduler

O servico `queue` roda automaticamente:

```bash
php artisan queue:work redis --sleep=3 --tries=3 --timeout=90
```

O servico `scheduler` roda automaticamente:

```bash
php artisan schedule:work
```

Para executar manualmente:

```bash
docker compose exec app php artisan queue:work redis
docker compose exec app php artisan schedule:run
```

## Frontend com Vite

Instalar dependencias:

```bash
docker compose exec app npm install
```

Rodar Vite em desenvolvimento:

```bash
docker compose --profile dev up -d node
```

Gerar build de producao:

```bash
docker compose exec app npm run build
```

## Testes e Qualidade

Rodar testes:

```bash
docker compose exec app php artisan test
```

Rodar Pint:

```bash
docker compose exec app ./vendor/bin/pint
```

## Banco de Dados

O MySQL usa:

- charset `utf8mb4`
- collation `utf8mb4_unicode_ci`
- timezone `-03:00`
- volume Docker `mysql-data`
- pasta local `docker/mysql/backups` para dumps manuais

Backup manual:

```bash
docker compose exec mysql mysqldump -uroot -proot instrucao > docker/mysql/backups/instrucao.sql
```

Restaurar backup:

```bash
docker compose exec -T mysql mysql -uroot -proot instrucao < docker/mysql/backups/instrucao.sql
```

## Xdebug

Xdebug e opcional. Para ativar no build:

```bash
INSTALL_XDEBUG=true docker compose build app
docker compose up -d
```

As configuracoes ficam em `docker/php/php.ini`.

## Producao

Existe um override inicial para producao:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build
```

Antes de usar em producao, revise:

- `APP_ENV=production`
- `APP_DEBUG=false`
- senhas fortes para MySQL
- `APP_KEY` gerada e persistida
- HTTPS no proxy ou load balancer
- backups automaticos
- politica de logs e retencao
- permissoes de arquivos em `storage` e `bootstrap/cache`

## Solucao de Problemas

Limpar cache Laravel:

```bash
docker compose exec app php artisan optimize:clear
```

Recriar banco local do zero:

```bash
docker compose exec app php artisan migrate:fresh --seed
```

Reinstalar dependencias:

```bash
docker compose exec app composer install
docker compose exec app npm install
```

Ver logs do Nginx, PHP, Queue ou Scheduler:

```bash
docker compose logs -f nginx
docker compose logs -f app
docker compose logs -f queue
docker compose logs -f scheduler
```
