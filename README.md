<p align="center">
  <img src="logo.png" alt="OpsPulse Logo" width="200">
</p>

# OpsPulse

**Real-time e-commerce operations dashboard for SMBs, built with Elixir, Phoenix, and LiveView.**

[![Elixir CI](https://github.com/kojjob/ops_pulse/actions/workflows/ci.yml/badge.svg)](https://github.com/kojjob/ops_pulse/actions/workflows/ci.yml)
[![Security Audit](https://github.com/kojjob/ops_pulse/actions/workflows/security.yml/badge.svg)](https://github.com/kojjob/ops_pulse/actions/workflows/security.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## Overview

OpsPulse is your **"mission control"** for e-commerce. It provides a single, beautiful, live view of your most critical metrics, bringing your store's data to life and empowering you to make instant, data-driven decisions.

- **Real-time Monitoring:** Watch orders, visitors, and cart activity as they happen.
- **Unified Insight:** Connect your Shopify or WooCommerce store in minutes.
- **Instant Alerts:** Get notified the moment high-value orders arrive or stock levels drop.

---

## The Problem & Solution

**The Problem:** E-commerce owners are often "data-rich but insight-poor." Standard dashboards update every 15-30 minutes, causing them to miss critical moments during flash sales or marketing campaigns.

**The Solution:** OpsPulse delivers a sub-second latency dashboard that captures the live, exciting nature of your business, ensuring you never miss a beat.

---

## Tech Stack

OpsPulse is built on a modern, high-performance stack:

- **[Elixir](https://elixir-lang.org/)** & **[Erlang/OTP](https://www.erlang.org/)** – Scalable, fault-tolerant foundation.
- **[Phoenix LiveView](https://phoenixframework.org/)** – Real-time user experiences with server-rendered HTML.
- **[Ash Framework](https://ash-hq.org/)** – Declarative, resource-based data layer.
- **[PostgreSQL](https://www.postgresql.org/)** – Reliable data storage.
- **[TailwindCSS](https://tailwindcss.com/)** – Premium, responsive UI design.
- **[Oban](https://github.com/sorentwo/oban)** – Robust background job processing.

---

## Architecture

OpsPulse uses a reactive architecture to ensure data is always fresh.

```mermaid
graph LR
    A[E-commerce Platform] -- Webhook --> B[Phoenix Endpoint]
    B -- Enqueue --> C[Oban Job]
    C -- Update --> D[Ash Resource / DB]
    D -- Broadcast --> E[Phoenix PubSub]
    E -- Push --> F[LiveView Browser]
```

1. **Ingestion:** Platforms like Shopify send webhooks for new orders.
2. **Reliable Processing:** Oban jobs process payloads to ensure zero data loss.
3. **Reactive Updates:** Changes in the data layer are broadcasted via PubSub.
4. **Live UI:** Connected clients receive updates instantly via WebSockets.

---

## Getting Started

### Prerequisites

- **Elixir 1.15+**
- **Erlang/OTP 26+**
- **PostgreSQL 15+**
- **Node.js 18+**

### Installation

1. **Clone the repository**
   ```sh
   git clone https://github.com/kojjob/ops_pulse.git
   cd ops_pulse
   ```

2. **Install dependencies**
   ```sh
   mix deps.get
   cd assets && npm install && cd ..
   ```

3. **Database Setup**
   ```sh
   mix ecto.create
   mix ecto.migrate
   ```

4. **Environment Configuration**
   Create a `.env` file:
   ```env
   SECRET_KEY_BASE="your_secret_key_base"
   DATABASE_URL="postgresql://postgres:password@localhost/opspulse_dev"
   SHOPIFY_API_KEY="..."
   SHOPIFY_API_SECRET="..."
   STRIPE_SECRET_KEY="..."
   ```

5. **Start the Server**
   ```sh
   mix phx.server
   ```
   Visit `http://localhost:4000`.

---

## Testing

We value quality. Run the test suite with:

```sh
mix test
```

---

## Roadmap

- [x] **MVP:** User Auth, Shopify Integration, Core 4 Metrics, Live Feed.
- [ ] **v1.1:** WooCommerce, Custom Email/Push Alerts, Date Filtering.
- [ ] **v2.0:** AI Insights, Meta Ads Integration, Team Collaboration.

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repo.
2. Create a feature branch (`git checkout -b feature/amazing-feature`).
3. Commit your changes (`git commit -m 'Add some amazing feature'`).
4. Push to the branch (`git push origin feature/amazing-feature`).
5. Open a Pull Request.

---

## License

Distributed under the MIT License. See `LICENSE.md` for more information.
