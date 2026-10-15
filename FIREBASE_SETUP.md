# SHOPX Firebase setup

SHOPX uses Firebase Authentication and Cloud Firestore. Product images are stored as Base64 data URLs in Firestore, so Cloud Storage is not required. The Firebase web config is in `index.html`; web API keys identify the Firebase project and are not server credentials. Deploy the included Firestore rules before using real customer data.

## Enable Firebase services

1. Open the `shopx-921b7` project in the Firebase console.
2. In **Authentication → Sign-in method**, enable **Google** and select a support email.
3. In **Authentication → Settings → Authorized domains**, add `localhost` for local development and the domain used to host SHOPX.
4. Create a Cloud Firestore database.

## Run and deploy

Do not open `index.html` using a `file://` URL. Google sign-in needs an HTTP origin. Start a local static server from this folder, for example:

```powershell
npx serve .
```

After confirming that the Firebase CLI targets the intended project, deploy the security rules:

```powershell
firebase use shopx-921b7
firebase deploy --only firestore:rules
```

This replaces any existing rules for the selected project. Do not use test-mode rules with real customer data.

## Data model and marketplace flows

- `users/{uid}`: user's name, phone, immutable `buyer`, `seller`, or `both` role, Google email/photo, and an `addresses` array (up to 20 saved addresses).
- `products/{productId}`: seller-owned listing, INR price, integer stock, and a Base64 image data URL or the fixed SHOPX placeholder URL/path. Product image files are capped at 700 KB so the resulting document fits Firestore's document size limit. Sellers may create, edit price/stock, and delete only their own listings.
- `products/{productId}/reviews/{uid}`: one 1–5 star review per signed-in user and product, with reviewer name and text.
- `orders/{orderId}`: COD-only order line with buyer, seller, delivery address, item/price snapshot, status, and optional expected delivery date. A cart with multiple products creates one independently accepted order per item.
  Checkout writes order documents and decrements stock in one Firestore transaction. Sellers see pending orders in the header Mailbox and can accept an order with an expected delivery date or reject it; rejection updates the order status and returns the reserved quantity to the listing in one transaction. Buyers see order status and delivery dates in **Your orders**.

## Smoke test

1. Sign in with Google, choose a role, and complete the profile. Add a product as a seller or `both` user.
2. Publish with an image and then publish without one; the latter listing should use the SHOPX placeholder. Verify a listing with zero stock cannot be added to the bag.
3. As a buyer or `both` user, add an available listing, select or add a delivery address, and place a COD order.
4. Confirm stock decreased and the order appears as **Pending Acceptance** in the seller Mailbox and buyer's **Your orders**.
5. Accept the order as its seller, set a future delivery date, and confirm the updated status/date appears for the buyer.
6. Place another order, reject it as its seller, and confirm its status changes for the buyer and stock is restored.
7. Submit a product review and confirm its rating and reviewer name appear on the product card.

Legacy documents in `profiles/{uid}` are read only to prefill onboarding; new profiles are stored in `users/{uid}`. The current storefront treats legacy product documents without a `stock` field as out of stock; edit those listings and set an inventory count in the seller studio.
