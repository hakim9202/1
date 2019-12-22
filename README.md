# 1

The DBController.php class will handle the DAO operation and fetch products result.

<?php
$product_array = $db_handle->runQuery("SELECT * FROM tblproduct ORDER BY id ASC");
if (!empty($product_array)) { 
	foreach($product_array as $key=>$value){
?>
	<div class="product-item">
		<form method="post" action="index.php?action=add&code=<?php echo $product_array[$key]["code"]; ?>">
		<div class="product-image"><img src="<?php echo $product_array[$key]["image"]; ?>"></div>
		<div class="product-tile-footer">
		<div class="product-title"><?php echo $product_array[$key]["name"]; ?></div>
		<div class="product-price"><?php echo "$".$product_array[$key]["price"]; ?></div>
		<div class="cart-action"><input type="text" class="product-quantity" name="quantity" value="1" size="2" /><input type="submit" value="Add to Cart" class="btnAddAction" /></div>
		</div>
		</form>
	</div>
<?php
	}
}
?>


#2
After creating the product gallery page, we need to work on the PHP code to perform the cart actions. 
They are add-to-cart, remove a single item from the cart, clear the complete cart and similar.

In the above code, I have added the HTML option to add the product to the shopping cart from the product gallery. When the user clicks the ‘Add to Cart’ button, 
the HTML form passes the product id to the backend PHP script.

The “add” case handles the add to cart action. In this case, I have received the product id and the quantity.
The cart session stores the submitted product. I have updated the quantity of the product if it already exists 
in the session cart. The PHP session index is the reference to perform this action.

case "add":
	if(!empty($_POST["quantity"])) {
		$productByCode = $db_handle->runQuery("SELECT * FROM tblproduct WHERE code='" . $_GET["code"] . "'");
		$itemArray = array($productByCode[0]["code"]=>array('name'=>$productByCode[0]["name"], 'code'=>$productByCode[0]["code"], 'quantity'=>$_POST["quantity"], 'price'=>$productByCode[0]["price"], 'image'=>$productByCode[0]["image"]));
////////////////////////////////////////////////////////////////////////////////////////////////////		
		if(!empty($_SESSION["cart_item"])) {
			if(in_array($productByCode[0]["code"],array_keys($_SESSION["cart_item"]))) {
				foreach($_SESSION["cart_item"] as $k => $v) {
						if($productByCode[0]["code"] == $k) {
							if(empty($_SESSION["cart_item"][$k]["quantity"])) {
								$_SESSION["cart_item"][$k]["quantity"] = 0;
							}
							$_SESSION["cart_item"][$k]["quantity"] += $_POST["quantity"];
						}
				}
			} else {
				$_SESSION["cart_item"] = array_merge($_SESSION["cart_item"],$itemArray);
			}
		} else {
			$_SESSION["cart_item"] = $itemArray;
		}
	}
	break;
  ////////////////////////////////////////////////////////////////////////////////////////////
  
  
  #3
  This code shows the HTML to display the shopping cart with action controls. 
  The loop iterates the cart session to list the cart items in a tabular format.
  Each row shows the product image, title, price, item quantity with a remove option. Also, it shows the total item quantity 
  and the total item price by summing up the individual cart items.on this section item with selected code such as Apple Tv and 
  Super iPad will directly have their own discount rate if the user meet certain amount criteria.
*********************************************************************************************************  
  <th style="text-align:left;">Name</th>
<th style="text-align:left;">Code</th>
<th style="text-align:right;" width="5%">Quantity</th>
<th style="text-align:right;" width="10%">Unit Price</th>
<th style="text-align:right;" width="10%">Price</th>
<th style="text-align:center;" width="5%">Remove</th>
</tr>	
<?php		
    foreach ($_SESSION["cart_item"] as $item){
		if($item["quantity"] >= 2 && $item["code"]==="atv"){
			$item_price_b4_dis = $item["quantity"]*$item["price"];
			$item_price=$item_price_b4_dis-$item["price"];
		}
		else if($item["quantity"] >= 4 && $item["code"]=== "ipd"){
			$item_price_b4_dis = $item["quantity"]*$item["price"];
			$discount_rate = 400;
			$total_amount_discount = $discount_rate * $item["quantity"];
			$item_price=$item_price_b4_dis-$total_amount_discount;

		}
		else{
		$item_price = $item["quantity"]*$item["price"];
		}
		?>
				<tr>
				<td><?php echo $item["name"]; ?></td>
				<td><?php echo $item["code"]; ?></td>
				<td style="text-align:right;"><?php echo $item["quantity"]; ?></td>
				<td  style="text-align:right;"><?php echo "$ ".$item["price"]; ?></td>
				<td  style="text-align:right;"><?php echo "$ ". number_format($item_price,2); ?></td>
				<td style="text-align:center;"><a href="index.php?action=remove&code=<?php echo $item["code"]; ?>" class="btnRemoveAction"><img src="icon-delete.png" alt="Remove Item" /></a></td>
				</tr>
				<?php
				$total_quantity += $item["quantity"];
				$total_price += $item_price;
		}
		?>
    ****************************************************************************************************************************************************

handling discount function

  foreach ($_SESSION["cart_item"] as $item){
  //select particular item  in this case we select apple tv and we set only quantity with more than or equal 2 will be execute in this function  
		if($item["quantity"] >= 2 && $item["code"]==="atv"){
    
// if the code match then first we calculate their total price without any discount given
			$item_price_b4_dis = $item["quantity"]*$item["price"];
 // then we calculate actual total price which     total  price without discount-price per unit 
			$item_price=$item_price_b4_dis-$item["price"];
		}
    
    //select particular item  in this case we select super ipod and we set only quantity with more than or equal to 4 will be execute in this function    
		else if($item["quantity"] >= 4 && $item["code"]=== "ipd"){
			$item_price_b4_dis = $item["quantity"]*$item["price"];
      // assign discount rate into variable
			$discount_rate = 400;
      // with times it with quantity to get total discount price will be given
			$total_amount_discount = $discount_rate * $item["quantity"];
      //we minus  total price B4 discount with total amount discount to get actual total price to be pay
			$item_price=$item_price_b4_dis-$total_amount_discount;
		}
		else{
    // the rest item will be calculate as normal without any discount given
		$item_price = $item["quantity"]*$item["price"];
		}
    
   
*******************************************************************************************************
  
