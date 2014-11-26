---
layout: "content"
image: "Modeling"
title: "STORE: Economy Model & Operations"
text: "Every game economy can be based on SOOMLA's economy model. The game economy entities that SOOMLA provides are virtual currencies, currency packs, and virtual items of all sorts."
position: 2
theme: 'platforms'
collection: 'ios_store'
module: 'store'
platform: 'ios'
---

#**STORE: Economy Model & Operations**

SOOMLA's iOS-store provides a complete data model implementation for virtual economies. Every game economy has currencies, packs of currencies that can be sold, and items that can be sold either for money or in exchange for other items. This tutorial contains descriptions of each entity in the economy model, along with examples.

![alt text](/img/tutorial_img/soomla_diagrams/EconomyModel.png "Soomla Economy Model")

## Purchase Types
Purchase types are used to indicate whether an item will be purchased with money or with other virtual items.

<div class="info-box">In the examples below the declarations of purchase types are shown as a part of `PurchasableVirtualItem` declarations, because this is the most common use of purchase types.</div>

###[PurchaseWithMarket](https://github.com/soomla/ios-store/blob/master/SoomlaiOSStore/PurchaseTypes/PurchaseWithMarket.h)

This type of purchase is with money. Items with this purchase type must be defined in the App Store (To learn more see our [App Store IAB](/ios/store/AppStoreIAB) tutorial).

There are 2 ways to define this purchase type.

``` objectivec
NO_ADS = [[LifetimeVG alloc]
    ...
    andPurchaseType:[[PurchaseWithMarket alloc]
        initWithMarketItem:[[MarketItem alloc]
            initWithProductId:NO_ADS_PRODUCT_ID
            andConsumable:kNonConsumable
            andPrice:1.99]]];
```

OR

``` objectivec
_1000_MUFFINS_PACK = [[VirtualCurrencyPack alloc]
    ...
    andPurchaseType:[[PurchaseWithMarket alloc]
        initWithProductId:_1000_MUFFINS_PRODUCT_ID
        andPrice:8.99]];
```

<div class="info-box">The `productId` that is used to define a new `MarketItem` must match the product ID defined in the App Store.</div>


###[PurchaseWithVirtualItem](https://github.com/soomla/ios-store/blob/master/SoomlaiOSStore/PurchaseTypes/PurchaseWithVirtualItem.h)

This type of purchase is with some amount of other virtual items.

``` objectivec
// A chocolate cake good can be purchased with 250 muffin currencies
CHOCOLATE_CAKE_GOOD = [[SingleUseVG alloc]
    ...
    andPurchaseType:[[PurchaseWithVirtualItem alloc]
        initWithVirtualItem:MUFFINS_CURRENCY_ITEM_ID
        andAmount:250]];
```


##Virtual Currencies
Virtual currencies need to be declared in your implementation of `IStoreAssets`.

###[VirtualCurrency](https://github.com/soomla/ios-store/blob/master/SoomlaiOSStore/domain/virtualCurrencies/VirtualCurrency.h)

####**How to define**

``` objectivec
NSString* const MUFFINS_CURRENCY_ITEM_ID = @"currency_muffin";

MUFFINS_CURRENCY = [[VirtualCurrency alloc]
    initWithName:@"Muffins"
    andDescription:@"Muffin currency"
    andItemId:MUFFINS_CURRENCY_ITEM_ID];
```

####**How to use**
A `VirtualCurrency` by itself is not very useful, because it cannot be sold individually. To sell currency, you need to use a `VirtualCurrencyPack` (see section below).

Use `VirtualCurrency` when defining `VirtualCurrencyPack`s:

``` objectivec
NSString* const _10_MUFFINS_PACK_ITEM_ID = @"muffins_10";

_10_MUFFINS_PACK = [[VirtualCurrencyPack alloc]
    initWithName:@"10 Muffins"
    andDescription:@"Pack of ten muffins"
    andItemId:_10_MUFFINS_PACK_ITEM_ID
    andCurrencyAmount:10
    andCurrency:MUFFINS_CURRENCY_ITEM_ID
    andPurchaseType:[[PurchaseWithMarket alloc]
        initWithMarketItem:[[MarketItem alloc]
            initWithProductId:_10_MUFFINS_PRODUCT_ID
            andConsumable:kConsumable
            andPrice:0.99]]];
```

