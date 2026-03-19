# WordPress + Sage — Runbook Completo (Ubuntu 22.04)

Boilerplate de landing page com WordPress + Sage, Tailwind v4, formulário com banco de dados e menu responsivo.

---

## 1. Atualizar o sistema

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. PHP 8.2

```bash
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update

sudo apt install -y php8.2 php8.2-cli php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl php8.2-zip libapache2-mod-php8.2

php -v
```

---

## 3. Apache 2.4

```bash
sudo apt install -y apache2
sudo a2enmod php8.2
sudo a2enmod rewrite
sudo systemctl enable apache2
sudo systemctl start apache2
```

---

## 4. MariaDB

```bash
sudo apt install -y mariadb-server
sudo mysql_secure_installation
```

---

## 5. Composer

```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer --version
```

---

## 6. Node + npm via NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 24
nvm use 24
node -v && npm -v
```

---

## 7. WP-CLI

```bash
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp
wp --version
```

---

## 8. Banco de Dados

```bash
sudo mariadb
```

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

---

## 10. Virtual Host no Apache

```bash
sudo nano /etc/apache2/sites-available/evolu-e.conf
```

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

```bash
sudo a2ensite evolu-e.conf
sudo systemctl restart apache2
```

---

## 11. DNS Local

```bash
sudo nano /etc/hosts
```

Adicionar:

```
127.0.0.1       evolu-e.local
```

---

## 12. .htaccess

Criar `/var/www/evolu-e/.htaccess`:

```apache
# BEGIN WordPress
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
# END WordPress
```

> Sem esse arquivo o WordPress não processa URLs amigáveis e a REST API retorna 404.

---

## 13. WordPress via WP-CLI

```bash
wp core download --path=/var/www/evolu-e --locale=pt_BR

wp config create \
  --path=/var/www/evolu-e \
  --dbname=evolu_e \
  --dbuser=evolu_e_user \
  --dbpass=sua_senha \
  --dbhost=localhost

wp core install \
  --path=/var/www/evolu-e \
  --url=http://evolu-e.local \
  --title="Nome do Site" \
  --admin_user=admin \
  --admin_password=admin123 \
  --admin_email=seu@email.com
```

---

## 14. Tema Sage

```bash
cd /var/www/evolu-e/wp-content/themes
composer create-project roots/sage evolu-e-theme
cd evolu-e-theme && npm install
wp theme activate evolu-e-theme --path=/var/www/evolu-e
```

---

## 15. Configurar Vite

Editar `vite.config.js`:

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

## 16. Permissões de Cache

```bash
sudo mkdir -p /var/www/evolu-e/wp-content/cache
sudo chown -R SEU_USUARIO:www-data /var/www/evolu-e/wp-content/cache
sudo chmod -R 775 /var/www/evolu-e/wp-content/cache

wp acorn optimize --path=/var/www/evolu-e
```

---

## 17. Página Inicial Estática

Em `/wp-admin/options-reading.php`:

- Sua página inicial exibe → **Uma página estática**
- Página inicial → selecionar uma página existente

---

## 18. Desenvolvimento

```bash
cd /var/www/evolu-e/wp-content/themes/evolu-e-theme
npm run dev   # desenvolvimento com hot reload
npm run build # build para produção
```

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

*, *::before, *::after {
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

/* Botão principal */
.btn-primary {
  background-color: #c8f04d;
  color: #0a0a0a;
  font-weight: 700;
  padding: 1rem 2rem;
  border-radius: 9999px;
  transition: opacity 0.2s;
  cursor: pointer;
  display: inline-block;
}

.btn-primary:hover { opacity: 0.9; }

/* Links do menu WordPress */
.nav li { list-style: none; }
.nav a { color: #888888; text-decoration: none; transition: color 0.2s; }
.nav a:hover { color: #ffffff; }

/* Selects */
select {
  color: #555555;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath fill='%23888888' d='M6 8L1 3h10z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 1.25rem center;
  padding-right: 3rem;
}

select option { background-color: #0a0a0a; color: #f5f5f5; }
select:has(option:checked:not([value=""])) { color: #f5f5f5; }

/* Compensar barra admin do WordPress */
#site-header { top: 32px; }

@media screen and (max-width: 782px) {
  #site-header { top: 46px; }
}
```

