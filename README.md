# WordPress + Sage — Runbook de Setup Local

Guia completo para configurar um ambiente de desenvolvimento WordPress com tema Sage no Ubuntu 22.04.

---

## Pré-requisitos

- Ubuntu 22.04
- PHP 8.2 + extensões (`php8.2-mysql`, `php8.2-mbstring`)
- Composer 2.x
- Node 24 + npm 11 (via NVM)
- Apache 2.4 com `mod_php8.2`
- MariaDB 10.6

---

## 1. WP-CLI

```bash
# Baixar
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar

# Dar permissão de execução
chmod +x wp-cli.phar

# Tornar global
sudo mv wp-cli.phar /usr/local/bin/wp

# Verificar
wp --version
```

---

## 2. Banco de Dados (MariaDB)

```bash
sudo mariadb
```

Dentro do MariaDB:

```sql
CREATE DATABASE evolu_e;
CREATE USER 'evolu_e_user'@'localhost' IDENTIFIED BY 'sua_senha';
GRANT ALL PRIVILEGES ON evolu_e.* TO 'evolu_e_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 3. Pasta do Projeto

```bash
sudo mkdir /var/www/evolu-e
sudo chown -R evag:evag /var/www/evolu-e
```

---

## 4. Virtual Host no Apache

```bash
sudo nano /etc/apache2/sites-available/evolu-e.conf
```

Conteúdo:

```apache
<VirtualHost *:80>
    ServerName evolu-e.local
    DocumentRoot /var/www/evolu-e

    <Directory /var/www/evolu-e>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Ativar site e módulo rewrite:

```bash
sudo a2ensite evolu-e.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## 5. DNS Local

```bash
sudo nano /etc/hosts
```

Adicionar:

```
127.0.0.1       evolu-e.local
```

---

## 6. WordPress via WP-CLI

```bash
# Baixar WordPress em pt_BR
wp core download --path=/var/www/evolu-e --locale=pt_BR

# Criar wp-config.php
wp config create \
  --path=/var/www/evolu-e \
  --dbname=evolu_e \
  --dbuser=evolu_e_user \
  --dbpass=sua_senha \
  --dbhost=localhost

# Instalar WordPress
wp core install \
  --path=/var/www/evolu-e \
  --url=http://evolu-e.local \
  --title="Nome do Site" \
  --admin_user=admin \
  --admin_password=admin123 \
  --admin_email=seu@email.com
```

---

## 7. Tema Sage

```bash
cd /var/www/evolu-e/wp-content/themes

# Instalar Sage via Composer
composer create-project roots/sage evolu-e-theme

# Instalar dependências JS
cd evolu-e-theme && npm install

# Ativar tema
wp theme activate evolu-e-theme --path=/var/www/evolu-e
```

---

## 8. Configurar Vite

Editar `vite.config.js` na pasta do tema:

```js
// Trocar
if (! process.env.APP_URL) {
  process.env.APP_URL = 'http://example.test';
}

// Por
if (! process.env.APP_URL) {
  process.env.APP_URL = 'http://evolu-e.local';
}
```

---

## 9. Permissões de Cache

```bash
sudo mkdir -p /var/www/evolu-e/wp-content/cache
sudo chown -R evag:www-data /var/www/evolu-e/wp-content/cache
sudo chmod -R 775 /var/www/evolu-e/wp-content/cache
```

Inicializar Acorn (framework do Sage):

```bash
wp acorn optimize --path=/var/www/evolu-e
```

---

## 10. Página Inicial Estática

No painel WordPress (`/wp-admin/options-reading.php`):

- Sua página inicial exibe → **Uma página estática**
- Página inicial → selecionar uma página existente

---

## 11. Desenvolvimento

### Rodar servidor de desenvolvimento

```bash
cd /var/www/evolu-e/wp-content/themes/evolu-e-theme
npm run dev
```

### Build para produção (deploy)

```bash
npm run build
```

Os arquivos compilados ficam em `public/build/` — esses vão para o Hostgator.

---

## 12. Estrutura de Views (Blade)

```
resources/views/
├── layouts/
│   └── app.blade.php       # Layout base (html, head, body)
├── sections/
│   ├── header.blade.php
│   ├── footer.blade.php
│   └── sidebar.blade.php
├── partials/
├── front-page.blade.php    # Página inicial (hierarquia WordPress)
├── page.blade.php
└── index.blade.php
```

### Criar página inicial customizada

`resources/views/front-page.blade.php`:

```blade
@extends('layouts.app')

@section('content')

  <section id="hero">
    <h1>Título</h1>
  </section>

  {{-- Outras seções... --}}

@endsection
```

---

## 13. CSS Base (Tailwind v4)

`resources/css/app.css`:

```css
@import "tailwindcss" theme(static);
@source "../../app/**/*.php";
@source "../**/*.blade.php";
@source "../**/*.js";

@theme {
  --color-primary: #0a0a0a;
  --color-accent: #c8f04d;
  --color-text: #f5f5f5;
  --color-muted: #888888;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html { scroll-behavior: smooth; }

body {
  background-color: #0a0a0a;
  color: #f5f5f5;
  font-family: 'Inter', sans-serif;
}

/* Ocultar elementos padrão do Sage */
.sidebar, .content-info, .banner { display: none; }
```

> **Nota:** No Tailwind v4, não existe mais `tailwind.config.js`. As configurações de tema ficam no `@theme {}` dentro do próprio CSS.

---

## Extensões PHP necessárias

```bash
sudo apt install php8.2-mysql
sudo apt install php8.2-mbstring
sudo systemctl restart apache2
```

---

## Fluxo de Deploy (Hostgator)

1. Rodar `npm run build` na pasta do tema
2. Enviar via FTP para o Hostgator:
   - Pasta `public/build/` do tema
   - Demais arquivos do tema (exceto `node_modules` e `vendor`)
3. O WordPress e banco já devem estar configurados no Hostgator

---

*Gerado durante o desenvolvimento do freela Evolu-e — março de 2026*
