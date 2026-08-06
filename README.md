# Pastry Pup website

The storefront is a static GitHub Pages site. Firebase's no-cost services provide the live menu, staff login, and arcade scores.

## Square checkout

Customers build an order in the cart, then email the pre-filled order request to Pastry Pup. After confirming availability, Pastry Pup sends the customer a secure Square invoice/payment link. This keeps payment in Square without requiring Firebase Cloud Functions or exposing Square credentials in browser code.