> **Tailwind v4:** Não existe mais `tailwind.config.js`. As configurações ficam no `@theme {}` dentro do CSS.
> **Atenção:** Não use `* { margin: 0; padding: 0 }` — isso sobrescreve as classes do Tailwind.

---

## 20. Estrutura de Views (Blade)

```
resources/views/
├── layouts/
│   └── app.blade.php           # Layout base
├── sections/
│   ├── header.blade.php        # Menu fixo com sandwich mobile
│   ├── footer.blade.php
│   └── sidebar.blade.php
├── partials/
└── front-page.blade.php        # Página inicial (hierarquia WordPress)
```

---

## 21. Header com Menu do WordPress

`resources/views/sections/header.blade.php`:

```blade
<header id="site-header" class="fixed left-0 w-full z-50 px-6 py-4 flex justify-between items-center bg-[#0a0a0a]/90 backdrop-blur-sm border-b border-[#222222]">

  <a href="{{ home_url('/') }}" class="text-xl font-bold text-white hover:text-[#c8f04d] transition">
    Evolu-e
  </a>

  <nav class="hidden md:flex items-center gap-8 text-sm text-[#888888]">
    {!! wp_nav_menu([
      'theme_location' => 'primary_navigation',
      'menu_class'     => 'flex gap-8 items-center',
      'container'      => false,
      'echo'           => false,
      'depth'          => 1,
      'fallback_cb'    => false,
    ]) !!}
  </nav>

  <button id="menu-toggle" class="md:hidden flex flex-col gap-1.5 p-2" aria-label="Menu">
    <span class="block w-6 h-0.5 bg-white transition-all"></span>
    <span class="block w-6 h-0.5 bg-white transition-all"></span>
    <span class="block w-6 h-0.5 bg-white transition-all"></span>
  </button>

</header>

<div id="mobile-menu" class="fixed left-0 w-full h-screen bg-[#0a0a0a] z-40 flex-col items-center justify-center gap-8 text-2xl font-bold hidden"
  style="top: 0; transform: translateY(-100%); transition: transform 0.4s ease;">
  {!! wp_nav_menu([
    'theme_location' => 'primary_navigation',
    'menu_class'     => 'flex flex-col items-center gap-8',
    'container'      => false,
    'echo'           => false,
    'depth'          => 1,
    'fallback_cb'    => false,
  ]) !!}
</div>

<script>
  const toggle = document.getElementById('menu-toggle');
  const mobileMenu = document.getElementById('mobile-menu');

  toggle.addEventListener('click', () => {
    if (mobileMenu.classList.contains('hidden')) {
      mobileMenu.classList.remove('hidden');
      mobileMenu.classList.add('flex');
      setTimeout(() => mobileMenu.style.transform = 'translateY(0)', 10);
    } else {
      mobileMenu.style.transform = 'translateY(-100%)';
      setTimeout(() => {
        mobileMenu.classList.add('hidden');
        mobileMenu.classList.remove('flex');
      }, 400);
    }
  });

  mobileMenu.querySelectorAll('a').forEach(link => {
    link.addEventListener('click', () => {
      mobileMenu.style.transform = 'translateY(-100%)';
      setTimeout(() => {
        mobileMenu.classList.add('hidden');
        mobileMenu.classList.remove('flex');
      }, 400);
    });
  });
</script>
```

> Para adicionar itens ao menu: `/wp-admin/nav-menus.php` → Links personalizados → usar URLs como `http://evolu-e.local/#sobre`.

---

## 22. Formulário com Banco de Dados

