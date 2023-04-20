# Shopware6_pluginChangeColor_AddToCartButton


Створити новий плагін для Shopware 6, який змінюватиме колір кнопки "Додати 
в кошик" на сторінці товару (Product detail page), а також на сторінці категорії 
(Product list page).

• У налаштуваннях плагіна має бути можливість задавати один із кольорів (за 
замовчуванням, червоний, зелений, синій).
• Оформити плагін відповідно до стандартів Shopware 6.
• Як результат кандидат має надати посилання на репозиторій, де буде розміщено 
код плагіна.



Щоб створити плагін для Shopware 6, який змінюватиме колір кнопки "Додати в кошик" на сторінці товару (Product detail page) та сторінці категорії (Product list page), можна використовувати наступний алгоритм:

1.  Створіть папку з назвою вашого плагіна у директорії "custom/plugins" в кореневій папці Shopware 6.
2.  Додайте у папку плагіна файл composer.json з наступним вмістом:

```
swiftCopy code{
   "name": "vendor-name/plugin-name",
   "type": "shopware-platform-plugin",
   "description": "My plugin description",
   "license": "MIT",
   "autoload": {
       "psr-4": {
           "VendorName\\PluginName\\": "src/"
       }
   },
   "extra": {
       "shopware-plugin-class": "VendorName\\PluginName\\Plugin"
   }
}

```

3.  Додайте у папку плагіна папку src та у цій папці створіть файл Plugin.php з наступним вмістом:

```
phpCopy code<?php

namespace VendorName\PluginName;

use Shopware\Core\Framework\Plugin;

class Plugin extends Plugin
{
}

```

4.  Додайте у папку src папку Resources та у цій папці створіть файліви css/app.css та js/app.js. У файлі css/app.css додайте наступний код:

```css
cssCopy code.add-to-cart-button {
   background-color: {$color} !important;
}

```

У файлі js/app.js додайте наступний код:

```js
javascriptCopy codeimport template from './template.twig';

Shopware.Component.override('sw-product-detail-buy', {
   template,

   computed: {
       buttonColor() {
           return this.$store.state.pluginName.color;
       }
   }
});

```

5.  Створіть файл template.twig у папці src/Resources/views/storefront/component/product-detail/buy з наступним вмістом:

```tpl
bashCopy code{% block sw_product_detail_buy_button %}
   {% set buttonColor = this.$store.state.pluginName.color %}
   {% set buttonClasses = [
       'btn-primary',
       'add-to-cart-button'
   ] %}

   <button class="{{ buttonClasses|join(' ') }}" type="button">
       {{ "listing.boxAddProduct"|trans|sw_sanitize }}
   </button>
{% endblock %}

```

6.  Додайте у папку src/DependencyInjection файл Plugin.php з наступним вмістом:

```php
phpCopy code<?php

namespace VendorName\PluginName\DependencyInjection;

use Shopware\Core\Framework\DependencyInjection\Extension;
use Symfony\Component\Config\FileLocator;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Loader\XmlFileLoader;

class Plugin extends Extension
{
   public function load(array $configs, ContainerBuilder $container): void
   {
       $loader = new XmlFileLoader($container, new FileLocator(__DIR__.'/../Resources/config'));
       $loader->load('services.xml');
   }
}

```

7.  Додайте у папку src/Resources/config файл services.xml з наступним вмістом:

