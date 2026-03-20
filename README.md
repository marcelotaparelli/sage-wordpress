# WordPress + Sage — Runbook Completo (Ubuntu 22.04)

Boilerplate de landing page com WordPress + Sage, Tailwind v4, CPTs, formulário com banco de dados, segurança e painel admin completo.

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

## 14. Permalinks

Em `/wp-admin/options-permalink.php` seleciona **"Nome do post"** e salva.

---

## 15. Permissões de uploads

```bash
sudo chown -R www-data:www-data /var/www/evolu-e/wp-content/uploads
sudo chmod -R 775 /var/www/evolu-e/wp-content/uploads
```

> Em produção (Hostgator), verificar se a pasta `uploads` tem permissão de escrita via cPanel.

---

## 16. Tema Sage

```bash
cd /var/www/evolu-e/wp-content/themes
composer create-project roots/sage evolu-e-theme
cd evolu-e-theme && npm install
wp theme activate evolu-e-theme --path=/var/www/evolu-e
```

---

## 17. Configurar Vite

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

## 18. Permissões de Cache

```bash
sudo mkdir -p /var/www/evolu-e/wp-content/cache
sudo chown -R SEU_USUARIO:www-data /var/www/evolu-e/wp-content/cache
sudo chmod -R 775 /var/www/evolu-e/wp-content/cache

wp acorn optimize --path=/var/www/evolu-e
```

> Sempre que alterar arquivos PHP no Sage, rodar `wp acorn optimize` e `composer dump-autoload`.

---

## 19. Página Inicial Estática

Em `/wp-admin/options-reading.php`:

- Sua página inicial exibe → **Uma página estática**
- Página inicial → selecionar uma página existente

---

## 20. Menu no WordPress

Em `/wp-admin/nav-menus.php`:

1. Criar novo menu chamado "Menu Principal"
2. Em **Links personalizados** adicionar URLs como `http://evolu-e.local/#sobre`
3. Em **Local do menu** marcar **Primary Navigation**
4. Salvar

---

## 21. Desenvolvimento

```bash
cd /var/www/evolu-e/wp-content/themes/evolu-e-theme
npm run dev    # desenvolvimento com hot reload
npm run build  # build para produção
```

---

## 22. CSS Base (Tailwind v4)

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

/* Offset do scroll para compensar o header fixo */
[id] { scroll-margin-top: 80px; }

/* Padding top para compensar header fixo */
#main { padding-top: 80px; }

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
.admin-bar #site-header { top: 32px; }

@media screen and (max-width: 782px) {
  .admin-bar #site-header { top: 46px; }
}

/* Animações de entrada */
.animate-on-scroll {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 0.5s ease, transform 0.5s ease;
}

.animate-on-scroll.visible {
  opacity: 1;
  transform: translateY(0);
}

