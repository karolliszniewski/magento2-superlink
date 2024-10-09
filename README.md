# magento2-superlink

Superlink in Magento 2:
Superlink is a feature that allows customers to navigate directly to a specific configuration of a configurable product. It creates unique URLs for each combination of options, making it easier for customers to share or bookmark specific product variations.
Database Structure:
In Magento 2, the superlink functionality is primarily handled by the following tables:

catalog_product_super_link
catalog_product_super_attribute
catalog_product_super_attribute_label

Here's a visual representation of the database structure:

<img width="412" alt="image" src="https://github.com/user-attachments/assets/170d6ccb-c824-419c-96bc-74a7cc86aa73">

This diagram shows the relationships between the main tables involved in the superlink functionality.
Theoretical Aspect:
The superlink feature works by creating associations between the configurable product (parent) and its simple products (children). The catalog_product_super_link table stores these associations. The catalog_product_super_attribute table defines which attributes are used for configurations, and the catalog_product_super_attribute_label table stores the labels for these attributes in different store views.
Removing Superlink Programmatically:
To remove the superlink from a configurable product, you need to remove the associations between the configurable product and its simple products. Here's an example of how you can do this programmatically in Magento 2:

```php
<?php

namespace YourNamespace\YourModule\Model;

use Magento\ConfigurableProduct\Model\Product\Type\Configurable;
use Magento\Catalog\Api\ProductRepositoryInterface;
use Magento\Framework\App\ResourceConnection;

class SuperlinkRemover
{
    protected $configurable;
    protected $productRepository;
    protected $resourceConnection;

    public function __construct(
        Configurable $configurable,
        ProductRepositoryInterface $productRepository,
        ResourceConnection $resourceConnection
    ) {
        $this->configurable = $configurable;
        $this->productRepository = $productRepository;
        $this->resourceConnection = $resourceConnection;
    }

    public function removeSuperlink($productSku)
    {
        try {
            $product = $this->productRepository->get($productSku);
            
            if ($product->getTypeId() !== Configurable::TYPE_CODE) {
                throw new \Exception('Product is not configurable.');
            }

            $connection = $this->resourceConnection->getConnection();
            $linkTable = $this->resourceConnection->getTableName('catalog_product_super_link');

            // Remove associations
            $connection->delete($linkTable, ['parent_id = ?' => $product->getId()]);

            // Remove super attributes
            $this->configurable->getTypeInstance()->getUsedProducts($product, null);

            return true;
        } catch (\Exception $e) {
            // Handle exception
            return false;
        }
    }
}
```


To use this code:

Create a new file in your custom module under Model/SuperlinkRemover.php.
Paste the code into the file.
You can then inject this class into your controller or command and use it like this:

```php
$result = $this->superlinkRemover->removeSuperlink('configurable-product-sku');
if ($result) {
    echo "Superlink removed successfully.";
} else {
    echo "Failed to remove superlink.";
}
```

This code removes the associations in the catalog_product_super_link table and clears the configurable product's super attributes.
Remember to handle any potential side effects, such as updating indexes or clearing caches, after removing the superlink. Also, ensure that you have proper error handling and logging in place when using this in a production environment.
Would you like me to explain any part of this process in more detail?
