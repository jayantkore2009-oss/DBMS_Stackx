# SHOPX Firebase setup

SHOPX uses Firebase Authentication, Cloud Firestore, and Cloud Storage. The Firebase web config is in `index.html`; web API keys identify the Firebase project and are not server credentials. Deploy the included security rules before using real customer data.

## Enable Firebase services

1. Open the `shopx-921b7` project in the Firebase console.
2. In **Authentication → Sign-in method**, enable **Google** and select a support email.
3. In **Authentication → Settings → Authorized domains**, add `localhost` for local development and the domain used to host SHOPX.
4. Create a Cloud Firestore database.
5. In **Storage → Files**, finish **Get started** and create the project's default bucket. Confirm that the bucket matches `firebaseConfig.storageBucket` in `index.html`.

## Run and deploy

Do not open `index.html` using a `file://` URL. Google sign-in needs an HTTP origin. Start a local static server from this folder, for example:

```powershell
npx serve .
```

After confirming that the Firebase CLI targets the intended project, deploy the security rules:

```powershell
firebase use shopx-921b7
firebase deploy --only firestore:rules,storage
```

This replaces any existing rules for the selected project. Do not use test-mode rules with real customer data.

## Data model and marketplace flows

- `users/{uid}`: user's name, phone, immutable `buyer`, `seller`, or `both` role, Google email/photo, and an `addresses` array (up to 20 saved addresses).
- `products/{productId}`: seller-owned listing, INR price, integer stock, and Storage image URL/path. Sellers may create, edit price/stock, and delete only their own listings.
- `products/{productId}/reviews/{uid}`: one 1–5 star review per signed-in user and product, with reviewer name and text.
- `orders/{orderId}`: COD-only order line with buyer, seller, delivery address, item/price snapshot, status, and optional expected delivery date. A cart with multiple products creates one independently accepted order per item.
- `product-images/{uid}/{fileName}`: seller-uploaded JPG, PNG, or WebP images up to 5 MB.

Checkout writes order documents and decrements stock in one Firestore transaction. Sellers see their pending orders in the Mailbox and can accept an order with an expected delivery date. Buyers see order status and delivery dates in **Your orders**.

## Smoke test

1. Sign in with Google, choose a role, and complete the profile. Add a product as a seller or `both` user.
2. Verify a listing with zero stock cannot be added to the bag. As a buyer or `both` user, add an available listing, select or add a delivery address, and place a COD order.
3. Confirm stock decreased and the order appears as **Pending Acceptance** in the seller Mailbox and buyer's **Your orders**.
4. Accept the order as its seller, set a future delivery date, and confirm the updated status/date appears for the buyer.
5. Submit a product review and confirm its rating and reviewer name appear on the product card.

Legacy documents in `profiles/{uid}` are read only to prefill onboarding; new profiles are stored in `users/{uid}`. The current storefront treats legacy product documents without a `stock` field as out of stock; edit those listings and set an inventory count in the seller studio.

Cloud Storage must be initialized in the Firebase project before image upload works. If Firebase CLI reports that Storage is not initialized, complete **Storage → Files → Get started**, verify the bucket name, then retry rule deployment.
