# WordPress + Sage — Runbook Completo (Ubuntu 22.04)

Guia completo para configurar um ambiente de desenvolvimento WordPress com tema Sage do zero.

---

## 1. Atualizar o sistema

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. PHP 8.2

```bash
# Adicionar repositório do PHP
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update

# Instalar PHP e extensões necessárias
sudo apt install -y php8.2 php8.2-cli php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl php8.2-zip libapache2-mod-php8.2

# Verificar
php -v
```

---

## 3. Apache 2.4

```bash
# Instalar
sudo apt install -y apache2

# Ativar mod_php e mod_rewrite
sudo a2enmod php8.2
sudo a2enmod rewrite

# Iniciar e habilitar no boot
sudo systemctl enable apache2
sudo systemctl start apache2

# Verificar
apache2 -v
```

---

## 4. MariaDB

```bash
# Instalar
sudo apt install -y mariadb-server

# Rodar setup de segurança
sudo mysql_secure_installation

# Verificar
mariadb --version
```

---

## 5. Composer

```bash
# Baixar instalador
curl -sS https://getcomposer.org/installer | php

# Mover para global
sudo mv composer.phar /usr/local/bin/composer

# Verificar
composer --version
```

---

## 6. Node + npm via NVM

```bash
# Instalar NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Recarregar shell
source ~/.bashrc

# Instalar Node 24
nvm install 24
nvm use 24

# Verificar
node -v && npm -v
```

---

## 7. WP-CLI

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

## 8. Banco de Dados (MariaDB)

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

## 9. Pasta do Projeto

```bash
sudo mkdir /var/www/evolu-e
sudo chown -R SEU_USUARIO:SEU_USUARIO /var/www/evolu-e
```

> Substitua `SEU_USUARIO` pelo seu usuário do sistema (ex: `evag`).

---

## 10. Virtual Host no Apache

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

Ativar site e reiniciar Apache:

```bash
sudo a2ensite evolu-e.conf
sudo systemctl restart apache2
```

---

## 11. DNS Local

```bash
sudo nano /etc/hosts
```

Adicionar ao final:

```
127.0.0.1       evolu-e.local
```

---

## 12. WordPress via WP-CLI

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

## 13. Tema Sage

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

## 14. Configurar Vite

Editar `vite.config.js` na pasta do tema:

```js
// Antes
if (! process.env.APP_URL) {
  process.env.APP_URL = 'http://example.test';
}

// Depois
if (! process.env.APP_URL) {
  process.env.APP_URL = 'http://evolu-e.local';
}
```

---

## 15. Permissões de Cache

```bash
sudo mkdir -p /var/www/evolu-e/wp-content/cache
sudo chown -R SEU_USUARIO:www-data /var/www/evolu-e/wp-content/cache
sudo chmod -R 775 /var/www/evolu-e/wp-content/cache

# Inicializar Acorn
wp acorn optimize --path=/var/www/evolu-e
```

---

## 16. Página Inicial Estática

No painel WordPress (`/wp-admin/options-reading.php`):

- **Sua página inicial exibe** → Uma página estática
- **Página inicial** → selecionar uma página existente

---

## 17. Desenvolvimento

```bash
# Rodar servidor de desenvolvimento
cd /var/www/evolu-e/wp-content/themes/evolu-e-theme
npm run dev

# Build para produção
npm run build
```

---

## 18. Estrutura de Views (Blade)

```
resources/views/
├── layouts/
│   └── app.blade.php       # Layout base
├── sections/
│   ├── header.blade.php
│   ├── footer.blade.php
│   └── sidebar.blade.php
├── partials/
├── front-page.blade.php    # Página inicial
├── page.blade.php
└── index.blade.php
```

### Página inicial customizada

`resources/views/front-page.blade.php`:

```blade
@extends('layouts.app')

@section('content')

  <section id="hero">
    <h1>Título</h1>
  </section>

@endsection
```

> O nome `front-page` é reservado pelo WordPress — ele usa automaticamente esse template para a página inicial.

---

## 19. CSS Base (Tailwind v4)

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

> **Tailwind v4:** Não existe mais `tailwind.config.js`. As configurações ficam no `@theme {}` dentro do CSS.

---

## Problemas comuns

### `Call to undefined function mysqli_init()`
```bash
sudo apt install php8.2-mysql && sudo systemctl restart apache2
```

### `Call to undefined function mb_split()`
```bash
sudo apt install php8.2-mbstring && sudo systemctl restart apache2
```

### `Permission denied` na pasta de cache
```bash
sudo chown -R SEU_USUARIO:www-data /var/www/evolu-e/wp-content/cache
sudo chmod -R 775 /var/www/evolu-e/wp-content/cache
```

### Vite mostrando `APP_URL: http://example.test`
Editar `vite.config.js` conforme passo 14.

---

## Deploy (Hostgator)

1. Rodar `npm run build` na pasta do tema
2. Enviar via FTP:
   - Pasta `public/build/` do tema
   - Demais arquivos do tema (exceto `node_modules` e `vendor`)
3. WordPress e banco devem estar configurados no Hostgator previamente

---

*Gerado durante o desenvolvimento do freela Evolu-e — março de 2026*