### Controller

`app/Http/Controllers/ContatoController.php`:

```php
<?php

namespace App\Http\Controllers;

use WP_REST_Request;
use WP_REST_Response;

class ContatoController
{
    public static function criarTabela(): void
    {
        global $wpdb;
        $tabela  = $wpdb->prefix . 'evolu_e_contatos';
        $charset = $wpdb->get_charset_collate();

        $sql = "CREATE TABLE IF NOT EXISTS {$tabela} (
            id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
            nome VARCHAR(100) NOT NULL,
            email VARCHAR(100) NOT NULL,
            whatsapp VARCHAR(20),
            estado VARCHAR(2),
            cidade VARCHAR(100),
            aceita_novidades TINYINT(1) DEFAULT 0,
            criado_em DATETIME DEFAULT CURRENT_TIMESTAMP
        ) {$charset};";

        require_once ABSPATH . 'wp-admin/includes/upgrade.php';
        dbDelta($sql);
    }

    public static function registrarRota(): void
    {
        register_rest_route('evolu-e/v1', '/contato', [
            'methods'             => 'POST',
            'callback'            => [self::class, 'salvar'],
            'permission_callback' => '__return_true',
        ]);
    }

    public static function salvar(WP_REST_Request $request): WP_REST_Response
    {
        global $wpdb;

        $nome             = sanitize_text_field($request->get_param('nome'));
        $email            = sanitize_email($request->get_param('email'));
        $whatsapp         = sanitize_text_field($request->get_param('whatsapp'));
        $estado           = sanitize_text_field($request->get_param('estado'));
        $cidade           = sanitize_text_field($request->get_param('cidade'));
        $aceita_novidades = (int) $request->get_param('aceita_novidades');

        if (empty($nome) || empty($email)) {
            return new WP_REST_Response(['erro' => 'Nome e email são obrigatórios.'], 400);
        }

        $wpdb->insert($wpdb->prefix . 'evolu_e_contatos', [
            'nome'             => $nome,
            'email'            => $email,
            'whatsapp'         => $whatsapp,
            'estado'           => $estado,
            'cidade'           => $cidade,
            'aceita_novidades' => $aceita_novidades,
        ]);

        return new WP_REST_Response(['sucesso' => 'Contato registrado com sucesso!'], 201);
    }
}
```

### Registrar no ThemeServiceProvider

`app/Providers/ThemeServiceProvider.php`:

```php
<?php

namespace App\Providers;

use Roots\Acorn\Sage\SageServiceProvider;
use App\Http\Controllers\ContatoController;

class ThemeServiceProvider extends SageServiceProvider
{
    public function register()
    {
        parent::register();
    }

    public function boot()
    {
        parent::boot();
        add_action('after_setup_theme', [ContatoController::class, 'criarTabela']);
        add_action('rest_api_init', [ContatoController::class, 'registrarRota']);
    }
}
```

Após qualquer alteração no PHP:

```bash
wp acorn optimize --path=/var/www/evolu-e
```

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

### REST API retorna 404
Verificar se o `.htaccess` existe em `/var/www/evolu-e/`. Se não existir, criar conforme passo 12.

### Vite mostrando `APP_URL: http://example.test`
Editar `vite.config.js` conforme passo 15.

### Padding/margin do Tailwind não funciona
Não usar `* { margin: 0; padding: 0 }` no CSS — sobrescreve o Tailwind. Usar apenas `box-sizing: border-box`.

---

## Deploy (Hostgator)

1. Rodar `npm run build` na pasta do tema
2. Enviar via FTP:
   - Pasta `public/build/` do tema
   - Demais arquivos do tema (exceto `node_modules` e `vendor`)
3. WordPress e banco devem estar configurados no Hostgator previamente
4. Criar o `.htaccess` no Hostgator se não existir

---

*Gerado durante o desenvolvimento do boilerplate Evolu-e — março de 2026*
