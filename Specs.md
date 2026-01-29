
# OpsPulse: Technical Specifications

**Version:** 1.0  
**Date:** October 26, 2023

---

## 1. Overview
OpsPulse is a real-time web application built on the Elixir ecosystem. It leverages **Phoenix LiveView** for a highly interactive, server-rendered frontend, and the **Ash Framework** for a declarative, resource-based data layer. The application receives data from e-commerce platforms via webhooks and APIs, processes it, and broadcasts it to connected clients via WebSockets.

2. Technology Stack
Language: Elixir 1.15+
Web Framework: Phoenix 1.7+
Frontend: Phoenix LiveView, TailwindCSS 3.0+
Data Layer: Ash Framework 2.0+
Database: PostgreSQL 15+
Background Jobs: Oban
Hosting: Fly.io or Render (for ease of deployment and global distribution)
Payments: Stripe API
3. System Architecture
[E-commerce Platform] --(Webhook)--> [Phoenix Endpoint] --> [Oban Job]
|
V
[GenServer]
|
V
[Phoenix PubSub]
^
|
[LiveView Browser] <--(WebSocket)--- [LiveView Process] <----+


1.  **Ingestion:** Shopify/WooCommerce sends a webhook (e.g., `orders/create`) to a dedicated Phoenix endpoint.
2.  **Processing:** The endpoint enqueues an `Oban` job to process the payload, ensuring the webhook responds immediately and reliably.
3.  **State Management:** An Oban job processes the data, updating the `Ash` resources (e.g., `ash:create(Order)`). This also triggers a `PubSub` broadcast.
4.  **Real-time Distribution:** The `PubSub` broadcast is sent to a specific topic (e.g., `store_live:store_123`).
5.  **Frontend Update:** The `LiveView` process for each connected client subscribed to that topic receives the message and updates the UI on the user's screen.

## 4. Data Model (Ash Resources)

### Account
- Represents a customer account (the user who pays).
- `has_many :users`
- `has_many :stores`
- Attributes: `:name`, `:stripe_customer_id`, `:plan` (enum: `:basic`, `:plus`, `:pro`)

### Store
- Represents a single e-commerce store.
- `belongs_to :account`
- `has_many :integrations`
- Attributes: `:name`, `:platform` (enum: `:shopify`, `:woocommerce`), `:domain`, `:external_id` (e.g., Shopify shop ID)

### Integration
- Stores credentials and state for a store connection.
- `belongs_to :store`
- Attributes: `:type` (enum: `:shopify`), `:access_token`, `:webhook_secret`, `:status` (enum: `:active`, `:error`)

### Metric
- A polymorphic resource to store pre-aggregated daily/hourly stats.
- `belongs_to :store`
- Attributes: `:name` (e.g., `daily_revenue`, `live_visitors`), `:value`, `:timestamp`

### Alert
- User-defined notifications.
- `belongs_to :store`
- Attributes: `:type` (enum: `:high_value_order`, `:low_stock`), `:config` (a map of settings, e.g., `%{threshold: 500}`), `:is_active`

## 5. API & Integration Details

### Shopify
- **Authentication:** OAuth 2.0 flow.
- **Webhooks:** Subscribe to `orders/create`, `orders/updated`, `inventory_levels/update`, `carts/create`. Use `shopify.engineering` best practices for verification.
- **API:** Use the GraphQL Admin API for fetching historical data during initial sync and the REST API for specific webhook data if needed. Respect rate limits strictly (2 calls/sec per store).

### WooCommerce
- **Authentication:** API keys (Consumer Key & Secret).
- **Webhooks:** Requires a plugin (e.g., a custom-built one or a third-party solution) to send webhooks for new orders and stock changes.
- **API:** Use the WooCommerce REST API v3.

## 6. Real-Time Implementation

- **Phoenix PubSub:** Each store will have its own topic: `store_live:#{store_id}`.
- **LiveView Mount:** When a user's dashboard LiveView mounts, it will subscribe to the relevant store topic(s).
- **Broadcasts:** After processing a webhook, the application will broadcast a structured message: `{:new_order, order_data}` or `{:metric_update, metric_name, new_value}`.
- **LiveView `handle_info`:** The LiveView process will have `handle_info` callbacks to listen for these broadcasts and update its assigns, triggering a UI diff.

## 7. Deployment & Infrastructure

- **App Server:** Deployed on Fly.io to leverage their global Anycast network, reducing latency for users worldwide. A single `performance-2x` instance should be sufficient for the first 1000 stores.
- **Database:** Fly.io Postgres or a managed service like ElephantSQL/Render Postgres. Use read replicas for scaling read-heavy operations (like historical data queries).
- **Ingress:** Use Cloudflare (free tier) in front of the app for DDoS protection and SSL termination.
- **Background Jobs:** Oban will use the database as its job queue, which is simple and reliable for this scale.

## 8. Security Considerations

- **API Keys:** All external API keys and webhook secrets must be stored encrypted at rest in the database. Use `:crypto.strong_rand_bytes/1` for generating secrets and `Ecto.Type` for encryption/decryption.
- **Data in Transit:** Enforce HTTPS everywhere. All communication between the client and server, and between our server and external APIs, must be over TLS.
- **Authentication:** Use `phx.gen.auth` as a starting point for secure user authentication, including session management and password hashing (Argon2).
- **Compliance:** Be mindful of GDPR/CCPA. Ensure data can be exported or deleted upon user request. The dashboard should not display PII unless necessary.