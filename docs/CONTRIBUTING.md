# Development Setup

## Environment

Copy the provided environment template.

```bash
cp .env.example .env
```

Fill in all required credentials before starting the application.

---

## Backend

Install dependencies.

```bash
npm install
```

Run migrations.

```bash
pnpm run db:verify-migrations
```

Verify migrations forward and rollback:
```bash
pnpm run db:verify-migrations
```

Run mutation testing for financial authorization:
```bash
pnpm run test:mutation
```

Run Horizon performance regression tests:
```bash
pnpm run test:perf:horizon
```

Run cross-package generated type checks:
```bash
pnpm run check:generated-types
```

Start development.

```bash
npm run dev
```

---

## Rust Contracts

Compile contracts.

```bash
cargo build
```

Execute tests.

```bash
cargo test
```

Lint.

```bash
cargo clippy --all-targets --all-features
```

Format.

```bash
cargo fmt
```

---

## Before Opening a Pull Request

Verify that:

- The project builds successfully.
- Database migrations are up to date.
- Rust tests pass.
- TypeScript tests pass.
- Linting passes.
- Formatting passes.