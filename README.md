![page-login](https://user-images.githubusercontent.com/8649070/120119713-efae3900-c1a1-11eb-88cf-481afb058b07.png)

![page-admin](https://user-images.githubusercontent.com/8649070/120119714-f177fc80-c1a1-11eb-8853-3d60b5c1c4ed.png)

pdAdmin
=========
Supported **PHP8 and Composer 2**

Symfony Powerful Dashboard & Admin. Developed with **Symfony 5**, **Vue 3**, **Bootstrap 5** framework.

No changes were made to the symfony structure, the current directory structure is used. A custom namespace for Admin has been created. This field is used for all administrator operations.

The interface is designed to be responsive using Twitter Bootstrap. The least possible dependency was tried to be used.

Properties
--------------------
* Messenger was used for queuing.
* PM2 has been set for background processes.
* Cron processes are managed by PM2.
* A special Data Table has been written to the panel (Vue3)
* Supports CSV, Excel export.
* Special package written for HTTP vs Mail logging.
* JWT is used for API login.
* Responsive design
* Vue documentation is not yet available, see source file.

Installation
--------------------
1. Download pdAdmin
    ```
    composer create-project appaydin/pd-admin pdadmin
    ```
2. Create and configure the `.env` file.

3. Create database schemas
    ```
    bin/console doctrine:schema:create
    ```
4. Run built-in web server
     ```
     symfony server:start --no-tls -d
     ```
5. Install & Build assets
     ```
     yarn install
     yarn run build
     ```
6. Run Backround Process
     ```
     pm2 start

     # Manuel
     # bin/console messenger:consume -vv
     # bin/console schedule:run
     ```

Documentation
--------------------

* [User Management](#user-management)
* [Multilingual System](#multilingual-system)
* [Delegation](#delegation)
* [System Settings](#system-settings)
* [Create New Widget](#create-new-widget)
* [Create New Menu](#create-new-menu)

### User Management
There is [pd-user](https://github.com/appaydin/pd-user) for user management. All settings are in __config/packages/pd_user.yaml__ file.

* __Create User:__
    ```
    bin/console user:create
    ```
* __Change User Password:__
    ```
    bin/console user:changepassword
    ```
* __Change User Roles:__
    ```
    bin/console user:role
    ```

### Multilingual System
User logon for multi language is used. Each user can choose his / her own language.
When you log in, you are automatically redirected.

New languages can be added from the kernel settings. You need to translate manually for the new language.

### Delegation
[SensioFrameworkExtraBundle](https://symfony.com/doc/master/bundles/SensioFrameworkExtraBundle/annotations/security.html) is used with Symfony security component. There are three default user roles.

* ROLE_USER
* ROLE_SUPER_ADMIN

ROLE_SUPER_ADMIN has full authority. ROLE_USER authorities can be restricted and panel access can be turned off in the __security.yaml__ file.

### System Settings
System settings are stored in the database. All settings can be used as parameters after container assembly. Since all settings are compiled with the container
it does not create any additional load on the system. Settings can be configured using Symfony Forms and added to the Settings menu from the outside via the "Menu Event" system.
Clear the cache after changes to system settings, otherwise the new settings will not be enabled.

For general settings, you can add it to __src/Admin/Forms/System/GeneralForm__

__Add New Menu to Settings__:
```php
<?php
//src/Admin/Menu/SettingsMenu.php

namespace App\Admin\Menu;

use Pd\MenuBundle\Builder\ItemInterface;
use Pd\MenuBundle\Builder\Menu;

class SettingsMenu extends Menu
{
    public function createMenu(array $options = []): ItemInterface
    {
        // Create Root Item
        $menu = $this->createRoot('settings_menu')->setChildAttr([
            'class' => 'nav nav-pills',
            'data-parent' => 'admin_config_general',
        ]);

        // Create Menu Items
        $menu->addChild('nav_config_general')
            ->setLabel('nav_config_general')
            ->setRoute('admin_config_general')
            ->setLinkAttr(['class' => 'nav-item'])
            ->setRoles(['ROLE_CONFIG_GENERAL'])
            // Email
            ->addChildParent('nav_config_email')
            ->setLabel('nav_config_email')
            ->setRoute('admin_settings_email')
            ->setLinkAttr(['class' => 'nav-item'])
            ->setRoles(['ROLE_SETTINGS_EMAIL']);

        return $menu;
    }
}
```

### Create New Widget
Widget system was created with Symfony "EventDispatcher Component".
It has an adjustable structure for each user and it can be specially designed with "Twig Template" engine.
For more information visit [pd-widget](https://github.com/appaydin/pd-widget)

Create New Admin Widget:
```php
<?php
//src/Admin/Widgets/AccountWidget.php

namespace App\Admin\Widgets;

use Pd\WidgetBundle\Builder\Item;
use Pd\WidgetBundle\Event\WidgetEvent;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\HttpFoundation\Request;

class AccountWidget
{
    private $entityManager;

    public function __construct(EntityManagerInterface $entityManager)
    {
        $this->entityManager = $entityManager;
    }

    /**
     * Build Widgets.
     *
     * @param WidgetEvent $event
     */
    public function builder(WidgetEvent $event)
    {
        // Get Widget Container
        $widgets = $event->getWidgetContainer();

        // Add Widgets
        $widgets
            ->addWidget((new Item('user_statistics', 3600))
                ->setGroup('admin') // Widget Adds to "Admin" Group
                ->setName('widget_user_statistics.name')
                ->setDescription('widget_user_statistics.description')
                ->setTemplate('@Admin/Widget/userStatistics.html.twig')
                ->setRole(['ROLE_WIDGET_USERSTATISTICS'])
                ->setConfigProcess(function (Request $request) {
                    /**
                     * Controller for Widget Settings
                     * The return value is stored in the user specific database
                     */
                    if ($type = $request->get('type')) {
                        switch ($type) {
                            case '1week':
                                return ['type' => '1week'];
                            case '1month':
                                return ['type' => '1month'];
                            case '3month':
                                return ['type' => '3month'];
                        }
                    }

                    return false;
                })
                ->setData(function ($config) {
                    /**
                     * The return value can be used in the twig template.
                     * The function will not execute unless you call it in the template.
                     * You can use the database operations here.
                     */
                    // Set Default Config
                    if (!isset($config['type'])) {
                        $config['type'] = '1week';
                    }

                    // Create Statistics Data
                    if ($config['type'] === '1month') {
                        $data = ['chartDay' => '7'];
                        // Create Data
                    } else if ($config['type'] === '1month') {
                        $data = ['chartDay' => '30'];
                    } else {
                        $data = ['chartDay' => '90'];
                    }

                    return $data;
                })
            );
    }
}
```

### Create New Menu
The menu system was created with Symfony "EventDispatcher Component".
For each menu created, Event is generated by default, can be turned off by menu configuration.
For more information visit the [pd-menu](https://github.com/appaydin/pd-menu)

Create Menu:
```php
<?php
// src/Admin/Menu/MainNav.php

namespace App\Admin\Menu;

use Pd\MenuBundle\Builder\ItemInterface;
use Pd\MenuBundle\Builder\Menu;

class MainNav extends Menu
{
    public function createMenu(array $options = []): ItemInterface
    {
        // Create ROOT Menu
        $menu = $this->createRoot('main_menu', true); // Event enabled

        // Create Dashboard
        $menu->addChild('nav_dashboard', 1)
            ->setLabel('nav_dashboard')
            ->setRoute('admin_dashboard')
            ->setRoles(['ROLE_DASHBOARD'])
            ->setExtra('label_icon', 'dashboard');

        /*
         * Create Account Section
         */
        $menu
            ->addChild('nav_account', 5)
            ->setLabel('nav_account')
            ->setRoute('admin_account_list')
            ->setRoles(['ROLE_ACCOUNT_LIST'])
            ->setExtra('label_icon', 'people')
                // Account List
                ->addChild('nav_account', 1)
                ->setLabel('nav_account')
                ->setRoute('admin_account_list')
                ->setRoles(['ROLE_ACCOUNT_LIST'])
                // Group List
                ->addChildParent('nav_group', 2)
                ->setLabel('nav_group')
                ->setRoute('admin_account_group_list')
                ->setRoles(['ROLE_GROUP_LIST']);

        return $menu;
    }
}
```
