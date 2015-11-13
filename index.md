---
layout: home
title: Bill McRoberts
home: true
---

## Dependency Service for Carrier In-App Purchases (Fortumo)

Android Services &bull; Binding a Java Library &bull; Xamarin.Forms &bull; DependencyService &bull; In-App Purchases &bull; Carrier Billing

Xamarin.Forms support for cross-platform in-app purchasing via Fortumo for carrier billing.

<img src="images/screenshot-android-inapp.png" width="40%">

My DependencyService for In-App Purchases (IAP) is supported in all mobile markets in the world except Android for China - ouch. This may change in the future, but in the meantime, let's see if we can fill this gap. In researching this, I came across Fortumo ([http://fortumo.com](http://fortumo.com "Fortumo")), a service provider for carrier based billing with support for China. While they didn't have a Xamarin component, they did have an Android library. OK, I've done several Bindings to Java Libraries and I have a defined interface for an IAP DependencyService, let's see if we can implement a DependencyService for IAP via Fortumo's carrier billing service. The fact that Fortumo requires you to create an Android Service only makes this more interesting.

### Setup
1. In Visual Studio, start with a new Blank App (Xamarin.Forms.Portable), and remove the iOS and Windows Phone platform projects.
1. Head over to Fortumo ()[http://fortumo.com](http://fortumo.com "Fortumo")) and sign-up as a developer.
2. Download the Fortumo Android SDK jar.
3. In your Visual Studio solution you created above, create a new Bindings Library (Android) project.
1. Place the Fortumo Android SDK jar file in the resulting Jars folder.
2. Add the Fortumo Android SDK jar file to the project and set the Build Action to ```EmbeddedJar```.
3. Build the Bindings project and make sure there are no errors.

### The Interface
The interface exposes an API and set of events to consume IAP.

> But wait you say, can't we do this with a Task based API instead of hooking up events? Hang in there that's exactly where I want to evolve this sample in the future.

In your portable project create a Services folder and the interface ```IFortumoInAppService```.

Add the following API to the interface:

        string PracticeModeProductId { get; }

        /// <summary>
        /// Starts the setup of this Android application by connection to the Google Play Service
        /// to handle In-App purchases.
        /// </summary>
        void Initialize();

        /// <summary>
        /// Queries the inventory asynchronously and returns a list of Xamarin.Android.InAppBilling.Products
        /// matching the given list of SKU numbers.
        /// </summary>
        /// <param name="skuList">Sku list.</param>
        /// <param name="itemType">The Xamarin.Android.InAppBilling.ItemType of product being queried.</param>
        /// <returns>List of Xamarin.Android.InAppBilling.Products matching the given list of SKUs.
        /// </returns>
        void QueryInventory();

        /// <summary>
        /// Buys the given Xamarin.Android.InAppBilling.Product
        /// 
        /// This method automatically generates a unique GUID and attaches it as the
        /// developer payload for this purchase.
        /// </summary>
        /// <param name="product">The Xamarin.Android.InAppBilling.Product representing the item the users
        /// wants to purchase.</param>
        void PurchaseProduct(string productId);

        void RestoreProducts();

        /// <summary>
        /// For testing purposes only.
        /// </summary>
        void RefundProduct();

Add the following events to the interface:

        /// <summary>
        /// Occurs when a query inventory transactions completes successfully with Google Play Services.
        /// </summary>
        event OnQueryInventoryDelegate OnQueryInventory;

        /// <summary>
        /// Occurs after a product has been successfully purchased Google Play.
        /// 
        /// This event is fired after a OnProductPurchased which is raised when the user
        /// successfully logs an intent to purchase with Google Play.
        /// </summary>
        event OnPurchaseProductDelegate OnPurchaseProduct;

        /// <summary>
        /// Occurs after a successful products restored transactions with Google Play.
        /// </summary>
        event OnRestoreProductsDelegate OnRestoreProducts;

        /// <summary>
        /// Occurs when there is an error querying inventory from Google Play Services.
        /// </summary>
        event OnQueryInventoryErrorDelegate OnQueryInventoryError;

        /// <summary>
        /// Occurs when the user attempts to buy a product and there is an error.
        /// </summary>
        event OnPurchaseProductErrorDelegate OnPurchaseProductError;

        /// <summary>
        /// Occurs when the user attempts to restore products and there is an error.
        /// </summary>
        event OnRestoreProductsErrorDelegate OnRestoreProductsError;

        /// <summary>
        /// Occurs when on user canceled.
        /// </summary>
        event OnUserCanceledDelegate OnUserCanceled;

        /// <summary>
        /// Occurs when there is an in app billing procesing error.
        /// </summary>
        event OnInAppBillingProcessingErrorDelegate OnInAppBillingProcesingError;

        /// <summary>
        /// Raised when Google Play Services returns an invalid bundle from previously
        /// purchased items
        /// </summary>
        event OnInvalidOwnedItemsBundleReturnedDelegate OnInvalidOwnedItemsBundleReturned;

        /// <summary>
        /// Occurs when a previously purchased product fails to validate.
        /// </summary>
        event OnPurchaseFailedValidationDelegate OnPurchaseFailedValidation;

And finally add these delegates in the ```IFortumaInAppService``` file outside of the interface definition. The delegate definitions are used in the event signatures:

    public delegate void OnQueryInventoryDelegate();

    public delegate void OnPurchaseProductDelegate();

    public delegate void OnRestoreProductsDelegate();

    public delegate void OnQueryInventoryErrorDelegate(int responseCode, IDictionary<string, object> skuDetails);

    public delegate void OnPurchaseProductErrorDelegate(int responseCode, string sku);

    public delegate void OnRestoreProductsErrorDelegate(int responseCode, IDictionary<string, object> skuDetails);

    public delegate void OnUserCanceledDelegate();

    public delegate void OnInAppBillingProcessingErrorDelegate(string message);

    public delegate void OnInvalidOwnedItemsBundleReturnedDelegate(IDictionary<string, object> ownedItems);

    public delegate void OnPurchaseFailedValidationDelegate(InAppPurchase purchase, string purchaseData, string purchaseSignature);

### The Implementation
I won't be showing code in this section - only pointing you to the necessary files in the GitHub repository to add to your implementation. Actual code will be exposed in the Code Walk-through below.

1. Add a Services folder to your Android platform project.
2. Add the following folders from the GitHub repository ([https://github.com/simsip-admin/FortumoInApp](https://github.com/simsip-admin/FortumoInApp "FortumoInApp"))
   * FortumoInAppService.cs
   * FortumoPaymentStatusReceiver.cs

### Android Configuration
The Fortumo Android IAP implementation depends on an Android Service. You just added the code for that above with the file ```FortumoPaymentStatusReceiver.cs```. But we still need to add the appropriate Android configuration for this Service.

In the Android platform project, in the Properties folder, open the ```AndroidManifest.xml``` file.

Add the following entries outside of the ```<application>``` entry:


	<!-- Fortumo: permissions -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
	<uses-permission android:name="android.permission.RECEIVE_SMS" />
	<uses-permission android:name="android.permission.SEND_SMS" />
    
	<!-- Fortumo: Define your own permission to protect payment broadcast -->
	<permission android:name="com.simsip.permission.PAYMENT_BROADCAST_PERMISSION" android:label="Read payment status" android:protectionLevel="signature" />
	<!-- Fortumo: "signature" permission granted automatically by system, without notifying user. -->
	<uses-permission android:name="com.simsip.permission.PAYMENT_BROADCAST_PERMISSION" />

Add the following entries inside of the ```<application>``` entry:

		<!-- Fortumo: Declare these objects, this is part of Fortumo SDK, and should not be called directly -->
		<receiver android:name="mp.MpSMSReceiver">
			<intent-filter>
				<action android:name="android.provider.Telephony.SMS_RECEIVED" />
			</intent-filter>
		</receiver>
		<service android:name="mp.MpService" />
		<service android:name="mp.StatusUpdateService" />
		<activity android:name="mp.MpActivity" android:theme="@android:style/Theme.Translucent.NoTitleBar" android:configChanges="orientation|keyboardHidden|screenSize" />
		<!-- Fortumo: Implement you own BroadcastReceiver to track payment status, should be protected by "signature" permission -->
		<!-- See Simsip.LineRunner.Services.Inapp.PaymentStatusReceiver for attribute based build up of this:
		<receiver android:name="Simsip.LineRunner.Services.Inapp.PaymentStatusReceiver" android:permission="com.simsip.permission.PAYMENT_BROADCAST_PERMISSION">
			<intent-filter>
				<action android:name="mp.info.PAYMENT_STATUS_CHANGED" />
			</intent-filter>
		</receiver>
        -->

We'll see how all of this hooks-up in the Code Walk-through section below.

### The Sample App
I have kept the UI and MVVM architecture as simple as possible so that the focus can be on the transaction flows for IAP.

### Code Walk-through

#### Initializing

#### Querying Inventory

#### Making a Purchase

#### Restoring a Purchase

### Testing

### Publishing

   