**Give:**

Give a `VirtualCurrency` and get nothing in return.
This is useful if you'd like to give your users some amount of currency to begin with when they first download your game.

``` objectivec
// Give the user an initial balance of 1000 muffins.
[StoreInventory giveAmount:1000 ofItem:@"currency_muffin"];
```

####**Get the balance**
Get the balance of a specific `VirtualCurrency`.

``` objectivec
[StoreInventory getItemBalance:@"currency_muffin"];
```

###[VirtualCurrencyPack](https://github.com/soomla/ios-store/blob/master/SoomlaiOSStore/domain/virtualCurrencies/VirtualCurrencyPack.h)

####**How to define**

``` objectivec
 _50_MUFFINS_PACK = [[VirtualCurrencyPack alloc]
    initWithName:@"50 Muffins"
    andDescription:@"A currency pack of 50 muffins"
    andItemId:_50_MUFFINS_PACK_ITEM_ID
    andCurrencyAmount:50
    andCurrency:MUFFINS_CURRENCY_ITEM_ID
    andPurchaseType:[[PurchaseWithMarket alloc]
        initWithMarketItem:[[MarketItem alloc]
            initWithProductId:_50_MUFFINS_PRODUCT_ID
            andConsumable:kConsumable
            andPrice:1.99]]];
```

####**How to use**

**Buy:**

When your user buys a `VirtualCurrencyPack` of 50 muffins, his/her muffin currency balance will be increased by 50, and the payment will be deducted.

``` objectivec
[StoreInventory buyItemWithItemId:@"muffins_50"];
```

**Give:**

Give your users a 50-muffin pack for free. This is useful if you’d like to give your users a currency pack to begin with when they first download your game.
``` objectivec
[StoreInventory giveAmount:1 ofItem:@"muffins_50"];
```

**Take:**

This function simply deducts the user's balance. In case of a refund request, it is your responsibility to give the user back whatever he/she paid.

Take back the 50-muffin pack that the user owns:
``` objectivec
[StoreInventory takeAmount:1 ofItem:@"muffins_50"];
```

####**Get the balance**

`VirtualCurrencyPack`s do not have a balance of their own in the database. When a user purchases a `VirtualCurrencyPack`, the balance of the associated `VirtualCurrency` is increased.

``` objectivec
[StoreInventory getItemBalance:@"currency_muffin"];
```

##Virtual Goods

Virtual goods need to be declared in your implementation of `IStoreAssets`.

###[SingleUseVG](https://github.com/soomla/ios-store/blob/master/SoomlaiOSStore/domain/virtualGoods/SingleUseVG.h)

####**How to define**

``` objectivec
FRUIT_CAKE_GOOD = [[SingleUseVG alloc]
    initWithName:@"Fruit Cake"
    andDescription:@"Customers buy a double portion on each purchase of this cake"
    andItemId:@"fruit_cake"
    andPurchaseType:[[PurchaseWithVirtualItem alloc]
        initWithVirtualItem:MUFFINS_CURRENCY_ITEM_ID
        andAmount:225]];
```

####**How to use**

**Buy:**

When your user buys a `SingleUseVG` ("fruit_cake" in our example) his/her "fruit_cake" balance will be increased by 1, and the payment will be deducted.

``` objectivec
[StoreInventory buyItemWithItemId:@"fruit_cake"];
```

**Give:**

Gives your user the given amount of the `SingleUseVG` with the given `itemId` ("fruit_cake" in our example) for free. This is useful if you'd like to give your users a `SingleUseVG` to start with when they first download your game.

``` objectivec
[StoreInventory giveAmount:1 ofItem:@"fruit_cake"];
```