/* Prose - conteúdo de páginas estáticas */
.prose h2 { color: #f5f5f5; font-size: 1.5rem; font-weight: 700; margin-top: 2rem; margin-bottom: 1rem; }
.prose h3 { color: #f5f5f5; font-size: 1.2rem; font-weight: 600; margin-top: 1.5rem; margin-bottom: 0.5rem; }
.prose p  { margin-bottom: 1rem; }
.prose a  { color: #c8f04d; text-decoration: underline; }
.prose ul { list-style: disc; padding-left: 1.5rem; margin-bottom: 1rem; }
.prose li { margin-bottom: 0.25rem; }
```

> **Tailwind v4:** Não existe mais `tailwind.config.js`. As configurações ficam no `@theme {}`.
> **Atenção:** Não use `* { margin: 0; padding: 0 }` — sobrescreve as classes do Tailwind.

---

## 23. Estrutura de Views (Blade)

```
resources/views/
├── layouts/
│   └── app.blade.php                          # Layout base
├── sections/
│   ├── header.blade.php                       # Menu fixo com sandwich mobile
│   ├── footer.blade.php
│   └── sidebar.blade.php
├── partials/
├── front-page.blade.php                       # Landing page
├── page.blade.php                             # Páginas genéricas
└── page-politica-de-privacidade.blade.php     # Template específico por slug
```

> O nome `page-{slug}.blade.php` é usado automaticamente pelo WordPress para a página com aquele slug.

---

## 24. Custom Post Types

`app/Http/Controllers/PostTypesController.php`:

```php
<?php

namespace App\Http\Controllers;

class PostTypesController
{
    public static function registrar(): void
    {
        register_post_type('servico', [
            'labels'        => ['name' => 'Serviços', 'singular_name' => 'Serviço'],
            'public'        => false,
            'show_ui'       => true,
            'show_in_menu'  => true,
            'menu_icon'     => 'dashicons-hammer',
            'supports'      => ['title', 'editor', 'thumbnail', 'excerpt'],
            'menu_position' => 7,
        ]);

        register_post_type('cliente', [
            'labels'        => ['name' => 'Clientes', 'singular_name' => 'Cliente'],
            'public'        => false,
            'show_ui'       => true,
            'show_in_menu'  => true,
            'menu_icon'     => 'dashicons-groups',
            'supports'      => ['title', 'thumbnail'],
            'menu_position' => 8,
        ]);

        register_post_type('membro', [
            'labels'        => ['name' => 'Equipe', 'singular_name' => 'Membro'],
            'public'        => false,
            'show_ui'       => true,
            'show_in_menu'  => true,
            'menu_icon'     => 'dashicons-businessman',
            'supports'      => ['title', 'thumbnail', 'excerpt'],
            'menu_position' => 9,
        ]);

        register_post_type('depoimento', [
            'labels'        => ['name' => 'Depoimentos', 'singular_name' => 'Depoimento'],
            'public'        => false,
            'show_ui'       => true,
            'show_in_menu'  => true,
            'menu_icon'     => 'dashicons-format-quote',
            'supports'      => ['title', 'editor', 'thumbnail', 'excerpt'],
            'menu_position' => 10,
        ]);
    }
}
```

> Para Serviços: título = nome, conteúdo = descrição, excerpt = ícone emoji, thumbnail = imagem.
> Para Equipe: título = nome, excerpt = cargo, thumbnail = foto.
> Para Depoimentos: título = nome, conteúdo = depoimento, excerpt = cargo/empresa, thumbnail = foto.
> Para Clientes: título = nome, thumbnail = logo.

---

## 25. View Composer (variáveis globais)

`app/View/Composers/App.php`:

```php
<?php

namespace App\View\Composers;

use Roots\Acorn\View\Composer;

class App extends Composer
{
    protected static $views = ['*'];

    public function siteName(): string
    {
        return get_bloginfo('name', 'display');
    }

    public function empresa(): string { return get_option('evolu_e_empresa', 'Evolu-e'); }
    public function whatsapp(): string { return get_option('evolu_e_whatsapp', '5519999999999'); }
    public function email(): string { return get_option('evolu_e_email', 'contato@evolu-e.com.br'); }
    public function instagram(): string { return get_option('evolu_e_instagram', '#'); }
    public function linkedin(): string { return get_option('evolu_e_linkedin', '#'); }
    public function twitter(): string { return get_option('evolu_e_twitter', '#'); }
    public function facebook(): string { return get_option('evolu_e_facebook', '#'); }
    public function googleAnalyticsId(): string { return get_option('evolu_e_google_analytics_id', ''); }
}
```

---

## 26. ThemeServiceProvider

`app/Providers/ThemeServiceProvider.php`:

```php
<?php

namespace App\Providers;

use Roots\Acorn\Sage\SageServiceProvider;
use App\Http\Controllers\ContatoController;
use App\Http\Controllers\AdminContatosController;
use App\Http\Controllers\AdminConfiguracoesController;
use App\Http\Controllers\AdminConteudoController;
use App\Http\Controllers\PostTypesController;

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
        add_action('init', [PostTypesController::class, 'registrar']);

        add_action('admin_menu', [AdminContatosController::class, 'registrarMenu']);
        add_action('admin_menu', [AdminConfiguracoesController::class, 'registrarMenu']);
        add_action('admin_menu', function() {
            AdminConteudoController::registrarMenu();
        });

        add_action('admin_init', function() {
            if (
                isset($_GET['page'], $_GET['exportar']) &&
                $_GET['page'] === 'evolu-e-contatos' &&
                $_GET['exportar'] === 'csv'
            ) {
                AdminContatosController::exportarCsv();
            }
        });
    }
}
```

---

## 27. Formulário com Banco de Dados e Segurança

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
            ip VARCHAR(45),
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

        // Honeypot
        if (!empty($request->get_param('honeypot'))) {
            return new WP_REST_Response(['erro' => 'Envio inválido.'], 400);
        }

        // Rate limiting: máximo 3 envios por IP por hora
        $ip        = $_SERVER['REMOTE_ADDR'];
        $tabela    = $wpdb->prefix . 'evolu_e_contatos';
        $uma_hora  = date('Y-m-d H:i:s', strtotime('-1 hour'));
        $tentativas = $wpdb->get_var($wpdb->prepare(
            "SELECT COUNT(*) FROM {$tabela} WHERE ip = %s AND criado_em > %s",
            $ip, $uma_hora
        ));

        if ($tentativas >= 3) {
            return new WP_REST_Response(['erro' => 'Muitas tentativas. Tente novamente mais tarde.'], 429);
        }

        $nome             = sanitize_text_field($request->get_param('nome'));
        $email            = sanitize_email($request->get_param('email'));
        $whatsapp         = sanitize_text_field($request->get_param('whatsapp'));
        $estado           = sanitize_text_field($request->get_param('estado'));
        $cidade           = sanitize_text_field($request->get_param('cidade'));
        $aceita_novidades = (int) $request->get_param('aceita_novidades');

        if (empty($nome) || empty($email)) {
            return new WP_REST_Response(['erro' => 'Nome e email são obrigatórios.'], 400);
        }

        $wpdb->insert($tabela, [
            'nome'             => $nome,
            'email'            => $email,
            'whatsapp'         => $whatsapp,
            'estado'           => $estado,
            'cidade'           => $cidade,
            'aceita_novidades' => $aceita_novidades,
            'ip'               => $ip,
        ]);

        return new WP_REST_Response(['sucesso' => 'Contato registrado com sucesso!'], 201);
    }
}
```

---

## 28. Painel Admin — Configurações da Empresa

Campos configuráveis em `/wp-admin/admin.php?page=evolu-e-configuracoes`:

- Nome da Empresa
- WhatsApp (formato: `5519999999999`)
- E-mail de contato
- Instagram, LinkedIn, Twitter/X, Facebook (URLs completas)
- Google Analytics ID (ex: `G-XXXXXXXXXX`)

---

## 29. Painel Admin — Conteúdo da LP

Campos configuráveis em `/wp-admin/admin.php?page=evolu-e-conteudo`:

**Hero:** pré-título, título parte 1, título destaque, subtítulo, texto do botão CTA

**Sobre:** título parte 1, título destaque, parágrafo 1, parágrafo 2, 4 métricas (número + label)

**Contato:** pré-título, título parte 1, título destaque, subtítulo

---

## 30. Google Analytics + Cookie Consent

No `layouts/app.blade.php`, antes do `</head>`:

```blade
@if($googleAnalyticsId)
  <script async src="https://www.googletagmanager.com/gtag/js?id={{ $googleAnalyticsId }}"></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}

    gtag('consent', 'default', { analytics_storage: 'denied' });
    gtag('js', new Date());
    gtag('config', '{{ $googleAnalyticsId }}');

    if (localStorage.getItem('evolu_e_cookie_consent') === 'aceito') {
      gtag('consent', 'update', { analytics_storage: 'granted' });
    }
  </script>
@endif
```

O cookie consent aparece automaticamente quando o GA está configurado.

---

## Segurança do Formulário

- **Honeypot** — campo invisível; se preenchido por bot, descarta o envio
- **Rate limiting** — máximo 3 envios por IP por hora
- **Sanitização** — `sanitize_text_field()` e `sanitize_email()` em todos os campos
- **XSS** — `esc_html()` ao exibir dados no admin

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

### `Permission denied` na pasta de uploads
```bash
sudo chown -R www-data:www-data /var/www/evolu-e/wp-content/uploads
sudo chmod -R 775 /var/www/evolu-e/wp-content/uploads
```

### REST API retorna 404
Verificar se o `.htaccess` existe em `/var/www/evolu-e/`. Se não, criar conforme passo 12.

### Vite mostrando `APP_URL: http://example.test`
Editar `vite.config.js` conforme passo 17.

### Padding/margin do Tailwind não funciona
Não usar `* { margin: 0; padding: 0 }` — sobrescreve o Tailwind. Usar apenas `box-sizing: border-box`.

### Classe PHP não encontrada após criar novo controller
```bash
composer dump-autoload
wp acorn optimize --path=/var/www/evolu-e
```

### Coluna não existe na tabela após atualizar o Controller
```bash
sudo mariadb -e "ALTER TABLE evolu_e.wp_evolu_e_contatos ADD COLUMN nome_coluna TIPO AFTER coluna_anterior;"
```

---

## Git

```bash
# Iniciar repositório
git init
git branch -m main
git add .
git commit -m "feat: boilerplate inicial"

# Criar repositório no GitHub
gh repo create nome-do-repo --private --source=. --remote=origin --push

# Commits seguintes
git add .
git commit -m "feat: descrição"
git push
```

---

## Deploy (Hostgator)

1. Rodar `npm run build` na pasta do tema
2. Enviar via FTP (exceto `node_modules` e `vendor`):
   - Pasta `public/build/` do tema
   - Demais arquivos do tema
3. WordPress e banco devem estar configurados no Hostgator previamente
4. Verificar se `.htaccess` existe na raiz do site
5. Verificar permissões da pasta `uploads` via cPanel
6. Ativar SSL via cPanel (Let's Encrypt gratuito)

---

*Gerado durante o desenvolvimento do boilerplate Evolu-e — março de 2026*
