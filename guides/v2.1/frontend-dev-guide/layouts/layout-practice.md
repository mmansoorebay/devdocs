---
layout: default
group: fedg
subgroup: B_Layouts
title: Customizing layout illustration
menu_title: Customizing layout illustration
menu_order: 7
version: 2.1
github_link: frontend-dev-guide/layouts/layout-practice.md
---

<h2>What's in this topic</h2>
This article features a step-by-step illustration of how a real-life layout customization task is performed. Namely, it illustrates how to change the layout of customer account links in a Magento store page header.

<h2>Moving customer account links</h2>
In their Orange theme, OrangeCo wants to transform the header links block to a drop-down, the way it is done in the Magento Luma theme:

<div style="border: 1px solid #ABABAB">
<img src="{{ site.baseurl }}common/images/layout_transform21.png">
</div>

To do this, they need to wrap the list of header links with a container and add a greeting with a drop-down arrow before the list.

The Orange theme [inherits]({{site.gdeurl21}}frontend-dev-guide/themes/theme-inherit.html) from Blank, so by default the rendered header links look like following:

<div style="border: 1px solid #ABABAB">
<img src="{{ site.baseurl }}common/images/layout_code_before121.png">
</div>

Needed:

<div style="border: 1px solid #ABABAB">
<img src="{{ site.baseurl }}common/images/layout_code_after21.png">
</div>

<br>
<u>Step 1: Define the blocks</u>

OrangeCo <a href="{{site.gdeurl21}}frontend-dev-guide/themes/theme-apply.html" target="_blank">applies the Luma theme</a>. Using the approach described in <a href="{{site.gdeurl21}}frontend-dev-guide/themes/debug-theme.html" target="_blank">Locate templates, layouts, and styles</a> they find out that the blocks responsible for displaying the header links are defined in `<Magento_Customer_module_dir>/view/frontend/layout/default.xml`:

{%highlight xml%}
...
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceBlock name="top.links">
            <block class="Magento\Customer\Block\Account\Link" name="my-account-link">
                <arguments>
                    <argument name="label" xsi:type="string" translate="true">My Account</argument>
                </arguments>
            </block>
            <block class="Magento\Customer\Block\Account\RegisterLink" name="register-link">
                <arguments>
                    <argument name="label" xsi:type="string" translate="true">Register</argument>
                </arguments>
            </block>
            <block class="Magento\Customer\Block\Account\AuthorizationLink" name="authorization-link" template="account/link/authorization.phtml"/>
        </referenceBlock>
    </body>
</page>
{%endhighlight xml%}


<u>Step 2: Define the templates</u>

Similar to the way they defined the layout on the previous step, OrangeCo 
defines the template which is used for rearranging the links:

`<Magento_Customer_module_dir>/view/frontend/templates/account/customer.phtml`

{%highlight php%}
<?php if($block->customerLoggedIn()): ?>
    <li class="customer-welcome">
        <span class="customer-name"
              role="link"
              tabindex="0"
              data-mage-init='{"dropdown":{}}'
              data-toggle="dropdown"
              data-trigger-keypress-button="true"
              data-bind="scope: 'customer'">
            <span data-bind="text: customer().fullname"></span>
            <button type="button"
                    class="action switch"
                    tabindex="-1"
                    data-action="customer-menu-toggle">
                <span><?php /* @escapeNotVerified */ echo __('Change')?></span>
            </button>
        </span>
        <script type="text/x-magento-init">
        {
            "*": {
                "Magento_Ui/js/core/app": {
                    "components": {
                        "customer": {
                            "component": "Magento_Customer/js/view/customer"
                        }
                    }
                }
            }
        }
        </script>
        <?php if($block->getChildHtml()):?>
        <div class="customer-menu" data-target="dropdown">
            <?php echo $block->getChildHtml();?>
        </div>
        <?php endif; ?>
    </li>
<?php endif; ?>
{%endhighlight php%}

<br>
<u>Step 3: Extend the base layout to add a block</u>

OrangeCo needs to create a new block, say, `header.links`, in the `header.panel` container, to move the links there. As the links can be added to this list by different modules, it is better to add this block to the `default.xml` page configuration of the `Magento_Theme` module.

So the following <a href="{{site.gdeurl21}}frontend-dev-guide/layouts/layout-extend.html" target="_blank">extending</a> layout is added in the Orange theme:

    app/design/frontend/OrangeCo/orange/Magento_Theme/layout/default.xml

{%highlight xml%}
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceContainer name="header.panel">
            <block class="Magento\Framework\View\Element\Html\Links" name="header.links">
                <arguments>
                    <argument name="css_class" xsi:type="string">header links</argument>
                </arguments>
            </block>
        </referenceContainer>
    </body>
</page>
{%endhighlight xml%}

<br>

<u>Step 4: Move links</u>

To move the links to the `header.links` block, OrangeCo adds an extending layout:

`app/design/frontend/OrangeCo/orange/Magento_Customer/layout/default.xml`


{%highlight xml%}
    <page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
        <body>
            <referenceBlock name="header.links">
                <block class="Magento\Customer\Block\Account\Customer" name="customer" template="account/customer.phtml" before="-"/>
                <block class="Magento\Customer\Block\Account\AuthorizationLink" name="authorization-link-login" template="account/link/authorization.phtml"/>
            </referenceBlock>
            <move element="register-link" destination="header.links"/>
            <move element="top.links" destination="customer"/>
            <move element="authorization-link" destination="top.links" after="-"/>
        </body>
    </page>
{%endhighlight xml%}

Now the customer links look like following:

<div style="border: 1px solid #ABABAB">
<img src="{{ site.baseurl }}common/images/layout_screen221.png">
</div>

The last touch is adding styles:

<div style="border: 1px solid #ABABAB">
<img src="{{ site.baseurl }}common/images/layout_screen321.png">
</div>