**Take:**

This function simply deducts the user's balance. In case of a refund request, it is your responsibility to give the user back whatever he/she paid.  

``` objectivec
[StoreInventory takeAmount:1 ofItem:@"fruit_cake"];
```


####**Get the balance**
Get the balance of a specific `SingleUseVG`.

``` objectivec
[StoreInventory getItemBalance:@"fruit_cake"];
```


###[SingleUsePackVG](https://github.com/soomla/ios-store/blob/master/SoomlaiOSStore/domain/virtualGoods/SingleUsePackVG.h)

####**How to define**

``` objectivec
// Define a pack of 5 "Fruit cake" goods that costs $2.99.
FRUIT_CAKE_GOOD_PACK = [SingleUsePackVG alloc]
    initWithName:@"fruit_cake_5pack"
    andDescription:@"A pack of 5 Fruit Cakes"
    andItemId:@"fruit_cake_5pack"
    andPurchaseType:[[PurchaseWithMarket alloc]
        initWithMarketItem:[[MarketItem alloc]
            initWithProductId:FRUIT_CAKE_5PACK_PRODUCT_ID
            andConsumable:kConsumable
            andPrice:2.99]]];
```

####**How to use**
The explanations for buying, giving, and taking are the same as those in [SingleUseVG](#singleusevg).

**Buy:**

``` objectivec
[StoreInventory buyItemWithItemId:@"fruit_cake_5pack"];
```

**Give:**

``` objectivec
[StoreInventory giveAmount:1 ofItem:@"fruit_cake_5pack"];
```

**Take:**

``` objectivec
[StoreInventory takeAmount:1 ofItem:@"fruit_cake_5pack"];
```

####**Get the balance**
`SingleUsePackVG`s do not have a balance of their own in the database. When a user buys a `SingleUsePackVG`, the balance of the associated `SingleUseVG` is increased. After buying a pack of 5 fruit cakes, your user's fruit cake balance should be increased by 5.

Query the balance of the virtual good with item ID "fruit_cake":

``` objectivec
[StoreInventory getItemBalance:@"fruit_cake"];
```


###[LifetimeVG](https://github.com/soomla/ios-store/blob/master/SoomlaiOSStore/domain/virtualGoods/LifetimeVG.h)

A LifetimeVG is a VirtualGood that is bought exactly once and kept forever.

<div class="info-box">Notice: When defining a `LifetimeVG` in the App Store (iTunesConnect), you MUST define its type as a Non-Consumable! For more information see our [guide](/docs/platforms/ios/appStoreIAB) for defining IAP products in the App Store.</div>

<br>

####**How to define**

``` objectivec
MARRIAGE_GOOD = [[LifetimeVG alloc]
    initWithName:@"Marriage"
    andDescription:@"This is a LIFETIME thing."
    andItemId:MARRIAGE_GOOD_ITEM_ID
    andPurchaseType:[[PurchaseWithMarket alloc]
        initWithMarketItem:[[MarketItem alloc]
            initWithProductId:MARRIAGE_PRODUCT_ID
            andConsumable:kNonConsumable andPrice:9.99]]];
```

####**How to use**

**Buy:**

Buying a `LifetimeVG` means that the user will now own the item for the rest of time. Lifetime goods can be bought only once.

``` objectivec
[StoreInventory buyItemWithItemId:@"marriage"];
```

**Give:**

Give a `LifetimeVG` and get nothing in return.
This is useful if you’d like to give your users a `LifetimeVG` when they first download your game.

``` objectivec
[StoreInventory giveAmount:1 ofItem:@"marriage"];
```

**Take:**

This function simply deducts the user's balance. In case of a refund request, it is your responsibility to give the user back whatever he/she paid.

``` objectivec
[StoreInventory takeAmount:1 ofItem:@"marriage"];
```

####**Check ownership**
Check the ownership of a lifetime good:

``` objectivec
//If the balance is greater than 0, the user owns this LifetimeVG.
[StoreInventory getItemBalance:@"marriage"];
```


###[EquippableVG](https://github.com/soomla/ios-store/blob/master/SoomlaiOSStore/domain/virtualGoods/EquippableVG.h)

####**How to define**
There are 3 types of Equipping models: `GLOBAL`, `CATEGORY`, and `LOCAL`. In this example we're defining 2 characters, George and Kramer. These are `CATEGORY` equippable goods because the user can own both characters but can play only as one at a time.

``` objectivec
// Character "George" can be purchased for 350 Muffins.
GEORGE_GOOD = [[EquippableVG alloc]
    initWithName:@"George"
    andDescription:@"The best muffin eater in the north"
    andItemId:GEORGE_GOOD_ITEM_ID
    andPurchaseType:[[PurchaseWithVirtualItem alloc]
        initWithVirtualItem:MUFFINS_CURRENCY_ITEM_ID
        andAmount:350]
    andEquippingModel:kCategory];

// Character "Kramer" can be purchased for 500 Muffins.
KRAMER_GOOD = [[EquippableVG alloc]
    initWithName:@"Kramer"
    andDescription:@"Knows how to get muffins"
    andItemId:KRAMER_GOOD_ITEM_ID
    andPurchaseType:[[PurchaseWithVirtualItem alloc]
        initWithVirtualItem:MUFFINS_CURRENCY_ITEM_ID
        andAmount:500]
    andEquippingModel:kCategory];
```

####**How to use**
**Buy:**

Buying an `EquippableVG` is exactly like buying a [`LifetimeVG`](#lifetimevg). The balance of "kramer" will be checked and if it is 0 buying will be allowed.

``` objectivec
[StoreInventory buyItemWithItemId:@"kramer"];
```

**Give:**

Give an `EquippableVG` and get nothing in return.
This is useful if you’d like to give your users a free character to begin with when they first download your game.

``` objectivec
[StoreInventory giveAmount:1 ofItem:@"george"];
```

**Take:**

This function simply deducts the user's balance. In case of a refund request, it is your responsibility to give the user back whatever he/she paid.

``` objectivec
[StoreInventory takeAmount:1 ofItem:@"kramer"];
```

**Equip & Unequip:**

``` objectivec
// The user equips an owned good, George:
[StoreInventory equipVirtualGoodWithItemId:@"george"];

// The user tries to equip Kramer (while he has George equipped):
[StoreInventory equipVirtualGoodWithItemId:@"kramer"];
// Internally, George will be unequipped and Kramer will be equipped instead.

// The user unequips the currently equipped character (Kramer).
[StoreInventory unEquipVirtualGoodWithItemId:@"kramer"];
```

####**Check ownership**
Check if user owns Kramer:

``` objectivec
//If the balance is greater than 0, the user owns Kramer.
[StoreInventory getItemBalance:@"kramer"];
```

####**Check equipping status**
Check if Kramer is currently equipped:

``` objectivec
[StoreInventory isVirtualGoodWithItemIdEquipped:@"kramer"];
```


###[UpgradeVG](https://github.com/soomla/ios-store/blob/master/SoomlaiOSStore/domain/virtualGoods/UpgradeVG.h)

####**How to define**
In the following example there is a `SingleUseVG` for "Muffin Cake" and 2 levels to upgrade the Muffin Cake.

``` objectivec
// Create a SingleUseVG for "Muffin Cake"

MUFFIN_CAKE_GOOD = [[SingleUseVG alloc]
    initWithName:@"Muffin Cake"
    andDescription:@"Customers buy a double portion on each purchase of this cake"
    andItemId:MUFFIN_CAKE_GOOD_ITEM_ID
    andPurchaseType:[[PurchaseWithVirtualItem alloc]
        initWithVirtualItem:MUFFINS_CURRENCY_ITEM_ID
        andAmount:225]];

// Create 2 UpgradeVGs that represent 2 levels for the Muffin Cake good.

LEVEL_1_GOOD = [[UpgradeVG alloc]
    initWithName:@"Level 1"
    andDescription:@"Muffin Cake Level 1"
    andItemId:LEVEL_1_GOOD_ITEM_ID
    andPurchaseType:[[PurchaseWithVirtualItem alloc]
        initWithVirtualItem:MUFFINS_CURRENCY_ITEM_ID
        andAmount:50]
    andLinkedGood:MUFFIN_CAKE_GOOD_ITEM_ID
    andPreviousUpgrade:@""
    andNextUpgrade:LEVEL_2_GOOD_ITEM_ID];

LEVEL_2_GOOD = [[UpgradeVG alloc]
    initWithName:@"Level 2"
    andDescription:@"Muffin Cake Level 2"
    andItemId:LEVEL_2_GOOD_ITEM_ID
    andPurchaseType:[[PurchaseWithVirtualItem alloc]
        initWithVirtualItem:MUFFINS_CURRENCY_ITEM_ID
        andAmount:250]
    andLinkedGood:MUFFIN_CAKE_GOOD_ITEM_ID
    andPreviousUpgrade:LEVEL_1_GOOD_ITEM_ID
    andNextUpgrade:@""];
```

####**How to use**
**Buy:**

When a user buys an upgrade, the `buy` method checks that the upgrade that's being purchased is valid.

``` objectivec
[StoreInventory buyItemWithItemId:LEVEL_1_GOOD_ITEM_ID];
```

**Upgrade:**
When you upgrade a virtual good, the method performs a check to see that this upgrade is valid.

``` objectivec
[StoreInventory upgradeVirtualGood:MUFFIN_CAKE_GOOD_ITEM_ID];
```

**Remove upgrades:**
Remove all upgrades from the virtual good with the given ID (Muffin cake in our example).

``` objectivec
[StoreInventory removeUpgrades:MUFFIN_CAKE_GOOD_ITEM_ID];
```

**Give:**

To give a free upgrade use `forceUpgrade`.

``` objectivec
[StoreInventory forceUpgrade:LEVEL_1_GOOD_ITEM_ID];
```

**Take:**

This function simply deducts the user's balance. In case of a refund request, it is your responsibility to give the user back whatever he/she paid. Essentially, taking an upgrade is the same as a downgrade.

``` objectivec
// The parameter amount is not used in this method.
[StoreInventory takeAmount:1 ofItem:LEVEL_1_GOOD_ITEM_ID];
```

####**Get current upgrade**

To get the current upgrade of a virtual good use `goodCurrentUpgrade`. If the good has no upgrades, the method will return null.

``` objectivec
[StoreInventory goodCurrentUpgrade:MUFFIN_CAKE_GOOD_ITEM_ID];
```
####**Get Current Upgrade Level**
To find out the upgrade level of a virtual good use `goodUpgradeLevel`. If the good has no upgrades, the method returns 0.

``` objectivec
[StoreInventory goodUpgradeLevel:MUFFIN_CAKE_GOOD_ITEM_ID];
```

##[VirtualCategory](https://github.com/soomla/ios-store/blob/master/SoomlaiOSStore/domain/VirtualCategory.h)

Divide your store's virtual goods into categories. Virtual categories become essential when you want to include `CATEGORY` `EquippableVG`s in your game.

####**How to define**

``` objectivec
// Assume that MUFFIN_CAKE_GOOD_ITEM_ID, PAVLOVA_GOOD_ITEM_ID, etc.. are item IDs of virtual goods that have been declared.
_MUFFINS_CATEGORY  = [[VirtualCategory alloc]
    initWithName:@"Muffins"
    andGoodsItemIds:@[MUFFIN_CAKE_GOOD_ITEM_ID, PAVLOVA_GOOD_ITEM_ID, MUFFIN_CAKE_GOOD_ITEM_ID]];
```

####**Get ccategory**
Check which category an item belongs to:

``` objectivec
[StoreInfo categoryForGoodWithItemId:MUFFIN_CAKE_GOOD_ITEM_ID];
```