```xml

<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://symfony.com/schema/dic/services https://symfony.com/schema/dic/services/services-1.0.xsd">
   <services>
       <service id="VendorName\PluginName\Core\Content\Product\SalesChannel\AddToCartRoute"
                decorates="Shopware\Core\Content\Product\SalesChannel\AddToCartRoute">
           <argument type="service" id="VendorName\PluginName\Core\Content\Product\SalesChannel\AddToCartRoute.inner"/>
           <argument type="service" id="Shopware\Core\System\SystemConfig\SystemConfigService"/>
           <argument type="service" id="Shopware\Core\Framework\DataAbstractionLayer\Search\CriteriaFactory"/>
           <argument type="service" id="Shopware\Core\Checkout\Cart\SalesChannel\CartService"/>
           <argument type="service" id="Shopware\Core\Checkout\Customer\SalesChannel\AccountService"/>
           <argument type="service" id="Shopware\Core\Framework\Routing\Router"/>
           <argument type="service" id="Shopware\Core\Framework\DataAbstractionLayer\EntityRepositoryInterface"/>
           <argument type="service" id="Shopware\Core\Framework\Context\SystemSource"/>
           <argument type="service" id="Shopware\Core\System\Language\LanguageEntity"/>
           <argument type="service" id="Shopware\Core\System\SalesChannel\SalesChannelContextFactory"/>
           <argument type="service" id="VendorName\PluginName\Core\Content\Product\SalesChannel\AddToCartRoute.color"/>
           <tag name="sales_channel.route"/>
       </service>
       <service id="VendorName\PluginName\Core\Content\Product\SalesChannel\AddToCartRoute.color">
           <argument>%pluginName.color%</argument>
       </service>
   </services>
</container>
```

Додайте у папку src/Core/Content/Product/SalesChannel файл AddToCartRoute.php з наступним вмістом:

```php

<?php

namespace VendorName\PluginName\Core\Content\Product\SalesChannel;

use Shopware\Core\Content\Product\SalesChannel\AddToCartRoute as ParentRoute;
use Shopware\Core\Framework\DataAbstractionLayer\Search\Criteria;
use Shopware\Core\Framework\DataAbstractionLayer\Search\Sorting\FieldSorting;
use Shopware\Core\Framework\Plugin\PluginServiceInterface;
use Shopware\Core\Framework\Validation\DataBag\RequestDataBag;
use Symfony\Component\HttpFoundation\Request;

class AddToCartRoute extends ParentRoute
{
   private $pluginService;
   private $color;

   public function __construct(
       ParentRoute $route,
       PluginServiceInterface $pluginService,
       string $color
   ) {
       parent::__construct($route->getDecorated());

       $this->pluginService = $pluginService;
       $this->color = $color;
   }

   public function load(Request $request, RequestDataBag $data, SalesChannelContext $context, Criteria $criteria): CartResponse
   {
       $criteria->addSorting(new FieldSorting('name', FieldSorting::ASCENDING));

       $cart = $this->decorated->load($request, $data, $context, $criteria);

       $cart->set('buttonColor', $this->color);

       return $cart;
   }
}
```

Додайте у файлі config/services.xml наступний код:

```xml
<service id="VendorName\Plugin
```

Додайте у папку resources/views/storefront/component/button/ файл add_to_cart.html.twig з наступним вмістом:

```php
{% sw_extends '@Storefront/storefront/component/button/add-to-cart.html.twig' %}

{% block button %}
    {% set buttonColor = cartResponse.get('buttonColor') ?? config('core.cart.addProductButtonColor') %}
    <button class="btn btn-block product-detail-buy btn-{{ buttonColor }}"{% if product.calculatedListingPrice.positionPriceDefinition.calculatedPrices.0.calculatedPrice.unitPrice == 0 %} disabled{% endif %}>
        {% if product.calculatedListingPrice.positionPriceDefinition.calculatedPrices.0.calculatedPrice.unitPrice == 0 %}
            {{ "listing.buyProductForFree"|trans|sw_sanitize }}
        {% else %}
            {{ "listing.addToBasket"|trans|sw_sanitize }}
        {% endif %}
    </button>
{% endblock %}
```

Збережіть всі файли та завантажте папку PluginName в папку custom/plugins вашого Shopware 6.

Зайдіть в адміністративну панель вашого Shopware 6 та встановіть плагін.

Перезавантажте сторінку товару (Product detail page) або сторінку категорії (Product list page) та перевірте, чи змінився колір кнопки "Додати в кошик" відповідно до значення, яке ви задали у налаштуваннях плагіна.

Для надання посилання на репозиторій з кодом плагіна вам необхідно завантажити код плагіна на публічний репозиторій, такий як GitHub, та надати посилання на репозиторій.
