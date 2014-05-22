# Basket Service
Business Entity Service responsible for Persisted storing of Items for immediate or later Purchase. 

# Operations
* **getBasket**

  Retrieves all Items in a Customer's Basket
* **removeFromBasket**

  Delete Item from Customer's Basket

* **addToBasket**

  Add Item to Customer's Basket

* **emptyBasket**

  Remove all Items from Customer's Basket (checkout scenario)

# Technical Points of Note

## Transformation
* DataMapper is not able to handle **structural mis-matches** easily. In the case of **getBasket**, we have our List of Maps retrieved from the Database and we need to create an Xml document according to the WSDL Schema. However, given that each Item has many Images and it's convenient for us to retrieve all in the one ResultSet, we need to de-duplicate the repeating Item data. We do this in MEL and we create a Map of data which is structurally similar to the target XML in order to give DataMapper a helping hand!
````xml
	<expression-transformer doc:name="to []" expression="
    	np = [ 'BasketItem' : [] ];
    	index = ['x' : 'x'];
    	for (record :  payload) {
    		if (index.containsKey(record.sku_id)) {
    			index[record.sku_id].images.add([ 'url' : record.url, 'type' : record.image_type_name ]);
    		}
    		else {
    			item = [ 'id' : record.sku_id, 'sku': record.sku_id, 'quantity' : record.quantity, 'name': record.sku_name, 'summary' : record.sku_summary, 'price' : record.price, 'stockQuantity' : record.stock_quantity,
    				'brand' : record.item_brand, 'type' : record.item_type_name, 'images' : [ ['url' : record.url, 'type' : record.image_type_name] ] ];
    			index.put(record.sku_id, item);
    		}
    	};
    	index.remove('x');
    	np.BasketItem	.addAll(index.values());
    	np
    ">
    </expression-transformer>
````
## XPath Namespaces
* Resolution is done using the **Namespace Manager**
````xml
	<mulexml:namespace-manager includeConfigNamespaces="true">
        <mulexml:namespace prefix="mes" uri="http://www.alainn.com/SOA/message/1.0" />
        <mulexml:namespace prefix="mod" uri="http://www.alainn.com/SOA/model/1.0" />
    </mulexml:namespace-manager>
````
## SSL
* The **HTTPS** (Jetty also) connector makes a reference to the keystore containing the Certificate in src/main/resources.
````xml
	<https:connector name="httpsConnector" cookieSpec="netscape" validateConnections="true" sendBufferSize="0" receiveBufferSize="0" receiveBacklog="0" clientSoTimeout="10000" serverSoTimeout="10000" socketSoLinger="0" doc:name="HTTP\HTTPS">
        <service-overrides messageFactory="org.mule.transport.http.HttpMultipartMuleMessageFactory" />
        <https:tls-key-store path="keystore.jks" keyPassword="changeit" storePassword="changeit" />
    </https:connector>
````

# Contact
nial.darbey@mulesoft.com