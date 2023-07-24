## Module Setup ##
Step 1 -> Place Module IN <magento_root>/app/code/Fancode/BookStore
Step 2 -> Run Full Deploy Commands of Magento
	sudo php bin/magento setup:upgrade
	sudo php bin/magento setup:di:compile
	sudo rm -rf pub/static/* var/view_preprocessed/*
	sudo php bin/magento setup:static-content:deploy -f 
	sudo php bin/magento cache:flush && sudo php bin/magento cache:clean && sudo chmod -R 777 generated/ var/ pub/
Step 3 -> Import product_import.csv file
	.) goto Magento Admin -> System -> Data Transfer -> Import
	.) Import Setting -> Entity Type (Choose "Products")
	.) Import Behavior -> Import Behavior (Choose "Add/Update")
	.) Import Behavior -> Fields enclosur "Checked"
	.) File to Import -> Select File to Import ("Choose File product_import.csv")
	.) Hit "Check Data" button 
	.) Hit "Import"
Step 4 -> Run Indexer Command to reindex the indexer
	sudo php bin/magento indexer:reindex
	sudo php bin/magento cache:flush && sudo php bin/magento cache:clean && sudo chmod -R 777 generated/ var/ pub/
Step 5 -> Enable show out of stock options
	sudo php bin/magento config:set cataloginventory/options/show_out_of_stock 1
	sudo php bin/magento cache:flush && sudo php bin/magento cache:clean && sudo chmod -R 777 generated/ var/ pub/
